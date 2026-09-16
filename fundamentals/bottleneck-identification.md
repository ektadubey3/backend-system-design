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

Use [Little's Law and its measurement boundary](latency-vs-throughput.md#concurrency-and-littles-law): `L = λ × W`, where `L` is mean queued plus executing work, `λ` is mean throughput, and `W` is mean time in that same system.

At a stable 2,000 requests/s and 100 ms mean response time, average concurrency is 200. This does not justify a 200-connection DB pool: each request may spend only part of its lifetime using a connection. A growing backlog is not a steady-state sizing measurement.

## Worked Diagnosis — Low CPU, High p99

Assume application CPU is 40%, p99 rose from 150 ms to 1 s, all 100 DB connections are occupied, and completion throughput is flat. These are observations, not proof that the pool is too small.

1. Split end-to-end time into pool wait, query execution, and other spans; compare the affected routes and recent changes.
2. If DB lock wait rose on one inventory row, inspect blocking transactions. Shorten the transaction and remove remote I/O from it; bound admission while recovering. Adding app replicas would add waiters.
3. If DB execution is healthy and capacity testing shows spare headroom, a carefully increased pool may help. Include the **sum of pools across instances**, not just one process.
4. Replay the same workload shape, including hot keys, at the same offered load. Verify goodput, tail latency, rejection, lock wait, and downstream saturation. Roll back if useful completions fall or the SLO worsens.

A load generator that waits for each response before sending the next request reduces offered load as the service slows. For arrival-driven traffic, also test a controlled arrival schedule and record generator lag or missed sends; otherwise the test can hide queueing.

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
