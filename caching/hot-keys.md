# Hot Keys

Hot keys are a traffic-distribution problem: total cache capacity can be healthy while one key, tenant, or partition saturates a single node or network path.

## Interview TL;DR

1. Even key distribution does not imply even traffic distribution.
2. Celebrity content, viral events, large tenants, and poor hashing can create hot keys or shards.
3. Adding cache nodes does not split one key's traffic unless the routing or representation changes.
4. Local L1 caching, replication, request coalescing, and edge caching are strong read mitigations.
5. Key splitting helps only when semantics permit independent partitions or replicated copies.
6. Rate limiting and load shedding protect the system when demand cannot be served safely.
7. Measure top-key and per-shard load; fleet averages hide skew.

## Mental Model

```text
Total cache capacity: healthy
Shard A:              healthy
Shard B:              overloaded by one key
Shard C:              healthy
```

Examples:

- a celebrity profile or viral post;
- a homepage configuration read by every request;
- one tenant generating a large percentage of traffic;
- a global rate-limit counter;
- many logical keys hashing to one partition;
- a write-heavy key serialized through one owner.

Distinguish:

```text
data skew    = bytes or key count distributed unevenly
traffic skew = operations or bandwidth distributed unevenly
```

A small key can be a severe traffic hotspot.

## Detection

Aggregate hit rate and memory usage are insufficient. Track:

```text
requests per key or sampled heavy hitters
top-key percentage of traffic
per-shard QPS and bandwidth
latency/error rate by shard
CPU/network/connections by node
miss rate by key class
tenant-level traffic and memory
rebuild frequency and waiter count per key
```

Exact per-key metrics may have unbounded cardinality. Use sampling, top-K/heavy-hitter algorithms, bounded labels, or cache-native diagnostics.

## Mitigations

### Local L1 caching

Each application instance serves the hottest object locally, reducing traffic to the distributed cache.

```text
many app requests -> one local copy per instance
```

This is effective for reads when a short per-instance stale window is acceptable. It creates duplicate memory and multi-copy invalidation.

### Read replication

Serve the same key from multiple replicas if the cache implementation and freshness model permit it. Replication distributes read capacity but introduces replica lag, routing, and failover behavior.

### Request coalescing

Collapse simultaneous misses or refreshes so the origin does not become the next hot component.

### Edge or CDN caching

For public, safely cacheable content, moving the hot value to edge caches can reduce both cache-cluster and application traffic. Authorization-sensitive responses require correct HTTP cache controls and keying.

### Key splitting or controlled replication

Create multiple copies:

```text
hot:item:42:replica:0
hot:item:42:replica:1
hot:item:42:replica:2
```

Readers select a copy. This can distribute immutable or bounded-stale reads, but invalidation must reach every copy and a writer cannot usually split one logical serialization point safely.

Alternatively, split aggregatable state, such as counters, into partial keys and combine results. This weakens immediacy or adds read aggregation; it is not valid for every invariant.

### Precomputation and fan-out reduction

If every request performs the same expensive assembly, precompute the object once. Avoid application fan-out that multiplies one hot request into many downstream calls.

### Rate limiting and load shedding

Protect cache nodes and the origin from unbounded demand. Shed optional traffic or serve a degraded static value rather than allowing saturation to cascade.

## Architecture

Choose mitigation by operation:

```text
hot immutable read
  -> CDN / replicas / L1

hot mutable read with bounded staleness
  -> short L1 + replicated L2 + versioned invalidation

hot atomic write/counter
  -> partitionable approximation, regional budgets, or authoritative owner

hot miss/regeneration
  -> coalescing + stale serving + origin budget
```

Adding generic nodes helps only if traffic can route to them. One fixed key mapped to one primary may remain limited by that node's CPU or network.

## Scaling Characteristics

Average load is misleading under a Zipf-like popularity distribution. Capacity-plan for:

- hottest key QPS and bytes/sec;
- largest tenant and burst multiplier;
- read/write mix on that key;
- replication fan-out and invalidation bandwidth;
- cache miss behavior if the key expires;
- recovery when a hot-key owner fails.

Replicating a hot value improves read capacity but increases write and invalidation fan-out. L1 reduces central reads but multiplies stale copies.

## Failure Modes

### Hot key expires

The hotspot moves from cache to origin. Use soft expiry, refresh-ahead, and coalescing.

### Hot shard fails

Failover concentrates traffic on a replica that may be cold or smaller. Ramp traffic and preserve headroom.

### Replicas diverge

Readers observe different versions. Bound staleness and use version-aware routing where monotonic reads matter.

### Viral traffic changes faster than autoscaling

Predefined overload controls and edge caching act faster than provisioning alone.

### One tenant becomes a noisy neighbor

Apply per-tenant quotas, isolation, or dedicated capacity before it evicts or saturates shared resources.

## Trade-offs

- L1 gives excellent read offload but weakens coherence.
- Replication adds read capacity but costs memory and update bandwidth.
- Key splitting spreads load but complicates lookup, invalidation, and aggregation.
- Approximate counters scale better but permit bounded error.
- Rate limiting protects shared capacity by intentionally rejecting demand.

## Production Gotchas

- Never put raw cache keys or tenant PII into high-cardinality public metrics.
- Detect heavy hitters continuously; today's hot key may be different tomorrow.
- Keep headroom for replica promotion and viral bursts.
- Avoid synchronized expiry of replicated hot copies.
- Validate that key splitting preserves the operation's invariant.
- Test hot-key node loss, popular-key expiry, and tenant bursts.

## Alternatives

- Remove the cache lookup with CDN/browser caching.
- Materialize or broadcast a small configuration into application memory.
- Redesign an atomic global counter into regional budgets where bounded error is acceptable.
- Serve a static degraded response during extreme demand.

## Interview Questions

### Why does adding cache nodes not fix one hot key?

If routing maps the key to one owner, extra nodes receive no share of its traffic. Replicate, cache locally, split only when semantics allow, or change the access pattern.

### How do you detect hot keys without unbounded metrics?

Use sampled access logs, top-K/heavy-hitter tracking, bounded key classes, and per-shard resource metrics.

### How do hot reads differ from hot writes?

Reads can often be replicated. Writes may require one authoritative serialization point or a deliberate approximate/partitioned model.

## 2-Minute Interview Answer

> “I treat hot keys as skew, not total-capacity shortage. I measure top-key traffic and per-shard QPS, latency, CPU, and bandwidth. For an immutable or bounded-stale read I can use CDN caching, local L1 copies, or read replicas; I coalesce refreshes so expiry does not move the hotspot to the origin. Key splitting helps only when I can replicate a read value or partition an aggregatable operation without breaking semantics. For a hot write I may need regional budgets, sharded partial counters, or one protected authoritative owner. I keep overload controls because adding generic nodes does not help a key pinned to one shard.”

## Senior-Level Follow-ups

- How do you invalidate 100 replicated copies safely?
- Can an exact global counter be split without coordination?
- What happens when the hottest shard fails?
- How do you prevent one tenant from consuming the cache?
- Which technique responds fastest to sudden viral traffic?

## References

- [Internal: Redis — Hot Keys](../databases/nosql/redis.md)
- [Internal: Cache Stampede](cache-stampede.md)
- [Internal: Local vs Distributed Cache](local-vs-distributed-cache.md)
- [AWS Builders' Library — Caching Challenges and Strategies](https://aws.amazon.com/builders-library/caching-challenges-and-strategies/)

