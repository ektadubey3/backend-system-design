# Caching

> A cache trades freshness, memory, and operational complexity for lower latency and reduced load on an authoritative system.

Caching is not a product choice made after drawing a database. It is a decision to introduce another copy of data or computation into a request path. That copy can improve latency and capacity, but it also creates consistency, failure, security, and recovery questions.

Keep these distinctions explicit:

```text
Cache ≠ source of truth
Cache hit rate ≠ cache quality
TTL ≠ consistency guarantee
More cache ≠ automatically more scalable
```

The goal of this section is to reason from a measured bottleneck and a business freshness requirement to a cache design that remains safe when entries are stale, missing, evicted, or unavailable.

## Recommended Reading Order

### Foundations

1. [Caching Fundamentals](caching-fundamentals.md) — Decide whether caching addresses the actual bottleneck and learn the core performance model.
2. [Local vs Distributed Cache](local-vs-distributed-cache.md) — Choose where cached state should live and understand the coherence cost.
3. [TTL and Expiration](ttl-and-expiration.md) — Translate a business staleness budget into refresh and expiry behavior.
4. [Eviction Policies](eviction-policies.md) — Match memory-pressure behavior to the workload rather than defaulting to LRU.

### Read/write strategies

5. [Cache-Aside](cache-aside.md) — Keep the authoritative store in control while the application owns population and invalidation.
6. [Read-Through and Write-Through](read-through-and-write-through.md) — Move loading or synchronous persistence behind a cache abstraction and reason about its failure contract.
7. [Write-Back Cache](write-back-cache.md) — Trade write latency for an asynchronous durability and recovery problem.

### Correctness and failure modes

8. [Cache Invalidation](cache-invalidation.md) — Treat invalidation as cross-system data consistency, including lost and reordered updates.
9. [Cache Consistency](cache-consistency.md) — Define concrete guarantees such as bounded staleness, read-your-writes, and monotonic reads.
10. [Cache Stampede](cache-stampede.md) — Prevent synchronized misses from overwhelming the authoritative system.
11. [Hot Keys](hot-keys.md) — Detect and mitigate traffic skew that aggregate capacity metrics hide.

### Distributed systems

12. [Distributed Caching](distributed-caching.md) — Design sharding, replication, routing, failover, rebalancing, and origin protection.
13. [Cache Design Framework](cache-design-framework.md) — Apply a reusable interview decision sequence to real workloads.

## What Interviewers Expect From a Senior Engineer

A strong candidate does not stop at “add Redis.” They can answer:

- Why cache this data or computation?
- Which measured bottleneck should the cache reduce?
- What is the source of truth?
- What dimensions belong in the cache key?
- What consistency or freshness guarantee is required?
- What happens on a cache hit, miss, timeout, and malformed value?
- What happens when the cache is completely unavailable?
- How is invalidation performed, retried, observed, and repaired?
- What protects the origin during cold start, mass expiry, or failover?
- How are hot keys and hot shards handled?
- What is the useful working set and memory budget?
- Which expiration and eviction policies fit the workload?
- Could cached data cross a tenant or authorization boundary?
- Which metrics prove that caching is helping?
- What change would cause the cache to be redesigned or removed?

The expected answer connects each choice to a requirement:

```text
measured bottleneck
        |
        v
cache candidate and source of truth
        |
        v
freshness and key semantics
        |
        v
read/write strategy
        |
        v
miss, invalidation, and failure behavior
        |
        v
metrics and reconsideration trigger
```

## Reliability Lens

The dangerous transition is:

```text
cache outage or cold start
          |
          v
miss/bypass surge
          |
          v
authoritative system overload
          |
          v
wider service outage
```

For every chapter, decide whether a cache problem should cause the application to:

- bypass the cache within an origin concurrency budget;
- serve a bounded-stale value;
- shed optional traffic;
- degrade the feature;
- fail open or fail closed according to the invariant.

Blind fallback is not a reliability strategy. If the origin cannot survive cold-cache traffic, the cache is part of the service's capacity architecture and must be operated and tested accordingly.

## Vocabulary

- **Source of truth / authoritative store:** the system whose state wins when copies disagree.
- **Cache hit:** a lookup returns a usable cached value.
- **Cache miss:** no usable value is available, including absent, expired, evicted, or rejected stale entries.
- **Expiration:** an entry becomes unusable because its lifetime policy ends.
- **Eviction:** an entry is removed because of memory pressure or another admission/replacement policy.
- **Invalidation:** an action marks or removes a cached representation because its source may have changed.
- **Staleness:** the cached representation does not reflect the relevant current source version.

Use these terms precisely. “Eventually consistent” and “high hit rate” are observations, not complete correctness or performance requirements.

## Related Curricula and Practice

- [Database consistency boundaries](../databases/consistency-boundaries.md) for deciding which state remains authoritative.
- [Distributed Systems](../distributed-systems/README.md) for generic replication, partitioning, quorum, and ownership concepts.
- [Reliability Engineering](../reliability/README.md) for overload control when a cache or origin fails.
- [URL Shortener](../case-studies/01-url-shortener.md) and [News Feed](../case-studies/05-news-feed.md) for cache-heavy design exercises.
