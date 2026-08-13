# TTL and Expiration

TTL is a policy for how long a cached entry may remain usable. It is not proof that the value is fresh, and it is not a consistency guarantee.

## Interview TL;DR

1. Begin with maximum acceptable staleness, not “how often the row changes.”
2. Balance freshness, memory occupancy, origin load, regeneration cost, and operational simplicity.
3. Use jitter when synchronized expiration could create a thundering herd.
4. Soft and hard TTLs can permit bounded stale serving while one worker refreshes.
5. Sliding expiration can retain popular entries indefinitely, so it is unsuitable when age since source validation must be bounded.
6. Negative entries usually need a shorter TTL because missing data can later appear.
7. Combine TTL with invalidation or version checking when the business stale bound is stricter.

## Mental Model

Suppose a value is cached at time `t0` for 300 seconds:

```text
t0: source version 10 cached
t0 + 1s: source becomes version 11
t0 + 300s: cache entry expires
```

The cache can serve version 10 for almost five minutes after the source changes. TTL bounds residence after cache population; it does not detect source changes.

Choose TTL by balancing:

```text
Freshness
Memory occupancy
Origin load
Regeneration cost
Operational simplicity
```

## How It Works

### Absolute TTL

The entry expires a fixed duration after it is inserted or refreshed.

```text
insert -------------------- expire
          fixed duration
```

This gives a simple upper bound from the last successful population, assuming the cache enforces expiration as documented.

### Sliding expiration

Each access extends the expiry window.

```text
read -> extend -> read -> extend -> read -> extend
```

It retains popular entries and removes idle ones. It does not bound age since validation: a continuously read stale entry can survive indefinitely unless it is refreshed separately.

### Soft TTL and hard TTL

```text
fresh until soft TTL
        |
        v
serve stale + refresh in background
        |
        v
hard TTL: stop serving without successful revalidation
```

This is useful when a bounded-stale response is safer than sending every request to a struggling origin. It requires one refresh owner, observability, and a defined hard limit.

### Stale-while-revalidate

The request may receive a stale value within an allowed window while revalidation happens asynchronously. HTTP defines this behavior through `stale-while-revalidate`; application caches can adopt the concept but must implement their own concurrency and freshness contract.

### Stale-if-error

Where the business permits it, a previously valid value may be served for a bounded period when the origin fails. This improves availability but deliberately weakens freshness. Never apply it blindly to permission grants, balances used for decisions, or mutable security policy.

### Refresh-ahead

Refresh known hot entries before they expire:

```text
remaining TTL below threshold
          |
          v
one background refresh
```

It can stabilize hot-key latency but refreshes data that might not be requested again. Ownership, rate limits, and failure retry budgets are required.

### Negative-cache TTL

Caching `NOT_FOUND` can protect an origin from repeated invalid IDs or dependency errors. Use a separately chosen TTL because a resource may be created shortly after the negative result.

## Choosing a TTL

Ask in this order:

1. What business consequence follows from serving an old value?
2. What is the maximum stale window for this operation?
3. Is invalidation expected, and how can it fail?
4. How expensive is regeneration?
5. Can the origin absorb the resulting steady-state and burst miss rate?
6. Does value size make long residency uneconomical?
7. Is stale serving acceptable during an origin outage?

Example:

```text
Product description: minutes may be acceptable
Feature configuration: seconds, with version checks for critical flags
Negative product lookup: short, because a product may be created
Authorization revocation: TTL alone may be unsafe
```

Do not publish a universal TTL or jitter percentage. Requirements and traffic shape determine both.

## Architecture

Combine expiry mechanisms deliberately:

```text
source update
   |
   +--> best-effort invalidation for prompt freshness
   |
   +--> TTL as a repair bound for missed invalidation
```

For multi-level caching, the end-to-end stale window includes every level. An L1 entry with 10 seconds remaining can still serve old data after L2 has been refreshed.

## Scaling Characteristics

In a simplified workload with read rate `R` and entries refreshing every `T`, shorter TTL generally raises regeneration and origin load. The relationship is not linear when popularity is skewed, entries are evicted early, or requests coalesce.

Spread non-semantic expiry times:

```text
TTL = base TTL +/- random jitter
```

Jitter reduces synchronized expiration; it does not protect the first request for each individual hot key. Pair it with request coalescing or refresh-ahead where needed.

## Failure Modes

### Mass expiration

Keys populated by a batch or deployment expire together, causing a coordinated origin surge.

### Refresh failure

A soft-expired entry remains stale. Decide whether to retry within a budget, serve until hard TTL, or fail.

### Sliding-expiry immortality

A popular value stays resident without source revalidation.

### Negative-cache staleness

A newly created object remains hidden behind an old `NOT_FOUND` value.

### Clock and implementation semantics

Absolute expiry behavior, clock dependence, and lazy versus active removal vary by cache. Verify the implementation instead of assuming an expired entry immediately releases memory everywhere.

## Trade-offs

- Longer TTL usually improves reuse and reduces origin load, but permits longer staleness and occupies memory longer.
- Shorter TTL improves freshness only probabilistically unless tied to source versions; it raises miss and regeneration traffic.
- Refresh-ahead smooths latency but adds background work and ownership logic.
- Stale serving protects availability and the origin but must be bounded by the business invariant.
- Sliding expiration improves residency for popular entries but weakens age bounds.

## Production Gotchas

- Record why a TTL exists and which requirement it enforces.
- Track expirations/sec, refresh latency, refresh failures, entry age, stale-served count, and origin QPS after expiry.
- Keep hard TTL longer than soft TTL and make the allowed stale interval explicit.
- Apply jitter only to time that is not itself a business deadline.
- Do not extend a security token, reservation, or lease merely because it was read.
- Ensure a failed refresh does not overwrite a newer cached version.

## Alternatives

- Event-driven invalidation gives more prompt freshness but needs durable delivery and repair.
- Versioned keys can move readers to a new immutable representation.
- Revalidate critical decisions against the authoritative store.
- Avoid caching when no safe stale window exists.

## Interview Questions

### How would you choose the TTL?

Start with tolerated staleness and failure consequences, then test whether the resulting origin load, regeneration cost, and memory residency are acceptable.

### Does a five-minute TTL guarantee data is at most five minutes stale?

It bounds time since that entry was populated, not necessarily time since its source snapshot was current. Upstream lag and multi-level caches can widen effective staleness.

### Why add TTL jitter?

To spread expiration work for entries created at similar times. It does not replace per-key stampede protection.

## 2-Minute Interview Answer

> “I choose TTL from the maximum stale window the business accepts, then validate origin load, regeneration cost, and memory occupancy. TTL is a repair bound, not a guarantee that a value stays current after its source changes. For known hot data I may use soft and hard expiry: serve bounded-stale data after soft expiry while one worker refreshes, and stop serving it at hard expiry. I add jitter to non-semantic TTLs so batch-populated keys do not expire together, use a shorter policy for negative entries, and track expiration rate, entry age, refresh failures, stale serving, and resulting origin QPS.”

## Senior-Level Follow-ups

- What is the effective stale bound across L1, L2, and a lagging replica?
- Can sliding expiration keep an incorrect value forever?
- How do refresh workers avoid overwriting a newer version?
- What happens when the origin is unavailable past hard TTL?
- Which deadlines must never receive random jitter?

## References

- [RFC 9111 — HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111.html)
- [RFC 5861 — stale-while-revalidate and stale-if-error](https://www.rfc-editor.org/rfc/rfc5861.html)
- [Redis documentation — EXPIRE](https://redis.io/docs/latest/commands/expire/)
- [AWS Builders' Library — Caching Challenges and Strategies](https://aws.amazon.com/builders-library/caching-challenges-and-strategies/)
- [Internal: Cache Invalidation](cache-invalidation.md)
- [Internal: Cache Stampede](cache-stampede.md)

