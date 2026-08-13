# Cache Design Framework

Use this framework to decide whether and how to cache during a system-design interview. The sequence keeps the discussion tied to requirements, failure behavior, and measurable outcomes instead of product names.

## Interview TL;DR

```text
1. Identify the bottleneck
2. Identify candidate data/work
3. Identify source of truth
4. Define freshness requirement
5. Choose cache location
6. Choose cache key
7. Choose read/write strategy
8. Choose TTL/invalidation
9. Set memory/eviction policy
10. Protect miss path
11. Design cache failure behavior
12. Design observability
13. Define reconsideration triggers
```

A strong answer can explain why every step exists, what it costs, how it fails, and which measurement would change it.

## 1. Identify the Bottleneck

Ask:

- Which endpoint, query, computation, dependency, or network path is limiting?
- Is the target latency p50, p95, or p99?
- Is the constraint CPU, I/O, connections, throughput, cost, rate limit, or cross-region distance?
- What are peak read/write QPS and reuse distribution?
- Would an index, query fix, batching, or data-model change solve it more directly?

Do not add a cache to an architecture diagram without naming the work it avoids.

## 2. Identify Candidate Data or Work

Good candidates have:

```text
repeated access
+ expensive source work
+ cacheable representation
+ acceptable freshness model
```

Estimate key cardinality, value-size distribution, reuse window, largest tenant, and hottest key. Decide whether to cache an object, response, fragment, query result, negative result, or computation.

## 3. Identify the Source of Truth

State which system wins after disagreement:

```text
Product DB         -> authoritative product data
Redis/application  -> derived cached representation
```

If the cache accepts writes before the source, define the durability and authority boundary explicitly. If deleting the cache loses business state, it is not merely disposable.

## 4. Define the Freshness Requirement

Replace “eventually consistent” with an operation-specific contract:

- maximum time or version staleness;
- read-your-writes requirement;
- monotonic-read requirement;
- whether stale values may be served on error;
- which critical decisions must revalidate against the source.

Examples:

```text
Profile photo: minutes of staleness may be acceptable
Product details: 30 seconds for display; checkout revalidates
Account balance: display may lag; mutation uses ledger authority
Permission grant/revocation: cache only with explicit policy-version semantics
```

## 5. Choose Cache Location

Options include client/browser, CDN/edge, gateway, local L1, distributed L2, or a hybrid.

Ask:

- Can the request be eliminated at the client or edge?
- Is shared state required across instances?
- Can copies disagree briefly?
- Does the working set fit per process?
- Is another network dependency justified?
- What tenant/privacy boundary applies?

See [Local vs Distributed Cache](local-vs-distributed-cache.md).

## 6. Choose the Cache Key

Include every dimension that changes the result:

```text
tenant + entity + locale + permission-version + representation-version
```

Avoid under-keying that leaks data and over-keying that destroys reuse. Decide whether keys reveal PII or secrets in logs and dashboards. Plan key/schema evolution across rolling deployments.

## 7. Choose the Read/Write Strategy

- **Cache-aside:** application loads misses and invalidates after source writes.
- **Read-through:** cache abstraction owns the loader.
- **Write-through:** abstraction synchronously persists cache/source according to a defined contract.
- **Write-back:** acknowledgment precedes asynchronous source persistence and needs a durability/replay design.
- **Refresh-ahead:** proactively maintain known hot values.

Choose from access pattern and acknowledgment semantics, not familiarity.

## 8. Choose TTL and Invalidation

Start with the stale-data consequence. Decide:

- absolute or sliding expiry;
- soft and hard TTL;
- TTL jitter;
- invalidate-on-write or update-on-write;
- event/CDC/outbox propagation;
- entity versions or versioned keys;
- negative-cache TTL;
- reconciliation after missed updates.

TTL is a repair bound, not proof of current data.

## 9. Set Memory and Eviction Policy

Estimate:

```text
keys + values + metadata + allocator overhead
+ replication + fragmentation + headroom
```

Match policy to workload:

- LRU for recency-predictive access;
- LFU for stable popularity;
- FIFO/random when simplicity fits;
- no eviction only with explicit population-failure and state semantics.

Separate pools when object size, importance, or tenant behavior causes interference. Consider admission control for scans and large low-reuse values.

## 10. Protect the Miss Path

Design for one key and for the whole cache:

- per-key request coalescing;
- TTL jitter;
- bounded stale serving;
- refresh-ahead;
- origin concurrency limits;
- backpressure and load shedding;
- controlled prewarming;
- negative/error caching where safe.

Calculate maximum origin QPS during a flush or cluster restart, not only steady-state miss QPS.

## 11. Design Cache Failure Behavior

For each endpoint, choose deliberately:

```text
bypass within origin budget
serve bounded-stale data
degrade optional functionality
fail open
fail closed
shed load
```

Examples:

- Product catalog may serve bounded-stale data and shed personalization.
- Rate limiting may fail open for low-risk APIs and fail closed for abuse-sensitive operations.
- Authorization should not serve a stale allow decision beyond the policy contract.

Test slow cache, partial shard failure, full outage, cold start, and recovery.

## 12. Design Observability

Track metrics that connect to the decision:

```text
hit and miss ratio by key class
cache and miss latency p50/p95/p99
lookup errors/timeouts
origin QPS and latency caused by misses
memory, fragmentation, evictions, expirations
connection-pool saturation
hot-key and per-shard distribution
regeneration time and coalescing waiters
invalidation/event lag and stale-version rejection
stale-served and fallback traffic
backend load during cache failure
```

A falling hit rate matters primarily if it raises meaningful origin load, cost, or user latency. Define alarms on the consequence.

## 13. Define Reconsideration Triggers

State when to change or remove the cache:

- hit rate or origin offload remains too low;
- database optimization removes the bottleneck;
- freshness requirements tighten;
- invalidation defects exceed the performance benefit;
- working set no longer fits economically;
- one tenant/key/shard dominates;
- cache cost exceeds saved source cost;
- outage recovery cannot meet the reliability objective.

Architecture should evolve from measured behavior.

## Reusable Examples

### Product catalog

> The workload is read-heavy and repeated product-detail joins are database-CPU bound. I would use cache-aside in a distributed cache because product display tolerates approximately 30 seconds of staleness. The product database remains authoritative. Writes commit there and invalidate a versioned object. TTL jitter and per-key coalescing protect popular expirations, while checkout revalidates current price and inventory. I would reconsider the cache if source optimization meets the target or invalidation defects exceed its value.

### User profile

Use a tenant-aware key and decide whether the session needs read-your-writes. Profile presentation can use bounded staleness; authorization roles should use their own stricter version contract rather than sharing a generic profile cache.

### Rate-limit metadata

Distinguish cached policy/configuration from the mutable counters that enforce limits. Cached policy can use versions; counter state may be authoritative in an in-memory store and therefore needs a separate availability, atomicity, and fail-open/fail-closed design.

### Feed objects

Cache immutable post objects by version and keep personalized ranking separately. A broad per-user feed-result cache may have low reuse and costly invalidation.

### Configuration

Small, frequently read configuration may fit an L1 cache with version notifications and periodic refresh. Security-sensitive revocation needs a strict propagation SLO and safe failure policy.

### Expensive query results

Cache only if query parameters form a safe key and reuse is high. Query-result invalidation may fan out broadly; a materialized view can be a better fit when correctness and rebuild behavior need to be explicit.

## Cases Where Caching Is Wrong

```text
Unique one-time reads
        -> no useful reuse

Slow unindexed query
        -> fix the source first

Strict authoritative decision
        -> validate source, perhaps cache display only

Rapid writes + little read reuse
        -> invalidation churn dominates

Huge low-reuse values
        -> memory/network cost exceeds savings

Unsafe tenant/authorization key
        -> correctness and security risk
```

## Failure Review

Before closing an interview answer, walk through:

```text
single key expires
many keys expire
cache returns slowly
one shard fails
entire cache fails
source fails during miss
invalidation event is delayed or reordered
deployment changes value format
traffic shifts to one tenant/key
cache recovers cold
```

For each, state the user-visible behavior and which component is protected.

## Interview Questions

### What is the first caching question?

Which bottleneck and repeated work the cache should reduce. Without that, placement and product selection are premature.

### What is the most important reliability question?

Whether the authoritative system survives cold-cache or bypass traffic.

### What proves the cache is successful?

The target latency, source utilization, throughput, or cost improves within the allowed freshness and reliability envelope.

## 2-Minute Interview Answer

> “I begin with the limiting operation and prove there is reuse. Then I name the authoritative store and define the stale window and any read-your-writes or monotonic-read requirement. I choose the closest safe cache location, a tenant- and version-aware key, and a read/write strategy. TTL and invalidation are selected from the freshness contract; memory and eviction come from the useful working set. I coalesce hot misses and cap total origin concurrency so flushes and outages cannot cause a database collapse. Failure behavior—stale serving, bypass, degradation, fail-open/closed, or shedding—is per operation. Finally, I monitor cache and miss tail latency, origin QPS, memory/evictions, key skew, invalidation lag, and fallback load, and I state the metric or requirement that would cause me to redesign or remove the cache.”

## Senior-Level Follow-ups

- Which part of this framework changes for multi-region writes?
- How do you estimate the useful working set before production?
- Which consistency guarantees are user-session-specific?
- How do you roll out a cache without creating a hidden source dependency?
- What is the decision if cache hit rate is high but database CPU remains unchanged?
- How do you audit caching of PII or authorization-sensitive responses?

## References

- [Internal: System Design Decision Frameworks](../interview/decision-frameworks.md)
- [Internal: 45-Minute System Design Framework](../interview/45-minute-framework.md)
- [Internal: Database Selection Framework](../interview/database-selection-framework.md)
- [Internal: Trade-off Analysis](../fundamentals/trade-off-analysis.md)
- [AWS Builders' Library — Caching Challenges and Strategies](https://aws.amazon.com/builders-library/caching-challenges-and-strategies/)

