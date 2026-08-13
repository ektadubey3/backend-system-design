# Cache Stampede

A cache stampede, or thundering herd, occurs when many requests simultaneously discover the same valuable item is unavailable and independently regenerate it. The cache may have enough capacity; the origin fails because miss work is not coordinated.

## Interview TL;DR

1. A popular-key expiry can convert one logical refresh into thousands of origin requests.
2. Request coalescing/single-flight is usually the first per-key control.
3. TTL jitter spreads different keys' expirations but does not serialize one hot key's rebuild.
4. Stale-while-revalidate keeps users off the miss path when bounded staleness is safe.
5. Distributed locks add contention, lease, availability, and fencing concerns.
6. Bound total regeneration concurrency and shed load so the origin survives.
7. Design recovery after a full flush, cluster restart, rebalancing, or deployment—not only normal expiry.

## Mental Model

```text
Popular key expires
        |
        v
Many requests miss simultaneously
        |
        v
All regenerate or read source
        |
        v
Origin overload
```

Amplification is:

```text
logical refreshes needed = 1
source calls made        = N concurrent callers
```

The purpose of stampede protection is to make source work proportional to keys needing refresh, not requests observing the miss.

## Mitigation Techniques

### Request coalescing / single-flight

Within a process or service scope:

```text
first request -> loader
other requests -> wait for same result
```

This is often the strongest first control. Bound waiter count and wait time; if the loader fails, avoid releasing every waiter into an immediate retry storm.

Process-local coalescing still permits one loader per application instance. For large fleets, that may be enough or may require broader coordination.

### Stale-while-revalidate

After soft expiry:

```text
serve bounded-stale value
        +
one asynchronous refresh
```

This avoids blocking callers and protects the origin. It is inappropriate when stale data violates correctness.

### TTL jitter

```text
TTL = base TTL +/- random jitter
```

Use it to prevent a batch of different keys from expiring at the same moment. It does not prevent concurrent callers from stampeding one key.

### Refresh-ahead

Refresh known hot keys before expiry. Trigger from remaining TTL, traffic, or a schedule. Limit background work so refresh traffic cannot become its own overload source.

### Bounded regeneration concurrency

Use both per-key and global limits:

```text
per-key single-flight
+ global origin semaphore/budget
```

This protects the origin when many distinct keys miss together.

### Backpressure and load shedding

If regeneration demand exceeds safe capacity, reject or degrade lower-priority traffic early. Accepting work that will time out consumes more resources and delays recovery.

### Prewarming

Load a proven hot set before sending full traffic to a new cache or region. Do not load the complete database blindly; that can overload the origin and fill memory with cold values.

### Distributed locking

A short-lived per-key lock can designate one rebuilder across instances:

```text
acquire rebuild lock
  +-- owner -> load and populate
  +-- loser -> wait, serve stale, or retry with budget
```

Downsides include lock service dependency, contention, lease expiry, orphaned work, and ambiguous ownership. If the lock protects a stateful external side effect rather than merely duplicate computation, use fencing or authoritative version checks so an expired owner cannot act after a successor.

Do not answer every stampede question with “use a distributed lock.” Duplicate idempotent reads may be cheaper and safer than global locking.

## Architecture

A resilient miss path:

```text
cache miss
   |
   +--> stale value allowed? -- yes --> serve + schedule refresh
   |
   v no
single-flight owner?
   +-- waiter --> wait within deadline
   +-- owner ---> acquire origin budget
                     |
                     +-- unavailable --> degrade/shed
                     +-- available ----> load, version-check, populate
```

The cache and the coalescing mechanism need separate timeouts and metrics.

## When To Use Each Control

- Use local single-flight for most application-level hot misses.
- Add TTL jitter for batch-created keys.
- Use stale-while-revalidate where a bounded stale response is safe.
- Refresh ahead for predictable, persistently hot keys.
- Use distributed coordination only when per-instance duplicate loads remain unsafe.
- Always include global origin protection for full-cache events.

## Scaling Characteristics

For `I` application instances, process-local coalescing can still produce approximately `I` source calls for one key. Decide whether the source can absorb that number during peak traffic.

Full cache loss is a different scale:

```text
many distinct keys x one/few rebuilds each
```

Origin concurrency limits, priority, gradual traffic ramp, and prewarming matter more than a per-key lock alone.

## Failure Modes

### Loader failure

All waiters receive an error and immediately retry. Apply backoff/jitter, short negative/error caching where appropriate, and retry budgets.

### Lock expires before rebuild completes

A second owner begins. For duplicate reads this may be acceptable; for stateful work use fencing/version checks.

### Lock holder crashes

Waiters need bounded lease expiry and a retry path. A long lease delays recovery; a short lease increases duplicate work.

### Stale serving exceeds the safe window

A failed refresh should not silently extend a hard correctness deadline.

### Complete flush or restart

Every key is cold. Rate-limit warm-up and prioritize high-value endpoints instead of bypassing all traffic to the origin.

### Rebalancing

Key ownership changes lower hit rate. Treat planned topology changes as controlled cold-cache events.

## Trade-offs

- Waiting for one loader reduces origin work but adds queueing latency.
- Serving stale improves availability but weakens freshness.
- Proactive refresh smooths latency but consumes background capacity.
- Distributed locking reduces duplicate work but adds coordination failure modes.
- Load shedding protects recovery but reduces request availability deliberately.

## Production Gotchas

- Use the request deadline to bound coalescing wait.
- Track active loaders, waiters per key, coalescing ratio, regeneration latency, lock wait, stale-served count, and origin QPS.
- Limit retry fan-out after a shared loader failure.
- Version-check before a slow rebuild overwrites a newer value.
- Test hot-key expiry, many-key mass expiry, cache flush, rolling restart, cache outage, and slow origin.
- Ensure user-supplied keys cannot create unbounded lock/coalescer cardinality.

## Alternatives

- Eliminate expiry for immutable, versioned content.
- Precompute or materialize the expensive result.
- Cache safely at a CDN or client to remove application demand.
- Increase source capacity only if the miss path remains the real bottleneck after coordination.

## Interview Questions

### Is TTL jitter enough?

No. It spreads different keys' expiry times but many callers can still miss the same hot key together.

### Should I use a distributed lock?

Only if cross-instance duplicate regeneration is unsafe enough to justify lease and availability complexity. Local single-flight plus source limits may be sufficient.

### What happens after a total cache flush?

The design progressively warms a priority hot set, bounds origin concurrency, sheds optional work, and monitors recovery rather than allowing unrestricted bypass.

## 2-Minute Interview Answer

> “A stampede happens when one popular expiry creates many concurrent source reads. I start with per-key single-flight so one loader runs and other requests wait within their deadlines. Where bounded staleness is safe, I use a soft TTL to serve stale while one refresh runs; jitter spreads different keys' expirations, and refresh-ahead handles predictable hot keys. I also cap total origin concurrency because a full flush creates misses across many keys. I use a distributed lock only when one loader per application instance still exceeds the source budget, and I account for lease expiry and fencing if work changes state. Recovery is tested after flush, restart, and rebalancing.”

## Senior-Level Follow-ups

- What happens to waiters when the single loader times out?
- How many source calls occur with 500 application instances?
- When does duplicate regeneration cost less than coordination?
- How do you prevent a stale rebuilder from overwriting a newer value?
- Which traffic is shed first during a full-cache recovery?

## References

- [AWS Builders' Library — Caching Challenges and Strategies](https://aws.amazon.com/builders-library/caching-challenges-and-strategies/)
- [RFC 5861 — stale-while-revalidate and stale-if-error](https://www.rfc-editor.org/rfc/rfc5861.html)
- [Internal: Fault Tolerance and Resilience](../fundamentals/fault-tolerance.md)
- [Internal: TTL and Expiration](ttl-and-expiration.md)
- [Internal: Hot Keys](hot-keys.md)

