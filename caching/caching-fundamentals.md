# Caching Fundamentals

Caching stores reusable data or computation closer to a request path so repeated work can be avoided. The senior-level question is not whether a cache is fast. It is:

> **Is the cache actually reducing the bottleneck that limits the system?**

A cache can reduce latency, origin load, network traffic, and repeated computation. It also consumes memory, adds lookup overhead, permits stale reads, complicates invalidation, and creates new failure modes.

## Interview TL;DR

1. Start with a measured or projected bottleneck, not a cache product.
2. Cache only work with enough reuse to justify lookup, memory, and consistency costs.
3. Name the source of truth and the maximum acceptable staleness.
4. Design the cache key as a correctness and tenant-isolation boundary.
5. Measure hit and miss paths separately; hit rate alone is insufficient.
6. Protect the authoritative system from cold starts, mass misses, and cache outages.
7. Define what happens when a cached value is stale, malformed, missing, or unavailable.
8. Remove the cache if it no longer materially improves the limiting resource or user-facing latency.

## Mental Model

A typical cache-assisted read is:

```text
Request
  |
  v
Cache lookup
  +-- Hit --> return cached value
  |
  +-- Miss --> authoritative system
                    |
                    v
                 populate cache
                    |
                    v
                  return
```

This creates two modes:

```text
warm mode: most requests use the fast path
cold mode: many requests use the expensive path
```

Production safety depends on both. A design validated only with a warm cache has not validated its failure path.

The cache is usually derived state:

```text
authoritative store = decides the correct value
cache               = reconstructable representation
```

If losing the cache loses acknowledged business data, the component is acting as an authoritative store or write buffer. Its durability, replication, recovery, and acknowledgment semantics must be designed explicitly.

## What Caching Can Improve

### Latency reduction

A local memory lookup or nearby cache request can avoid a slower database query, cross-region call, object-store read, or external API request. The benefit must be evaluated at p50, p95, and p99; a faster average can coexist with a worse miss tail.

### Origin/load reduction

Repeated reads can be collapsed into one source read followed by many cache hits. This can reduce database CPU, I/O, connections, rate-limited dependency calls, and infrastructure cost.

### Expensive computation reuse

Candidates include aggregations, rendering, authorization metadata construction, serialization, and stable query results. Cache only when inputs can be represented safely in the key and reuse is likely.

### Increased read scalability

A cache can move read traffic away from a constrained store, but it does not remove bottlenecks. The limiting resource may move to cache network bandwidth, a hot key, one shard, serialization CPU, connection pools, or the invalidation pipeline.

## Performance Model

Define:

```text
hit rate  = cache hits / total lookups
miss rate = 1 - hit rate
```

For total read rate `R`, a simplified steady-state model is:

```text
origin read rate ~= R x miss_rate
```

For latency:

```text
Expected latency
≈
(hit_rate × cache_latency)
+
(miss_rate × miss_path_latency)
```

The miss path normally includes the failed cache lookup, source request, serialization, cache population, and sometimes coordination with other miss waiters.

These models are useful for a first estimate, but they hide important behavior:

- traffic and miss cost are not uniformly distributed across keys;
- concurrent misses can amplify one logical miss into many source calls;
- averages hide p99 latency and correlated failures;
- errors, timeouts, stale responses, and bypasses may not be counted consistently;
- cache population consumes write bandwidth and CPU;
- a cache outage changes hit rate abruptly rather than gradually.

A 99% hit rate can still be poor if the remaining 1% executes the most expensive queries or transfers the largest objects. Conversely, a lower hit rate may be valuable if it removes the origin's hottest CPU-bound query.

Measure the outcome the cache exists to improve:

```text
cache hit latency
cache miss latency
origin QPS caused by misses
origin CPU / I/O / connections avoided
cache lookup overhead
cache memory footprint
byte hit ratio
stale-read rate, where measurable
evictions and expirations
backend amplification per logical miss
```

## Cache Keys and Granularity

A cache key defines which requests may share a value. Include every dimension that changes the representation, such as:

```text
tenant
entity ID
locale
currency
permission or policy version
API/schema version
feature variant
```

Under-keying can return the wrong user's or tenant's data. Over-keying fragments reuse and wastes memory. Avoid secrets and unnecessary PII in keys because keys appear in logs, traces, and dashboards.

Choose granularity deliberately:

- whole responses reduce reconstruction work but widen invalidation scope;
- objects often align with source versions and business ownership;
- fragments improve reuse but require more lookups and composition;
- query-result caches can be valuable but may have high-cardinality keys and broad invalidation.

## Architecture

Caches may exist in browsers, CDNs, gateways, application processes, distributed cache clusters, database engines, or storage clients. Each layer has different ownership, security, and failure semantics.

```text
Client / edge
      |
      v
Application L1 cache
      |
      v
Distributed L2 cache
      |
      v
Authoritative store
```

More levels reduce some latency and load but create more copies to expire or invalidate. See [Local vs Distributed Cache](local-vs-distributed-cache.md) before adopting a multi-level design.

## When To Use It

Caching is a strong candidate when:

- reads or computations repeat within a useful reuse window;
- the avoided operation is expensive or capacity-limited;
- values have a clear source of truth;
- the business permits a defined stale window or reliable revalidation;
- the useful working set fits economically in memory;
- the miss and outage paths can protect the origin.

Example:

> Product-detail reads have high temporal locality and repeated database joins consume most read CPU. Product metadata may be 30 seconds stale. A cache can target both p99 latency and database CPU, provided popular-key misses are coalesced and checkout still validates authoritative price and inventory.

## When Not To Use It

Avoid or remove a cache when:

- access is mostly one-time or scan-like, so reuse is low;
- an index, query fix, or data-model change solves the bottleneck more directly;
- the source already meets latency and capacity requirements;
- values change so frequently that invalidation dominates useful hits;
- stale data can violate an invariant and no safe validation path exists;
- values are too large or sensitive for the available cache boundary;
- cache cost and operational risk exceed origin savings.

Do not use cached authorization-sensitive responses unless the key includes the relevant identity and policy dimensions and revocation semantics are acceptable. Critical mutations should normally revalidate against an authoritative system.

## Scaling Characteristics

Capacity planning should target the useful working set rather than the complete source dataset:

```text
memory budget
~=
keys + values + metadata + allocator overhead
+ replication + fragmentation + headroom
```

Track item-size distribution, not only average size. A few large values can dominate memory and network bandwidth.

As traffic grows, ask:

- Is cache CPU, memory, network, or connection count limiting?
- Does key distribution match traffic distribution?
- Is one tenant or key class evicting another?
- Does adding application capacity create more independent cold caches?
- Can the authoritative system absorb rebalancing or full-cache loss?

## Failure Modes

### Cold start or flush

Many entries are missing at once, so origin QPS rises while the cache warms. Prewarm only the known hot set, rate-limit regeneration, and test recovery at production-like scale.

### Cache outage

Blind bypass can cause:

```text
cache unavailable
      |
      v
all requests hit origin
      |
      v
origin saturation
      |
      v
cascading failure
```

Use short cache timeouts, bounded origin concurrency, load shedding, degraded responses, or bounded stale serving according to the data's correctness requirements.

### Stale or incorrect values

Missed invalidation, out-of-order events, bad key dimensions, or old readers can return incorrect data. Version entries and make critical flows validate against the authority.

### Miss amplification

Many concurrent requests for one absent key may each call the source. Use request coalescing, bounded concurrency, and the techniques in [Cache Stampede](cache-stampede.md).

### Memory pressure

Evictions can lower hit rate, which raises source load and may create an eviction/miss feedback loop. Isolate workloads with different value sizes or business importance when they interfere.

### Poisoned or incompatible entries

Deployment changes can make older serialized values unreadable. Include a representation version, tolerate old formats during rolling deploys, and avoid treating mass discard as harmless—it can create a cold-cache event.

## Trade-offs

| Decision | Gain | Cost |
|---|---|---|
| Longer residency | More reuse, lower origin load | Longer potential staleness, more memory |
| Local cache | Lowest latency, no remote dependency | Duplicate memory and incoherent copies |
| Distributed cache | Shared capacity and reuse | Network hop and cluster failure modes |
| Cache final response | Avoids most work | Larger invalidation and security surface |
| Cache small components | More targeted reuse | More lookups and composition logic |
| Serve stale on failure | Higher availability, origin protection | Explicitly weaker freshness |

The correct choice is per operation. Serving a stale product description may be acceptable; serving a stale permission grant may not be.

## Production Gotchas

- Set cache timeouts well below the end-to-end request deadline.
- Distinguish hit, miss, bypass, timeout, stale-served, and population failure in telemetry.
- Bound value size and serialization/decompression cost.
- Keep tenant identity and authorization-sensitive dimensions in the key.
- Encrypt sensitive cached data in transit and restrict access to shared caches.
- Test warm, cold, partially unavailable, fully unavailable, high-eviction, and hot-key modes.
- Roll out gradually while comparing origin load and latency against an uncached control.
- Define a metric or requirement that would cause the cache to be removed.

## Alternatives

Before caching, consider:

- indexes and query optimization;
- read replicas for diverse low-reuse reads;
- materialized views or purpose-built projections;
- denormalization with explicit update semantics;
- batching and connection-pool fixes;
- CDN or HTTP caching for safe public responses;
- a source store better matched to the access pattern.

Caching reuses prior work. It should not indefinitely hide an unsuitable source design.

## Interview Questions

### Why add a cache here?

Name the repeated operation, limiting resource, reuse pattern, and tolerated staleness. “To make it fast” is incomplete.

### Is a 95% hit rate good?

Only if it materially improves the target bottleneck without unacceptable stale reads, memory cost, or miss-path risk. Inspect miss cost, byte hit ratio, origin load, and tail latency.

### What happens when the cache is down?

Choose a bounded policy: source fallback within a concurrency budget, stale serving, feature degradation, load shedding, or failure. Do not assume the origin can accept all traffic.

### What belongs in the key?

Every dimension that changes the returned representation, including tenant and relevant authorization or schema versions.

## 2-Minute Interview Answer

> “I add a cache only after identifying repeated work and the limiting resource. I name the authoritative store, define the key and acceptable stale window, and estimate whether the useful working set can produce meaningful origin offload. For latency I model both hit and miss paths, but I validate p99 latency, miss cost, origin QPS, memory, evictions, and stale-read indicators rather than relying on average hit rate. I choose local or distributed placement and a read/write strategy from the workload. The critical reliability case is a cold or unavailable cache, so origin calls are concurrency-limited and popular misses are coalesced; depending on the invariant I may serve bounded-stale data, degrade, shed load, or fail. I would remove the cache if it no longer reduces the bottleneck enough to justify consistency and operational complexity.”

## Senior-Level Follow-ups

- How would you prove that the cache reduced database CPU rather than only moving latency?
- What changes if the hit rate falls from 95% to 60%?
- Can a high request hit rate coexist with a poor byte hit rate?
- How does a deployment change the cache key or serialized value safely?
- Which flows may serve stale data during an outage?
- How do you prevent cross-tenant or authorization-sensitive cache leakage?
- What is the maximum origin QPS during a total cache flush?
- Which measurement would cause you to remove the cache?

## References

- [AWS Builders' Library — Caching Challenges and Strategies](https://aws.amazon.com/builders-library/caching-challenges-and-strategies/)
- [RFC 9111 — HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111.html)
- [Memcached documentation — architecture and design philosophy](https://docs.memcached.org/)
- [Redis documentation — key eviction](https://redis.io/docs/latest/develop/reference/eviction/)
- [Internal: Trade-off Analysis](../fundamentals/trade-off-analysis.md)
- [Internal: Fault Tolerance and Resilience](../fundamentals/fault-tolerance.md)
- [Internal: Redis](../databases/nosql/redis.md)
