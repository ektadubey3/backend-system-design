# Read-Through and Write-Through

Read-through and write-through move data-loading or write propagation behind a cache, library, or platform abstraction. They change who owns the path; they do not eliminate staleness, dual-system failure, or source-of-truth decisions.

## Interview TL;DR

1. Read-through makes the cache abstraction load misses from a configured source.
2. Write-through makes a write return only after the cache layer synchronously persists according to its contract.
3. Cache-aside leaves these responsibilities in application code.
4. Read-through standardizes loading but can hide expensive misses and stampedes.
5. Write-through warms new values but increases write latency and may cache data never read.
6. “Both calls succeeded” is not automatically one atomic transaction.
7. Product support varies; these are architecture patterns, not universal Redis-native behaviors.

## Mental Model

### Read-through

```text
Application
    |
    v
Cache abstraction
    |
   miss
    v
Loader / authoritative store
```

The application asks for a key. The caching component owns miss detection, source loading, and population.

### Write-through

```text
Application
    |
    v
Cache/storage abstraction
    |
    +--> update cache
    |
    +--> synchronously persist to authoritative store
    |
    v
return according to the abstraction's success contract
```

The important question is not the label. It is which component performs each step, in what order, and what the caller can infer after acknowledgment.

## How It Works

### Read-through lifecycle

```text
get(key)
  +-- value present --> return
  +-- absent ---------> invoke loader
                          |
                          v
                       cache result
                          |
                          v
                        return
```

The loader needs the same protections as cache-aside:

- per-key request coalescing;
- source concurrency limits;
- negative-result policy;
- TTL and version semantics;
- timeout and error behavior;
- value size and serialization bounds.

Read-through can reduce duplicated application code, but it can also make a harmless-looking `get` perform an expensive database or cross-region request.

### Write-through lifecycle

Possible implementations include:

```text
cache first -> source second
source first -> cache second
one integrated storage platform coordinating both
```

These have different failure windows. Ask:

- Is the authoritative store committed before acknowledgment?
- What happens if cache update succeeds and source persistence fails?
- What happens if source commit succeeds and cache update fails?
- Is rollback possible, or is later repair required?
- Can readers observe an uncommitted cached value?

Do not infer atomicity from the name “write-through.” Read the actual platform or library contract.

## Comparison With Cache-Aside

| Concern | Cache-aside | Read-through | Write-through |
|---|---|---|---|
| Miss owner | Application | Cache/loader abstraction | Depends on read path |
| Write owner | Application/source | Usually application/source | Cache/storage abstraction |
| Population | Lazy on reads | Lazy on reads | Eager on writes |
| App code | Explicit | Simpler reads | Simpler write call |
| Hidden coupling | Lower | Loader/source coupling | Cache/source coupling |
| Main risk | Stale invalidation and cold misses | Hidden miss cost and loader failure | Write latency and partial success |

Read-through can be implemented by an application library and may look similar to cache-aside internally. The architectural difference is the ownership boundary exposed to callers.

## Architecture

A safe abstraction exposes or documents:

```text
source of truth
load timeout
negative caching
freshness policy
coalescing scope
write acknowledgment point
partial-failure behavior
metrics
```

If callers cannot determine whether a `get` can block on a source or whether a `put` means “durably committed,” the abstraction is operationally dangerous.

## When To Use Them

Read-through fits when:

- many callers need the same loading policy;
- the loader and key schema have clear ownership;
- hiding cache plumbing improves consistency of implementation;
- the miss path can be centrally protected and observed.

Write-through fits when:

- values written are likely to be read soon;
- immediately warming the cache is valuable;
- additional synchronous write latency is acceptable;
- the platform's partial-failure and durability semantics meet the requirement.

## When Not To Use Them

- Do not use read-through if a generic cache layer cannot express domain-specific freshness, tenant security, or fallback.
- Do not use write-through for write-heavy, rarely read values; it pollutes memory.
- Do not use either label as a substitute for defining the source of truth.
- Do not use write-through when callers require an atomic guarantee the implementation does not provide.
- Do not make critical mutations depend on a cache service unless its availability role is intentional.

## Scaling Characteristics

Read-through centralizes loader behavior, so concurrency bugs can affect every caller. A popular expired key still needs single-flight or bounded regeneration.

Write-through adds cache and source work to every write. Estimate:

```text
write latency ~= coordination + cache work + source persistence
```

The actual critical path may be parallel or sequential, but failure recovery and consistency determine whether that optimization is safe.

## Failure Modes

### Read-through loader storm

Many cache misses invoke the loader concurrently and overload the source.

### Hidden timeout

Callers budget for cache latency, but the cache call includes a slow source read.

### Partial write-through success

The source commits but cache update fails, or the cache becomes visible while the source rolls back. The system needs ordering, versions, retry, and repair semantics.

### Cache outage on writes

If write-through requires the cache, cache unavailability may reject otherwise valid source writes. Decide whether that availability coupling is acceptable.

### Unused cached writes

Write-heavy traffic fills memory with values that receive no reads and evicts useful entries.

## Trade-offs

- Abstraction simplifies caller code but can hide latency and failure behavior.
- Eager write population improves immediate read hits but increases mutation cost and cache pollution.
- Central loaders standardize policy but create a shared operational component.
- Stronger coupling can improve read-your-writes behavior while reducing independent source availability.

## Production Gotchas

- Give loaders deadlines and bounded concurrency.
- Make negative-result and error-caching behavior explicit.
- Version cached representations across rolling deployments.
- Trace cache hits separately from reads that invoked a loader.
- Record source commit and cache update outcomes independently.
- Do not cache authorization-sensitive responses without complete identity/policy keying.
- Test cache failure during reads and at every write-through failure boundary.

## Alternatives

- Cache-aside provides explicit application control and often simpler source authority.
- Refresh-ahead handles known hot values without putting every source load behind `get`.
- Write-back reduces apparent write latency but creates a stronger durability problem.
- Direct source reads may be preferable when reuse is low.

## Interview Questions

### How is read-through different from cache-aside?

The cache or loader abstraction owns miss loading; with cache-aside, application code owns it. The underlying freshness and stampede problems remain.

### Does write-through guarantee consistency?

Only to the exact semantics of the implementation. Two independent updates are not inherently atomic, and failure ordering must be defined.

### Why can write-through waste cache memory?

It admits every written value even when many are never read.

## 2-Minute Interview Answer

> “Read-through and write-through describe ownership boundaries. In read-through, the cache abstraction invokes a loader on miss; I still need deadlines, coalescing, negative-result policy, versions, and origin protection. In write-through, the application writes through an abstraction that synchronously updates the cache and source according to a defined contract. I would not claim atomicity without verifying the ordering and partial-failure semantics. Write-through is attractive when newly written data is read soon and immediate warming matters, but it raises write latency and can pollute memory. I would choose it over cache-aside only when that coupling and the acknowledgment semantics match the requirement.”

## Senior-Level Follow-ups

- Can callers distinguish a cache hit from a loader-backed read?
- What does write acknowledgment prove after a cache-node failure?
- How does the system repair source-success/cache-failure?
- Should an unused write enter the cache?
- What happens when the cache is unavailable but the source is healthy?

## References

- [Microsoft Azure Architecture Center — Cache-Aside Pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/cache-aside)
- [AWS — Database Caching Strategies](https://docs.aws.amazon.com/whitepapers/latest/database-caching-strategies-using-redis/caching-patterns.html)
- [Internal: Cache-Aside](cache-aside.md)
- [Internal: Write-Back Cache](write-back-cache.md)

