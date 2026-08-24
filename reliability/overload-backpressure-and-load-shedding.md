# Overload, Backpressure, and Load Shedding

## TL;DR

Overload begins when offered work exceeds a bottleneck's sustainable capacity. Unbounded queues turn it into rising latency and resource exhaustion. Bound admission and concurrency, propagate pressure upstream, prioritize important work, and fail early and cheaply before useful throughput collapses.

## Queueing tells the story

Little's Law relates average concurrency `L`, throughput `lambda`, and time in system `W`:

```text
L = lambda * W
```

As a saturated server accepts more concurrent work without increasing completion rate, time in system grows. That longer latency causes caller timeouts and retries, adding more offered load. The objective under overload is not 100% acceptance; it is maximum useful completions with bounded latency.

## Bound every scarce resource

Identify the first resource to saturate: CPU, memory, worker slots, connections, file descriptors, disk IOPS, broker partitions, or a downstream quota. Apply:

- bounded request and task queues;
- semaphores for in-flight expensive work;
- connection-pool limits aligned with dependency capacity;
- per-tenant and per-priority concurrency;
- rate limits with burst control;
- maximum payload and fan-out limits.

A pool is admission control, not just a performance optimization. See [Connection pooling](../networking/connection-pooling.md).

## Backpressure

Backpressure tells upstream producers that current demand cannot be accepted at the original rate. It may be explicit—throttling status, retry-after, reduced stream credits—or implicit—bounded queue rejection.

If upstream cannot slow, choose what to discard. Dropping an obsolete metrics sample may be correct; dropping a payment command silently is not. Durable work should receive an explicit rejected/deferred state or land in a bounded durable queue with an age objective.

## Load shedding and prioritization

Reject before expensive parsing, authentication fan-out, or database work when possible. Preserve:

1. control and recovery traffic;
2. safety-critical writes;
3. critical reads;
4. interactive best-effort features;
5. batch and speculative work.

Reserve capacity so low-priority work cannot occupy every worker. Fairness limits one tenant or key from monopolizing the service.

## Autoscaling limits

Autoscaling reacts after measurement and startup delay. It may scale the frontend faster than a fixed database, increasing pressure. Scale on a causal signal such as concurrency, queue age, or work rate, respect downstream capacity, and keep headroom for failover. Load shedding remains necessary even with autoscaling.

## Failure modes

- Infinite queue keeps accepting requests until memory or deadlines collapse.
- Health checks fail under load, removing instances and worsening overload.
- Retry traffic is indistinguishable from first attempts.
- All tenants share one pool, so one noisy neighbor starves others.
- Autoscaler uses CPU while the real bottleneck is a connection pool.
- Shed response triggers immediate client retry and creates a loop.

## Interview prompts

- Which resource saturates first and at what measured limit?
- Where is the queue bounded, and what happens on rejection?
- Which work receives reserved capacity?
- Can autoscaling make the dependency overload worse?

## Two-minute answer

Measure sustainable completion capacity and bound queueing plus in-flight work at the first scarce resource. Propagate pressure through concurrency limits, throttling, and retry-after; reserve pools for critical traffic and isolate tenants. When capacity is exhausted, reject early or serve a cheaper honest response instead of allowing latency and retries to collapse throughput. Scale only within downstream limits and monitor queue age, concurrency, rejection, completion rate, retry ratio, and saturation.

## References

- [Google SRE — Addressing cascading failures](https://sre.google/sre-book/addressing-cascading-failures/)

