# Cache Invalidation

> Cache invalidation is a data-consistency problem, not merely a delete operation.

An authoritative write and a cache change usually cross independent systems. Without a shared transaction, any order leaves failure windows that must be bounded, detected, and repaired.

## Interview TL;DR

1. Name the authoritative store and the cached representations derived from it.
2. TTL-only bounds residence but permits staleness until expiry.
3. Invalidate-on-write is simple, but database commit and cache delete are not atomic.
4. Update-on-write can keep hot values warm but introduces partial-success and ordering risks.
5. Events improve reliability only when publication, consumption, versioning, replay, and reconciliation are designed.
6. Versioned keys turn many invalidations into a change of namespace, at the cost of cleanup and version lookup.
7. Duplicate invalidations are usually harmless; out-of-order updates are not.
8. Critical decisions should revalidate against the authority when no stale window is safe.

## Mental Model

Invalidation coordinates these operations:

```text
database write
cache update
cache delete
event publication
cache repopulation
```

There is no safe design merely because all operations appear in the happy path. Ask what happens after every individual step succeeds and the next one fails.

## Invalidation Strategies

### TTL-only

Let entries expire without explicit write notification.

```text
source changes -> old value remains until TTL
```

Benefits: simple and self-healing for forgotten keys. Costs: staleness lasts up to the effective expiry window, and short TTL raises source load.

Use when the stale bound is acceptable and update fan-out is not worth the complexity.

### Invalidate-on-write

```text
commit source update
        |
        v
delete cached representation
```

The next read repopulates from the source. This avoids constructing cached views on the write path, but creates a miss and can race with old readers.

### Update-on-write

```text
commit source update
        |
        v
write new cached representation
```

This keeps values warm and can improve read-after-write visibility. It couples mutation code to cache representation and must prevent an older update or retry from overwriting a newer version.

### Event-driven invalidation

```text
source transaction
  +-- business state
  +-- outbox record
          |
          v
publisher / CDC
          |
          v
idempotent version-aware cache consumer
```

This handles writers that do not share cache code and can close the process-crash gap after commit. The event path is asynchronous, so freshness depends on event lag and recovery.

### Versioned cache keys

```text
product:42:v17
product:42:v18
```

Readers discover or already know the current version and fetch an immutable representation. This prevents old data from overwriting a new namespace. Costs include version discovery, orphan cleanup, and increased key churn.

### Generation or namespace prefixes

```text
catalog:g81:item:42
```

Increment a generation to invalidate a group. Useful when enumerating all affected keys is impractical. A broad generation change can create a synchronized cold cache, so protect the origin.

### Explicit purge

An operator or deployment purges one key, tag, URL, tenant namespace, or entire cache. Purge APIs need authentication, auditability, bounded scope, and protection from accidental mass invalidation.

## Failure Matrix

### Database update succeeds; invalidation fails

The old value remains. Mitigate with retry, TTL, durable outbox/CDC, alerting, and reconciliation.

### Invalidation event is lost

At-least-once delivery, durable offsets, replay, or reconciliation is needed. TTL can bound exposure but does not prove prompt convergence.

### Consumer is delayed

Event lag becomes the stale window. Measure event age and define a consistency SLO.

### Duplicate event arrives

Delete operations are often naturally idempotent. Update operations need an event ID or version so duplicates cannot apply side effects repeatedly.

### Events arrive out of order

```text
version 18 arrives
version 17 arrives later
```

Blind update regresses the cache. Apply only newer versions or invalidate rather than update when ordering cannot be trusted.

### Cache update succeeds; database transaction rolls back

Readers can observe data that never committed. Do not expose cache state before the authoritative commit unless the architecture explicitly makes the cache authoritative.

### Old reader repopulates after invalidation

Store versions and reject stale population, invalidate again through a reliable event, or accept a bounded TTL where safe. See [Cache-Aside](cache-aside.md).

## Event-Driven Requirements

Event-driven invalidation requires:

- durable event creation through an outbox or equivalent when the write/event gap matters;
- idempotent consumers;
- entity versions or sequence awareness;
- lag, failure, and dead-letter observability;
- replay after consumer outage;
- reconciliation to detect drift outside normal retry windows;
- schema evolution and delete/tombstone handling.

“Publish an event” is not a consistency strategy until these behaviors are defined.

## When To Use Each Strategy

- Use TTL-only for low-risk, bounded-stale data with simple operations.
- Use invalidate-on-write when the writer knows affected keys and miss latency is acceptable.
- Use update-on-write when immediate warm reads justify representation coupling.
- Use events when many writers or derived views need reliable propagation.
- Use versioned keys for immutable representations or deploy/config generations.
- Revalidate against the authority for decisions that cannot tolerate stale state.

Hybrid strategies are common:

```text
invalidation for prompt freshness
+ TTL for repair bound
+ version for ordering safety
+ reconciliation for drift
```

## Scaling Characteristics

One source write may affect many cached keys:

```text
product update
  +--> product object
  +--> category query pages
  +--> search suggestions
  +--> regional representations
```

This fan-out can make query-result caching expensive to invalidate. Prefer stable object keys, versioned aggregates, or broader generation changes when enumeration is unsafe—while planning for the resulting miss burst.

## Failure Modes

- invalidation storm overwhelms cache or message consumers;
- broad purge creates origin overload;
- one poison event blocks a partition;
- cache key catalog misses a derived representation;
- version counter resets or collides;
- replayed old update overwrites current data;
- delayed security-policy invalidation leaves access open.

Failure behavior should follow the invariant. Stale display data may be served; authorization and critical mutations may fail closed or read the source.

## Trade-offs

- TTL-only is simple but less prompt.
- Deletes are easy and idempotent but create miss latency.
- Updates keep values warm but require stronger ordering and representation ownership.
- Events improve decoupling and repair but add asynchronous lag and operations.
- Versioned keys avoid in-place overwrite races but consume cleanup capacity.
- Broad namespace invalidation is simple to issue but can trigger a cold-start event.

## Production Gotchas

- Map each source entity to every representation that derives from it.
- Never include raw secrets or unnecessary PII in invalidation keys/events.
- Authenticate purge endpoints and audit broad invalidations.
- Monitor invalidation success/failure, event lag, oldest unprocessed event, replay rate, stale-version rejection, and origin QPS after purge.
- Make cache update consumers conditional on source version.
- Reconcile sample or full projections against the authoritative store.
- Exercise lost, delayed, duplicate, and reordered events in tests.

## Alternatives

- Avoid caching mutable data with unsafe stale consequences.
- Cache immutable, content-addressed representations.
- Use a materialized view with explicit refresh and rebuild semantics.
- Read the source for critical operations while caching display projections.

## Interview Questions

### Why not just delete the key after every write?

That is a useful strategy, but deletion can fail, a process can crash after commit, and an old reader can repopulate stale data. Add bounds and repair.

### Does an event bus solve invalidation?

Only if event creation is coupled safely to commit and consumers handle lag, duplicates, ordering, replay, and reconciliation.

### Why use versioned keys?

They prevent old writers from replacing a new representation and make cached payloads immutable. Version discovery and garbage collection remain.

## 2-Minute Interview Answer

> “I treat invalidation as propagation from one authoritative version to derived copies. For a simple object cache I commit the database first and invalidate the key, with TTL as a repair bound. If invalidation must survive a writer crash or many services write the entity, I record an outbox event in the source transaction and consume it idempotently. Cached values and events carry monotonic entity versions so duplicates are harmless and old events cannot overwrite newer state. I monitor propagation lag and invalidation failures, support replay, and reconcile drift. For correctness-sensitive decisions I still validate against the authoritative store rather than claiming invalidation makes two systems atomic.”

## Senior-Level Follow-ups

- How do you invalidate an unbounded query-result set?
- What happens when a namespace generation changes at peak traffic?
- How do you prove a permission revocation reached every L1 cache?
- Which source version works across sharded writers?
- How does reconciliation distinguish a stale cache from an old source replica?

## References

- [Microsoft Azure Architecture Center — Cache-Aside Pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/cache-aside)
- [Internal: Data Consistency Boundaries](../databases/consistency-boundaries.md)
- [Internal: Cache-Aside](cache-aside.md)
- [Internal: Cache Consistency](cache-consistency.md)

