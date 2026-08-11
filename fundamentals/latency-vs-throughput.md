# Latency vs Throughput

Latency and throughput describe different dimensions of performance.

- **Latency**: how long one operation takes.
- **Throughput**: how much work completes per unit time.

Senior-level reasoning also includes **tail latency, concurrency, queueing, saturation, and backpressure**.

## Interview TL;DR

1. Use percentiles (`p50`, `p95`, `p99`), not only averages.
2. Throughput normally plateaus when a constrained resource saturates; latency can then rise sharply because work queues.
3. Batching often improves throughput while increasing per-item latency.
4. Increasing concurrency helps only until the constrained resource is saturated.
5. Fan-out makes tail latency important.
6. Set an end-to-end latency budget and allocate it across dependencies.
7. Protect the system from unbounded queues with deadlines, concurrency limits, load shedding, and backpressure.

## Mental Model

```text
arrival rate
    ↓
queue
    ↓
service capacity
    ↓
completed work
```

When sustainable arrival rate approaches or exceeds service capacity:

```text
queue grows
latency rises
timeouts rise
retries may amplify load
```

## Latency

Useful components:

```text
DNS
+ connection establishment
+ TLS
+ queueing
+ application work
+ downstream calls
+ database/cache work
+ serialization
+ network transfer
```

Do not assign universal “good” or “bad” millisecond values.

## Tail Latency

Example:

```text
p50 = 40 ms
p95 = 90 ms
p99 = 900 ms
```

The average can look acceptable while the tail is poor.

Fan-out amplifies this effect:

```text
request
 ├─ dependency A
 ├─ dependency B
 ├─ dependency C
 └─ dependency D
```

The response may wait for the slowest required branch.

## Throughput

Examples:

```text
HTTP requests/sec
events/sec
messages/sec
rows/sec
bytes/sec
transactions/sec
```

Higher throughput is useful only if correctness and latency targets still hold.

## Concurrency and Little's Law

For a stable system:

```text
L = λ × W
```

- `L`: average work in the system;
- `λ`: average arrival/completion rate;
- `W`: average time in system.

Example:

```text
2,000 requests/sec × 0.1 sec ≈ 200 concurrent requests
```

This is a planning model, not a substitute for load testing.

## Saturation

Watch the constrained resource:

- CPU;
- DB connections;
- thread/event-loop capacity;
- disk IOPS;
- network;
- lock contention;
- queue consumers;
- downstream quotas.

Typical shape:

```text
load ↑
throughput ↑
resource saturates
throughput flattens
queue + p99 latency ↑
```

## Batching

Potential gain:

- fewer round trips;
- better sequential I/O;
- less per-item overhead.

Cost:

- waiting to fill a batch;
- larger failure unit;
- more memory;
- burstier downstream work.

## Latency Budget

If:

```text
API p99 target = 300 ms
```

allocate a budget across edge, app, database, dependencies, and safety margin. Exact values are assumptions; the exercise exposes where the budget can be consumed.

## Overload

Use:

- bounded queues;
- request deadlines;
- load shedding;
- backpressure;
- admission control;
- concurrency limits;
- priority classes.

A timeout without cancellation can still waste server work after the caller has given up.

## Retry Amplification

```text
client retries 3×
gateway retries 2×
service retries 2×

one logical request can create up to 12 downstream attempts
```

Retries must fit inside a deadline and retry budget.

## Common Mistakes

- optimizing averages while p99 fails;
- increasing pools after the downstream is saturated;
- calling a high-throughput system “fast” without discussing latency;
- using unbounded queues to hide overload;
- ignoring retries in capacity estimates;
- assuming more concurrency always increases throughput.

## Interview Answer Template

> “I’ll define a percentile latency target, estimate peak throughput and concurrency, and identify the first saturating resource. I’ll keep queues bounded and propagate deadlines so overload becomes controlled rejection rather than unbounded latency. I may trade latency for throughput with batching on non-interactive paths.”

## References

- [Google SRE — Handling Overload](https://sre.google/sre-book/handling-overload/)
- [Google SRE — Addressing Cascading Failures](https://sre.google/sre-book/addressing-cascading-failures/)
