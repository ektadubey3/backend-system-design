# Bottleneck Identification

A bottleneck is the resource or coordination point that constrains the system from meeting its throughput, latency, or capacity objective.

Do not scale from intuition. Form a bottleneck hypothesis, measure it, change one constraint, and measure again.

## Interview TL;DR

1. Rank workload by **frequency × cost × criticality**.
2. Look for saturation, queueing, lock wait, I/O wait, and downstream throttling.
3. High latency does not always mean high CPU.
4. A pool can hide a bottleneck by moving waiting into the application.
5. Increasing concurrency after saturation often increases tail latency.
6. Scale the constrained resource; scaling an unconstrained tier changes nothing.
7. Re-test after every optimization because the bottleneck moves.
8. Include hot keys/tenants and failure traffic, not only averages.

## Workflow

```text
define SLO
   ↓
measure end-to-end
   ↓
split cost by component
   ↓
identify saturation/queue
   ↓
form hypothesis
   ↓
change one thing
   ↓
load test / observe
```

## RED and USE

For request-driven services:

- Rate
- Errors
- Duration

For resources:

- Utilization
- Saturation
- Errors

These are starting frameworks, not complete observability strategies.

## CPU

Signals:

- sustained CPU;
- runnable queue;
- profile hot spots;
- throughput scales with CPU then flattens.

Possible fixes:

- algorithm/data-structure improvement;
- less serialization/compression work;
- caching computation;
- more cores/nodes.

## Memory / GC

Signals:

- allocation pressure;
- frequent/long GC;
- swap/page faults;
- OOM;
- unbounded cache.

## Storage / Database

Signals:

- I/O wait;
- rows examined ≫ returned;
- lock waits;
- connection queue;
- WAL/redo saturation.

Possible fixes:

- query/index changes;
- smaller transactions;
- batching;
- cache;
- appropriate replicas;
- partitioning when needed.

## Network

Signals:

- bandwidth saturation;
- retransmission/loss;
- RTT;
- handshake cost;
- egress throttling.

## Locks / Coordination

```text
CPU not saturated
but p99 high
and requests are waiting
```

Investigate hot DB rows, global locks, leaders, metadata services, or shard hotspots.

## Connection Pools

If:

```text
pool size = 100
all busy
queue growing
```

do not immediately increase it.

Ask whether the downstream can sustainably execute more concurrent work. The pool is an admission-control boundary.

## Queues

Track:

```text
oldest message age
arrival rate
completion rate
retry rate
consumer utilization
```

Depth alone is insufficient.

## Hotspots

Inspect:

```text
top tenant share
top key share
largest partition
top endpoint
```

## Little's Law

For a stable system:

```text
L = λ × W
```

Use as a model, then validate.

## Common Mistakes

- “database is slow, add Redis” before query analysis;
- “CPU is only 40%, so there is capacity” while locks/I/O saturate;
- increasing thread/connection pools indefinitely;
- optimizing one slow query instead of highest total cost;
- ignoring retries during incidents;
- benchmarking uniform keys while production is skewed.

## Interview Answer Template

> “I define the SLO, identify the first saturating shared resource, and inspect request rate/error/duration plus utilization, queueing and lock wait. I do not increase concurrency blindly: if the DB is saturated, a larger pool only moves the queue. After each fix I re-measure because the bottleneck moves.”

## References

- [Google SRE — Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/)
