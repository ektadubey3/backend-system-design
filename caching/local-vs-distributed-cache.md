# Local vs Distributed Cache

Cache placement determines latency, sharing, coherence, memory efficiency, and failure scope. A distributed cache is not universally better than an in-process cache; it exchanges one set of problems for another.

## Interview TL;DR

1. Use a local cache when the value is small, extremely hot, and short-lived staleness across instances is acceptable.
2. Use a distributed cache when instances need shared reuse, centralized invalidation, or more aggregate capacity.
3. Autoscaling multiplies local caches, memory duplication, and cold starts.
4. A distributed cache adds network, serialization, connection-pool, cluster, and hot-partition failure modes.
5. L1 plus L2 can reduce extreme read traffic, but every additional level creates another stale copy.
6. Choose placement from freshness and workload distribution, not from a product preference.

## Mental Model

### Local/in-process cache

```text
App instance A --> private cache A
App instance B --> private cache B
App instance C --> private cache C
```

Examples include an in-process LRU, application memoization, and a small per-instance configuration cache.

### Distributed cache

```text
App A --\
App B ----> shared cache cluster ----> authoritative store
App C --/
```

Redis and Memcached are common implementations, but the architecture is “shared remote cache,” not “use Redis.”

## How It Works

### Local cache characteristics

Benefits:

- memory-speed lookup with no network hop;
- no remote connection or serialization on a hit;
- failure is usually isolated to one process;
- simple for deterministic, process-local computation.

Costs:

- different instances can return different versions;
- the same value consumes memory in every process;
- each restart or scale-out event creates a cold cache;
- invalidation must reach every relevant instance;
- origin load may grow with application fleet size.

### Distributed cache characteristics

Benefits:

- values and memory capacity are shared across application instances;
- one instance can reuse a value populated by another;
- invalidation has a central target;
- the cache can be scaled and operated separately.

Costs:

- every lookup incurs a network hop and serialization work;
- connection pools, DNS, routing, and timeouts enter the request path;
- cache-cluster failures affect many application instances;
- one key or shard can be hot even when total capacity is healthy;
- failover or rebalancing can increase miss rate and origin load.

Centralized state makes invalidation easier to target, not automatically correct. Old readers and reordered writes can still repopulate stale values.

## Architecture

### Hybrid L1 + L2

```text
Request
  |
  v
L1: local cache
  |
 miss
  v
L2: distributed cache
  |
 miss
  v
Database / authoritative service
```

This is useful when extremely hot reads make distributed-cache traffic itself a bottleneck. L1 might use a short TTL while L2 retains a larger shared working set.

The cost is a multi-level coherence problem:

```text
source updated
      |
      +--> invalidate L2
      |
      +--> expire or invalidate every L1
```

If L1 invalidation is best-effort, the business stale window must include its TTL. A request can also move between instances and observe a newer value followed by an older one.

## When To Use Each

Prefer local caching when:

- values are process-specific or cheap to reconstruct;
- the hottest data set is small;
- microsecond-level lookup matters;
- duplicate memory is acceptable;
- short per-instance inconsistency is acceptable;
- a remote cache would cost more than the avoided work.

Prefer distributed caching when:

- many instances request the same keys;
- the useful working set exceeds practical per-process memory;
- origin offload must not grow with application fleet size;
- centralized invalidation and shared warm state are valuable;
- the team can operate or consume a reliable cache service.

Do not use either when the source already meets the target, reuse is low, or cached staleness cannot be made safe.

## Scaling Characteristics

For local caches, total memory is approximately:

```text
per-instance cache x application instance count
```

For a distributed cache, aggregate capacity can scale by sharding, but traffic distribution matters more than key count alone. Connection counts also grow with application instances, so pool size and cache-node client limits need explicit budgets.

A hybrid architecture can reduce L2 QPS, but L1 hit rate should be evaluated against added invalidation complexity and the stale window—not just latency.

## Failure Modes

### Local cache failures

- a rolling deploy makes many instances cold;
- one instance retains stale authorization or tenant data;
- an unbounded map causes process memory pressure;
- a large scan pollutes every instance independently.

### Distributed cache failures

- slow lookups consume request threads before fallback;
- a cluster outage redirects fleet-wide traffic to the origin;
- failover returns older replicated state depending on the cache semantics;
- a hot shard saturates while fleet averages remain healthy;
- connection storms delay recovery.

### Hybrid failures

L1 can mask an L2 outage for hot entries, but it can also continue serving values that should have been invalidated. Decide whether that is an intentional availability policy or a correctness bug.

## Trade-offs

| Concern | Local | Distributed | L1 + L2 |
|---|---|---|---|
| Hit latency | Lowest | Network-dependent | Lowest for hottest keys |
| Shared reuse | No | Yes | Yes through L2 |
| Memory duplication | High across fleet | Lower | Both |
| Invalidation | Fan-out/expiry | Central target | Multi-level |
| Cold start | Per instance | Cluster-wide events | L1 per instance |
| Failure scope | Usually one process | Fleet-wide dependency | More modes |

## Production Gotchas

- Bound local cache memory; do not rely only on application heap headroom.
- Add representation versions so rolling deployments can reject incompatible values safely.
- Keep local and distributed hit/miss metrics separate.
- Monitor L2 connection-pool saturation, timeouts, per-shard load, and fallback origin QPS.
- Include tenant and authorization dimensions in keys at every level.
- Encrypt remote cache traffic and restrict network and credential access when sensitive data is cached.
- Test scale-out, rolling restart, L2 outage, and fleet-wide cold start.

## Alternatives

- HTTP or CDN caching can remove the request before it reaches the application.
- Request-scoped memoization can eliminate duplicate work without cross-request staleness.
- A read replica or optimized source may serve diverse reads better than a cache.
- Precomputed projections provide a more explicit update and rebuild model.

## Interview Questions

### Why not use only local caches?

They duplicate memory, warm independently, and can disagree. They are excellent when those costs fit the freshness model.

### Why not always use a distributed cache?

It adds a network dependency and cluster operations. For small process-local data, it can make the path slower and less reliable.

### When is L1 + L2 justified?

When measured L2 latency or QPS is material, the hottest set fits locally, and the business accepts the extra coherence model.

## 2-Minute Interview Answer

> “I choose cache placement from reuse and freshness. An in-process cache gives the lowest latency but each instance has a separate copy, so memory is duplicated, scale-out creates cold caches, and users can observe inconsistent versions. A distributed cache shares capacity and warm state across the fleet, but it adds network, serialization, connection, sharding, and cluster failure modes. I use L1 plus L2 only when distributed-cache traffic is itself a measured bottleneck and a short L1 stale window is acceptable. In all cases I bound memory, version the representation, protect the origin during cold start or outage, and measure each layer separately.”

## Senior-Level Follow-ups

- How do you invalidate L1 across thousands of instances?
- Can a user observe version 10 and then version 9 after load balancing?
- How does autoscaling change origin QPS?
- What is the origin budget if L2 fails while all L1 caches are cold?
- Which data may remain in L1 during a control-plane outage?

## References

- [AWS Builders' Library — Caching Challenges and Strategies](https://aws.amazon.com/builders-library/caching-challenges-and-strategies/)
- [Memcached documentation — client/server architecture](https://docs.memcached.org/)
- [Internal: Redis](../databases/nosql/redis.md)
- [Internal: Cache Consistency](cache-consistency.md)
- [Internal: Distributed Caching](distributed-caching.md)

