# Cache-Aside

Cache-aside keeps the authoritative store outside the cache and makes the application responsible for reading, populating, and invalidating cached values. It is common because it is operationally simple, not because it eliminates consistency races.

## Interview TL;DR

1. On a read miss, the application reads the source and populates the cache.
2. On a write, update the authoritative store first and then invalidate the cache.
3. Only requested values enter the cache, which reduces unused cached writes.
4. Misses pay both cache and source latency; cold starts can amplify origin load.
5. Database update and cache invalidation are not normally one atomic transaction.
6. An old reader can repopulate stale data after a successful write and invalidation.
7. Mitigation depends on the stale-data consequence: TTL, versions, events, comparison, or source revalidation.
8. Delayed double invalidation narrows selected race windows but is not a universal correctness proof.

## Mental Model

Read path:

```text
GET key
  |
  v
Cache?
  +-- hit --> return
  |
  +-- miss
        |
        v
     Database
        |
        v
     Cache SET
        |
        v
      Return
```

Typical write path:

```text
Update database
       |
       v
Invalidate cache
```

The database remains authoritative. Cache deletion is usually safer than constructing and writing a new cached representation in the mutation path because deletion avoids some dual-write and serialization coupling. It still leaves a window in which the cache may be stale.

## How It Works

Conceptual read logic:

```text
value = cache.get(key)
if value is usable:
    return value

value = source.read(id)
cache.set(key, value, ttl)
return value
```

Production logic also needs:

- cache timeout and error handling;
- single-flight/request coalescing;
- negative-cache behavior;
- value and schema version validation;
- a policy for cache-population failure;
- bounded source concurrency.

The cache population step is normally best-effort. A successful source read should not necessarily fail because cache `SET` failed, but failures must be observable because sustained population failure changes origin load.

## Write Ordering

Prefer:

```text
1. commit authoritative update
2. invalidate cache
```

If deletion happens first:

```text
delete cache
reader misses
reader reads old database value
reader repopulates it
writer commits database
```

The cache may now retain stale data.

Database-first ordering avoids that specific window, but it cannot make two independent systems atomic:

```text
database commit succeeds
process crashes before invalidation
```

TTL, reliable events, retries, or reconciliation must bound or repair that divergence.

## The Stale Repopulation Race

Even database-first invalidation has a race:

```text
Reader A misses cache
Writer B updates database
Writer B invalidates cache
Reader A reads old database value
Reader A repopulates stale value
```

The exact possibility depends on transaction isolation, replicas, request timing, and which source Reader A uses. Do not claim a single ordering solves every race.

### Mitigation options

#### Short, requirement-driven TTL

Bounds how long a stale repopulation survives. It is suitable when bounded staleness is acceptable, not when the value controls a strict invariant.

#### Versioned values

Store the source version with the representation:

```text
{ entity_id: 42, version: 18, payload: ... }
```

Readers or cache writers can reject a value older than an observed watermark. This requires a trustworthy monotonic version within the entity's ownership boundary.

#### Conditional population

Before `SET`, compare the loaded version with a newer known version or use a cache operation that refuses to overwrite a newer entry. The cache and source still need well-defined version semantics.

#### Event-driven invalidation

Publish committed changes through CDC or a transactional outbox. Consumers invalidate or update cached versions. Durable events close a process-crash gap, but consumers must handle delay, duplicates, ordering, and replay.

#### Delayed/double invalidation

Invalidate once around the write and again after a delay chosen to outlast a known race window. This can reduce stale repopulation in specific architectures. It is not a universal solution because delays are not correctness boundaries, requests can exceed the assumed window, and the second delete can fail.

#### Do not cache the critical decision

Use cached data for display but revalidate inventory, authorization, balance, or uniqueness against the authoritative system before mutation.

## Architecture

Separate cache policy from domain code where possible, but keep the semantics visible:

```text
Application service
  +-- key/version policy
  +-- cache client
  +-- source repository
  +-- origin concurrency limiter
  +-- metrics
```

Avoid a generic cache wrapper whose callers cannot tell whether stale data, negative values, or fallback are possible.

## When To Use It

Cache-aside fits when:

- reads significantly exceed writes;
- values have useful temporal locality;
- the authoritative store remains safe and independently recoverable;
- first-read miss latency is acceptable;
- bounded staleness can be defined;
- application ownership of cache logic is acceptable.

It is often a good initial pattern for product details, profiles, public configuration, feed objects, and expensive query results—with different key, freshness, and invalidation policies for each.

## When Not To Use It

Avoid or modify it when:

- immediate read-after-write visibility is mandatory across all readers;
- writes are so frequent that entries are constantly invalidated;
- the first miss cannot tolerate source latency;
- every request is unique;
- the source cannot survive cold-cache traffic and no protection exists;
- stale data can make an authoritative decision incorrectly.

## Scaling Characteristics

Cache-aside loads only requested data, so it naturally follows the observed working set. That is efficient until a deploy, cache flush, topology change, or traffic shift makes many keys miss together.

For a hot missing key, amplification can be:

```text
one logical cache miss
        |
        v
N concurrent source reads
```

Coalesce by key and cap total source concurrency. For large objects, include serialization, transfer, and population bandwidth in capacity estimates.

## Failure Modes

### Cache unavailable

Fallback must be bounded. Use short cache timeouts, source concurrency limits, stale local copies where safe, degraded responses, or load shedding.

### Source unavailable on miss

Choose whether to serve a bounded-stale value, negative/error cache briefly, degrade, or fail. Repeatedly discarding the same failure can hammer an unhealthy origin.

### Invalidation fails

Retry asynchronously, rely on a bounded TTL, use durable invalidation events, and alert on invalidation failure or event lag.

### Negative-cache staleness

A newly created object can remain hidden. Use a shorter negative TTL and invalidate the negative key on creation.

### Cache pollution

Low-reuse reads populate values that evict the hot set. Add admission rules or bypass scan-like traffic.

## Trade-offs

- Lazy population stores only requested data but makes the first reader pay miss latency.
- Invalidate-on-write keeps mutation logic simpler but creates temporary misses and stale windows.
- Update-on-write may improve read-after-write behavior but adds more dual-write and representation coupling.
- Short TTL bounds drift but raises origin traffic.
- Durable event invalidation improves repairability but adds messaging, lag, and consumer operations.

## Production Gotchas

- Include tenant, locale, permission, and representation version where they change the value.
- Treat malformed cached data as a miss and record the reason.
- Make source reads and cache population idempotent.
- Do not put secrets or raw tokens in keys.
- Track hit/miss latency, coalesced waiters, cache errors, population failures, invalidation failures, origin QPS, and stale-version rejections.
- Load-test with the cache disabled and after a full flush.
- Do not let cache population extend the client deadline after the source response is ready.

## Alternatives

- Read-through moves the loading responsibility into a cache abstraction.
- Write-through synchronously maintains the cached representation during writes.
- Refresh-ahead can avoid first-reader miss latency for known hot keys.
- A materialized view may provide more explicit correctness and rebuild semantics.
- No cache may be simpler if the source meets the target.

## Interview Questions

### Why is cache-aside common?

It keeps the source authoritative, stores only demanded data, and allows the cache to be rebuilt. The application can also choose fallback per operation.

### Why update the database before deleting the cache?

Deleting first permits a reader to miss, read the old database value, and repopulate it before the database commit.

### Does database-first deletion guarantee consistency?

No. Invalidation can fail, a reader can have observed old source state, or a replica can lag. Define versions, TTL, events, and critical revalidation according to the requirement.

## 2-Minute Interview Answer

> “For this read-heavy workload I would use cache-aside because the database remains authoritative and only requested values consume cache memory. Reads check the cache, coalesce concurrent misses, read the source, and populate a versioned value. Writes commit the database first and then invalidate the cached representation. That ordering avoids one common stale-repopulation window but is not atomic, so I bound residual staleness with a requirement-driven TTL and, where needed, publish committed changes through an outbox or CDC. The cache has a short timeout, origin fallback is concurrency-limited, and critical mutations revalidate against the source. I monitor population and invalidation failures, source QPS from misses, stale-version rejection, and recovery after a flush.”

## Senior-Level Follow-ups

- How does replica lag change the stale-repopulation race?
- When is delayed double invalidation useful, and what can it not prove?
- How do you invalidate query-result keys when one entity changes?
- What is the negative-cache policy on object creation?
- How do you prevent 10,000 cold misses after a deploy?
- Which read-your-writes flows bypass or repair the cache?

## References

- [Microsoft Azure Architecture Center — Cache-Aside Pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/cache-aside)
- [AWS Builders' Library — Caching Challenges and Strategies](https://aws.amazon.com/builders-library/caching-challenges-and-strategies/)
- [Internal: Data Consistency Boundaries](../databases/consistency-boundaries.md)
- [Internal: Cache Invalidation](cache-invalidation.md)
- [Internal: Cache Stampede](cache-stampede.md)

