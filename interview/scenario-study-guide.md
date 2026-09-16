# System Design Interview Guide: Failure, Evidence, Decision

Use this guide to rehearse senior-level reasoning across reliability, performance, low-level design, and trade-offs. Numbers below are exercise assumptions, not universal production thresholds. Attempt each prompt before reading its answer; reattempt with the changed constraint.

## Answer Shape

1. **Contract:** critical journey, invariant, SLO, and failure scope.
2. **Evidence:** observations, competing hypotheses, and the measurement that separates them.
3. **Decision:** immediate containment, then the smallest justified design change.
4. **Downside:** new state, cost, failure mode, and rejected alternative.
5. **Verification:** recovery criteria and the trigger to revisit the choice.

Do not jump from a symptom to a product name. Distinguish “I would investigate” from “the evidence proves.”

## Shared Vocabulary

| Term | Meaning to use consistently |
|---|---|
| Availability | Successful eligible operations under a defined measurement contract; not merely process uptime |
| Reliability | Correct, recoverable outcomes over time despite specified faults and change |
| Durability | Preservation of acknowledged data under a stated failure model |
| Fault tolerance / resilience | Continued required service under specified faults / broader ability to absorb, degrade, and recover |
| Latency / throughput / goodput | Time per operation / completed work per time / successful useful work per time under the stated target |
| Offered / admitted load | Work attempting to enter / work accepted into the measured boundary |
| Utilization / saturation | Resource use / demand exceeding immediate service capacity, often seen as waiting |
| Attempt / retry | Any execution including the initial one / an additional execution after it |
| Timeout / deadline | Limit on one wait / end-to-end completion limit; neither proves rollback |
| SLI / SLO / SLA | Measured indicator / target over a window / external commitment and its consequences |
| RPO / RTO | Acceptable recovery data-loss window / acceptable time to restore a named capability |
| Linearizability / serializability | Operations respect real-time order / committed transactions behave as a serial execution; these are distinct guarantees |
| Eventual consistency / bounded staleness | Convergence under the model's conditions / an explicit freshness bound; eventual alone gives no fixed time bound |
| Idempotency | Repeating the same logical operation preserves its intended effect within the documented scope and retention |

Detailed definitions: [performance](../fundamentals/latency-vs-throughput.md), [reliability](../fundamentals/reliability.md), [availability](../fundamentals/availability.md), [CAP](../fundamentals/cap-theorem.md), [PACELC](../fundamentals/pacelc.md), [recovery](../reliability/disaster-recovery.md).

## Drill 1 — Payment Times Out After Capture

**Prompt:** The provider captured a payment, but your service timed out before saving the response. The client retries. How do you prevent another charge?

**Model reasoning:** A timeout means unknown outcome. Reuse the same logical operation key; enforce durable local uniqueness with a request fingerprint and pending/completed status. Carry a stable key to the provider and query its operation status. Reconcile pending operations after crashes. Return a truthful pending response with status lookup rather than marking the payment failed. Local atomicity cannot include an independent provider call.

**Trade-off:** Durable operation tracking and reconciliation add state and operational work. They address the ambiguity that blind retries cannot. Keep deduplication through the supported replay horizon; define what happens after expiry.

**Verify:** Concurrent duplicates and crashes before/after capture produce at most one provider effect within its idempotency contract. Track duplicate effects, pending age, and reconciliation mismatches.

**Changed constraint:** The provider has no idempotency support. Explain why automatic retries cannot promise at-most-one charge; use available status lookup and reconciliation, and route unresolved cases to controlled recovery.

Read: [idempotency boundaries](../messaging/delivery-semantics-and-idempotency.md).

## Drill 2 — Low CPU, Slow Requests, Full DB Pool

**Prompt:** App CPU is 40%; all 100 DB connections are occupied; p99 rises from 150 ms to 1 s while completions plateau. Increase the pool?

**Model reasoning:** First separate pool waiting from query and lock time, by endpoint and recent change. If a hot inventory row is blocking transactions, bound admission and shorten lock duration. Scaling app replicas adds waiters. If queries are healthy and the DB has measured spare capacity, cautiously increasing the aggregate pool across instances may help.

**Trade-off:** Smaller admission limits reject some work but protect useful throughput. A larger pool can move the queue into the database and worsen contention.

**Verify:** At the same offered load and key skew, compare goodput, p99, rejections, lock waits, and DB saturation. Low CPU alone is not headroom evidence.

**Changed constraint:** DB execution and lock waits remain low, but application pool wait dominates. Explain the experiment that would justify a bounded pool increase.

Read: [worked bottleneck diagnosis](../fundamentals/bottleneck-identification.md#worked-diagnosis--low-cpu-high-p99).

## Drill 3 — Dashboard Says Healthy, Users Say Slow

**Prompt:** Fleet p99 is calculated by averaging instance p99s. Handler latency is stable, yet users see timeouts. What is missing?

**Model reasoning:** Aggregate compatible histogram observations before calculating the fleet quantile. Measure the complete user journey, including gateway queues, retries, and network time. Separate first attempts from all attempts; retain failures/timeouts rather than reporting only successful latency. Segment meaningful routes, versions, and regions without raw user-ID metric labels.

**Trade-off:** More instrumentation has storage and cardinality cost. Keep aggregate metrics bounded and use sampled traces/logs for individual operations.

**Verify:** Replay a slow dependency and a skewed workload. The user-level SLI should worsen even when fast successful handler samples remain unchanged. Do not add dependency p99s to derive the journey p99.

**Changed constraint:** Request volume is tiny. Discuss quantile instability and use event counts, longer observation windows, and appropriately scoped synthetic probes.

Read: [measurement contract](../fundamentals/latency-vs-throughput.md#measurement-contract), [signals](../observability/signals-and-diagnostic-questions.md).

## Drill 4 — Queue Grows Despite More Workers

**Prompt:** Accepted arrival rate is 1,200 jobs/s, completions are 1,000/s, and the oldest job is getting older. What changes first?

**Model reasoning:** With no other departures, backlog grows by 200 jobs/s. Inspect per-partition lag, processing time, poison messages, retry traffic, and the downstream quota. If workers are quota-bound, adding them raises contention rather than capacity. Bound new admission, isolate poison work, and preserve already accepted durable jobs with explicit recovery/status.

**Trade-off:** Backpressure defers new work; DLQ routing needs an owner and safe replay. Neither silently dropping accepted jobs nor an infinite queue meets a completion contract.

**Verify:** If restored capacity is 1,500/s and new arrivals stay 1,200/s, a 90,000-job backlog drains in about `90,000 / 300 = 300 s`, assuming uniform work and no retries. Measure oldest age as well as depth.

**Changed constraint:** One ordered partition owns most traffic. More consumers beyond usable partitions will not help; reconsider the key/ordering scope only if business semantics allow it.

Read: [overload](../reliability/overload-backpressure-and-load-shedding.md), [ordering](../messaging/ordering-partitions-and-consumer-groups.md).

## Drill 5 — Two Requests Complete the Same Order

**Prompt:** Both processes load `PAYMENT_PENDING`, call `markPaid()`, and save. Does the domain method enforce correctness?

**Model reasoning:** It validates one object only. Require verified capture, then atomically update the row conditioned on expected status and version. One update wins; the other reloads and distinguishes duplicate completion from conflict. Commit any outbox event with the state change. Provider idempotency handles the external effect; the row update alone cannot.

**Trade-off:** Optimistic concurrency creates conflicts that callers must handle. A row lock may suit short, highly contended local transactions, but holding it across remote I/O adds latency and failure coupling.

**Verify:** Exercise concurrent updates, duplicate callbacks, timeout after capture, and a crash before publication. Exactly one valid local transition and one corresponding outbox record should commit.

**Changed constraint:** The invariant spans multiple rows. A single-row version check is insufficient; design an appropriate transaction, constraint, locking protocol, or serializable execution with retry handling.

Read: [LLD concurrency example](../fundamentals/lld.md#concurrency).

## Drill 6 — Cache Outage Overloads the Database

**Prompt:** A cache served 95% of 10,000 reads/s. It fails; every miss falls through to the DB.

**Model reasoning:** Under this simplified workload, DB demand rises from 500 to 10,000 reads/s: 20×. Protect the origin with bounded fallback concurrency, request coalescing where keys repeat, and controlled cache warm-up. Serve stale data only for flows whose freshness contract permits it; critical decisions revalidate with the authority or fail honestly.

**Trade-off:** Rejecting requests or returning approved stale data sacrifices some availability/freshness to preserve critical operations. A cache improves average load but creates a failure-capacity dependency.

**Verify:** Test cold-cache and full-cache-loss conditions. Track origin goodput, fallback rate, queueing, rejected traffic, and stale-data age.

**Changed constraint:** These reads authorize purchases or check stock. Reject stale fallback for the authoritative decision; distinguish browsing from purchase execution.

Read: [cache design](../caching/cache-design-framework.md), [trade-offs](../fundamentals/trade-off-analysis.md).

## Drill 7 — Region Lost, Replica Behind

**Prompt:** The primary region is unavailable. The remote replica lags by 30 seconds, while the business requires zero loss of acknowledged payments. Promote it?

**Model reasoning:** The visible lag means this replica cannot establish that requirement. Establish the durable commit position and fence the old writer before promotion. If there is no surviving copy with all acknowledged commits, choose unavailability while recovering rather than claiming zero-loss failover. Confirm what the acknowledgement protocol actually guaranteed.

**Trade-off:** Cross-region synchronous durability can reduce acknowledged-loss risk under its stated fault model, but adds write latency and can reduce write availability. Backups address corruption/deletion separately from failover.

**Verify:** A drill must prove write authority, acknowledged-data preservation, dependency readiness, actual recovery time, and safe rejoin. An HTTP health check alone is insufficient.

**Changed constraint:** Analytics permits a five-minute RPO. A lagging replica may now fit the loss objective, subject to validation, fencing, and the capability's RTO.

Read: [disaster recovery](../reliability/disaster-recovery.md), [ownership](../distributed-systems/multi-region-data-ownership.md).

## Drill 8 — “We Need Sharding Now”

**Prompt:** A URL shortener creates 100 million mappings/month and receives 100 redirects per create. Does it need a sharded database immediately?

**Model reasoning:** With 30 days/month, average load is about 39 creates/s and 3,858 redirects/s. Assuming 10× peak yields about 386 and 38,580/s. At 500 bytes/mapping before replication, monthly growth is about 50 GB decimal. Validate retention, read locality, index cost, skew, and failure headroom against measured capacity. An indexed relational store is a candidate, not a proven answer.

**Trade-off:** Caching may protect repeated reads but adds freshness/fallback work. Sharding adds routing, rebalancing, and transaction constraints. Choose it when measured capacity or data size requires distribution, not from the monthly count alone.

**Verify:** Benchmark realistic lookups and writes at projected peaks and under replica loss. Set a revisit trigger tied to the latency SLO, storage runway, or tested capacity margin.

**Changed constraint:** The input is 100 million creates **per day**. Average write rate and storage growth become 30× higher; rerun the decision instead of reusing the original conclusion.

Read: [capacity estimation](capacity-estimation.md), [decision comparisons](../fundamentals/trade-off-analysis.md#decision-matrix-example--url-shortener).

## Self-Check

For each drill, award one point for each element in the five-part answer shape. A complete rehearsal needs all five plus a defensible response to the changed constraint. This is a practice checklist, not a hiring-level predictor. Use the broader [scoring rubric](scoring-rubric.md) for complete designs and [rapid review](rapid-review.md) before an interview.
