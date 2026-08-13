# Eviction Policies

Eviction decides which entry loses memory when a cache cannot retain everything. The right policy depends on reuse, item cost, size distribution, and whether the stored state is actually disposable.

## Interview TL;DR

1. Expiration and eviction are different events.
2. LRU favors recent reuse; LFU favors sustained popularity; neither is universally best.
3. FIFO and random eviction can be reasonable when bookkeeping cost or workload simplicity matters.
4. TTL-aware policies express lifetime, not necessarily future reuse value.
5. No-eviction behavior may be required when state must not disappear, but that makes writes fail under memory pressure.
6. Large scans and one-time values can pollute a cache and evict the true working set.
7. If eviction can violate correctness, the component is not being used as a disposable cache.

## Mental Model

```text
Expiration:
entry becomes unusable because its time/policy ends

Eviction:
cache removes an entry because resource pressure or replacement policy chooses it
```

An entry can be evicted long before its TTL. Therefore TTL is not a guarantee of availability or residency.

The real decision is:

> Which retained byte is most likely to avoid valuable future work?

That value depends on access frequency, recency, object size, regeneration cost, and business importance.

## How Policies Behave

### Least Recently Used (LRU)

Evict the entry whose most recent access is oldest.

Good fit when recent access predicts near-future reuse, such as temporal locality in product or session reads.

Weaknesses:

- a large scan can make one-time items look recent;
- metadata and update cost may be approximated by real systems;
- it ignores regeneration cost and value size unless the implementation adds admission logic.

### Least Frequently Used (LFU)

Evict entries with the lowest estimated access frequency.

Good fit when popularity is stable enough that long-term frequency predicts reuse. Decay is important so yesterday's popular key does not remain forever.

Weaknesses:

- new items may lose to historically popular entries;
- abrupt popularity changes take time to learn;
- counters and decay add policy complexity.

### First In, First Out (FIFO)

Evict the oldest inserted item without tracking reads.

It is simple and predictable, but insertion age is often a weak signal for reuse. It can fit bounded buffers or workloads where tracking access is not worth the overhead.

### Random eviction

Select a victim without recency or frequency bookkeeping. Random can be cheap and surprisingly adequate when accesses are uniform, but it can discard hot or expensive entries.

### TTL-based selection

Prefer entries closest to expiration, or rely on expiration before memory pressure. This works when lifetime meaningfully predicts value, but the shortest remaining TTL is not always the least useful entry.

### No eviction

Reject writes or allocations once the memory limit is reached.

This avoids silently deleting entries, but callers must handle write failure and existing reads may continue while new cache population stops. For correctness-sensitive state, no eviction alone is not a durability plan; capacity, persistence, and recovery are still required.

## Workload Decision Framework

Ask:

```text
Does recency predict reuse?       -> consider LRU
Is popularity stable?            -> consider LFU
Is bookkeeping cost dominant?    -> consider FIFO/random
Does lifetime encode usefulness? -> consider TTL-aware behavior
May any entry disappear safely?  -> if no, redesign the storage role
```

Then include object economics:

- A 1 MB object and a 1 KB object should not automatically have equal admission value.
- A rare query that costs 20 seconds to regenerate may be worth more than a frequent cheap lookup.
- One tenant's scan should not evict another tenant's critical hot set.

Many cache products offer only a subset or approximation of these policies. Validate actual semantics and do not claim textbook LRU when the implementation samples candidates.

## Architecture

Separate pools when workloads have incompatible eviction economics:

```text
small hot objects  -> cache pool A
large query results -> cache pool B
ephemeral sessions  -> separate state policy
```

Admission control can reject values that are too large, scan-like, low-reuse, or more expensive to store than to fetch. Eviction decides what leaves; admission decides what should enter.

## Scaling Characteristics

Estimate memory with:

```text
keys + values + metadata + allocator overhead
+ replication + fragmentation + headroom
```

Monitor the full distribution of item sizes and ages. A rising eviction rate is meaningful primarily when it lowers useful hit rate and increases origin load or user latency.

Adding memory helps only if the newly retained entries are reused. If the working set is unbounded or access is one-time, a larger cache delays the same problem.

## Failure Modes

### Eviction storm

Memory pressure removes useful entries, misses increase, repopulation writes more entries, and churn continues while the origin receives higher load.

### Cache pollution

A batch scan admits millions of cold entries and displaces the hot set.

### Large-object domination

A few values consume most memory or network bandwidth while aggregate item-count metrics look healthy.

### Cross-workload interference

Optional recommendation results evict sessions or configuration from a shared pool.

### No-eviction write failures

The cache appears healthy for reads but silently stops accepting new representations unless the application observes and handles population errors.

## Trade-offs

- LRU adapts to recent changes but is vulnerable to scans.
- LFU protects established hot entries but may adapt slowly.
- Random and FIFO lower bookkeeping complexity but ignore reuse signals.
- Large isolated pools improve workload protection but increase operational overhead and stranded capacity.
- No eviction preserves existing entries but converts memory pressure into write-path errors.

## Production Gotchas

- Treat eviction rate together with miss cost, hit rate by key class, and origin QPS.
- Alert on memory headroom before eviction begins.
- Bound object size and consider not admitting scan results.
- Track per-tenant memory and top evicted key classes.
- Do not mix disposable cache entries with state whose loss changes correctness.
- Load-test access skew and realistic object-size distributions, not uniform random keys.

## Alternatives

- Increase memory only after proving the useful working set benefits.
- Use a cheaper backing projection or read replica for low-reuse reads.
- Apply explicit admission filters or separate cache pools.
- Precompute expensive results with a durable rebuild model.

## Interview Questions

### Is LRU the best eviction policy?

No. It fits recency-heavy workloads, but scans, stable popularity, large objects, or differing regeneration costs may favor another policy or admission control.

### Can an entry be evicted before its TTL?

Yes. TTL governs expiration; memory-pressure policy governs eviction.

### What if cached state must never be evicted?

Then it is not safely disposable. Use an explicit state-store design with capacity, durability, and recovery semantics rather than relying on cache behavior.

## 2-Minute Interview Answer

> “I choose eviction from the access distribution, not a default. LRU is plausible when recent reads predict reuse; LFU can fit stable popularity; FIFO or random may be acceptable when bookkeeping cost matters. I distinguish eviction from expiration because memory pressure can remove an entry before its TTL. I size for values, keys, overhead, replication, fragmentation, and headroom, and I protect the hot set from scans through admission control or separate pools. The success metric is not a low eviction count by itself—it is whether the retained working set reduces origin load and latency without letting one tenant or value class dominate memory.”

## Senior-Level Follow-ups

- How would you make an LRU-like cache resistant to full-table scans?
- Should a 10 MB result compete equally with a 1 KB object?
- What happens when `noeviction` rejects new entries?
- When does adding memory stop improving origin offload?
- Which workloads should not share an eviction pool?

## References

- [Redis documentation — Key Eviction](https://redis.io/docs/latest/develop/reference/eviction/)
- [Memcached documentation](https://docs.memcached.org/)
- [Internal: TTL and Expiration](ttl-and-expiration.md)
- [Internal: Caching Fundamentals](caching-fundamentals.md)

