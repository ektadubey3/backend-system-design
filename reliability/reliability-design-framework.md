# Reliability Design Framework

## TL;DR

Reliability design follows the user's critical journey through objectives, dependencies, capacity, failure containment, degradation, recovery, and evidence. Use this as an interview checklist and an operational review.

## Framework

### 1. Define the critical journey

Name the user intent and terminal success, including asynchronous completion. Choose an SLI, SLO window/target, and correctness threshold. Distinguish essential from optional features.

### 2. Map dependencies and failure domains

Draw the synchronous critical path, async continuations, data authorities, control plane, third parties, zones, and regions. Find shared pools and single points of failure.

### 3. Quantify capacity and headroom

Estimate peak arrival rate, fan-out, service time, concurrency, queue age, payload/storage growth, and failover capacity. Identify the first bottleneck and the tested knee where latency rises faster than useful throughput.

### 4. Bound work

Propagate deadlines and cancellation. Bound queues, in-flight concurrency, connections, payloads, and fan-out. Make operations idempotent and give one layer a finite jittered retry budget.

### 5. Contain failure

Bulkhead tenants, priority classes, and dependencies. Use scoped circuit breakers to fail fast. Protect recovery/control traffic and prevent health checks from competing with saturated work.

### 6. Define overload and degradation

Prioritize, throttle, defer, or reject before collapse. State the cheaper truthful fallback per feature and whether sensitive operations fail closed.

### 7. Recover data and service

Set RPO/RTO per capability. Define failover ownership, old-writer fencing, backup/restore, derived-state rebuild, backlog drain, reconciliation, and failback.

### 8. Observe and rehearse

Use SLO burn for user impact and saturation/cause metrics for diagnosis. Test dependency latency, retry storms, capacity loss, replica lag, zone/region evacuation, and restore. Feed post-incident learning into limits and runbooks.

## Failure matrix

| Failure | Prevent amplification | Preserve correctness | Recover |
|---|---|---|---|
| dependency slow | deadline, circuit, concurrency cap | ambiguous-result status/idempotency | half-open probes and ramp |
| demand spike | bounded queue, rate limit, shedding | explicit rejection/defer state | autoscale and controlled drain |
| instance/zone loss | headroom, bulkhead | durable replicated authority | reroute/rebuild |
| poison async work | capped retry, quarantine | idempotent transition | repair and targeted replay |
| corrupt data | stop writers | immutable backup/log | point-in-time restore and validate |
| region loss | no cross-region critical coupling | ownership epoch and fencing | promote, reconcile, fail back |

## Two-minute answer template

“The critical journey is [X], measured by [good event / eligible event] with SLO [Y]. The synchronous path depends on [A/B], while [C] is asynchronous. At peak, [resource] saturates first, so queues and concurrency are capped at [policy] with [headroom]. Deadlines propagate; one layer retries idempotently with jitter and a finite budget. Dependencies and tenants have separate pools and circuits. On overload we protect [priority] and degrade [feature] honestly. Data uses [replication] across [domains], with RPO/RTO [objectives] proven by [drill]. I alert on burn rate and monitor saturation, retries, lag, shedding, and recovery margin.”

## Interview follow-ups

- What happens if the fallback dependency is also slow?
- Can all clients retry at once after recovery?
- Does the surviving zone have capacity for failed-zone traffic?
- Which metric proves users, rather than hosts, are healthy?
- When did the team last restore the authoritative data?

## Further study

- [Messaging design framework](../messaging/messaging-design-framework.md)
- [Distributed-systems design framework](../distributed-systems/distributed-systems-design-framework.md)
- [Observability design framework](../observability/observability-design-framework.md)

