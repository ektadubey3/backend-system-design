# Distributed Caching

A distributed cache spreads cached state and traffic across multiple nodes. It adds aggregate capacity and shared reuse, but also introduces routing, partitioning, replication, rebalancing, network, and recovery behavior.

## Interview TL;DR

1. Define whether cached state is disposable, reconstructable but operationally critical, or durable/authoritative.
2. Sharding increases aggregate capacity; replication improves availability or read capacity. They solve different problems.
3. Consistent or rendezvous hashing reduces key movement when membership changes; it does not guarantee balanced traffic.
4. Rebalancing raises miss rate and origin load as key ownership changes.
5. Replication does not automatically make cached state strongly consistent or lossless.
6. Client-side routing simplifies the data path but complicates membership and client behavior; proxy routing centralizes those concerns but adds a hop.
7. Hot shards, connection pools, serialization, and network bandwidth can limit the system before memory.
8. Origin protection is part of cache-cluster design.

## Mental Model

```text
Client
  |
  v
Cache cluster
  +-- shard A
  +-- shard B
  +-- shard C
  |
  v
Authoritative store on miss
```

Start by classifying the cache role:

### Disposable optimization

All entries can be deleted and rebuilt without losing business data. Persistence may be unnecessary, but cold-cache recovery must still protect the origin.

### Reconstructable but operationally critical

Data can be rebuilt, yet the source cannot serve normal traffic without the cache. The cache is part of the availability and capacity architecture; recovery time, spare capacity, and failover matter.

### Durable/critical state

Losing the cache can lose acknowledged state or violate an invariant. This is a state-store design requiring explicit durability, replication, backup, restore, RPO, RTO, and concurrency semantics. Do not call it disposable.

## Partitioning and Routing

### Simple modulo hashing

```text
shard = hash(key) mod N
```

It distributes stable membership simply, but changing `N` remaps many keys and causes a large cold-cache event.

### Consistent hashing

Place nodes and keys in a hash space. Membership changes move a smaller subset of keys than simple modulo hashing. Virtual nodes can improve distribution and capacity weighting.

### Rendezvous hashing

Score each candidate node for a key and choose the highest score. It offers conceptually simple minimal-disruption assignment without a ring. Real implementations must still handle node weights, membership agreement, and failure detection.

Neither approach solves hot-key traffic. A hash function distributes keys, not necessarily request volume or object size.

### Client-side routing

Cache-aware clients select the shard directly.

Benefits:

- no proxy hop;
- data path can scale with clients.

Costs:

- every client needs consistent membership and routing logic;
- rolling client upgrades can behave differently;
- connection counts multiply across application instances;
- redirects/retries and topology refresh must be correct.

### Proxy/server-side routing

Clients contact a stable endpoint that routes internally.

Benefits: centralized topology and simpler clients. Costs: extra hop, proxy capacity, and another failure boundary. Scale and replicate the routing tier if it is in the data path.

## Replication and Failover

Replicate shards to tolerate node loss or add read capacity:

```text
shard A primary -> replica A1 -> replica A2
```

Ask:

- Is replication synchronous or asynchronous?
- Can reads go to replicas, and how stale may they be?
- Can recently acknowledged writes disappear on failover?
- Who elects/promotes a new owner?
- How do clients learn topology changes?
- Is split brain or stale primary access possible?

Replication improves specific failure behavior. It does not automatically make the cache authoritative, linearizable, or zero-RPO.

## Rebalancing

Adding or removing nodes changes ownership:

```text
membership change
      |
      v
keys move or become cold
      |
      v
temporary miss-rate increase
      |
      v
origin load increase
```

Plan gradual movement, rate-limited warming, spare cache capacity, and origin concurrency budgets. Test topology changes with realistic traffic before production.

## Connection and Serialization Architecture

Distributed cache latency includes:

```text
pool wait
+ network
+ server queue/execution
+ serialization/deserialization
+ optional compression
```

Avoid one connection per request. Size pools against application instance count and cache-node limits. Batching or pipelining can reduce round trips, but it is not atomicity.

Version serialized values for rolling deployments. A schema mismatch that causes every client to discard cached data can create a mass miss event.

## Multi-Region Design

Options include:

- independent regional caches backed by one authoritative region;
- regional caches fed by events/CDC;
- globally replicated cache service;
- CDN/edge cache for public representations.

Define:

- regional write ownership;
- acceptable cross-region staleness;
- behavior during inter-region partition;
- failover data loss and warm-up;
- data residency and PII controls;
- whether cache invalidation can cross regions within the required SLO.

A global cache can lower read latency, but stronger freshness may require cross-region coordination that gives back latency or availability.

## When To Use It

Use distributed caching when shared reuse, aggregate working-set memory, centralized invalidation, or fleet-independent origin offload justifies a remote service.

Do not introduce a cluster when an in-process cache, HTTP cache, optimized query, read replica, or no cache meets the requirement more simply.

## Scaling Characteristics

Sharding can scale aggregate:

- memory;
- CPU;
- network bandwidth;
- independent operations.

Limits remain:

- hottest key/shard;
- client connections;
- routing metadata;
- cross-shard operations;
- replication bandwidth;
- rebalancing rate;
- source capacity during misses;
- cluster control-plane behavior.

Capacity-plan per shard and for failover, not only across the full cluster. Losing one node should not overload its replacement.

## Failure Modes

### Node loss

Keys become unavailable, promote a replica, or remap. Each choice affects availability, consistency, and miss rate.

### Network partition

Clients may disagree about ownership, replicas may lag, and writes may be rejected or diverge depending on the system. Define operation behavior rather than labeling the whole cache “AP” or “CP.”

### Hot shard

Traffic or large values concentrate on one node. Adding unrelated nodes may not help.

### Connection storm

Failover or deploy causes many clients to reconnect simultaneously. Backoff with jitter and bound pool creation.

### Rebalance/cold-cache overload

Origin receives traffic for moved keys. Cap and prioritize source work.

### Serialization poison pill

One value crashes or repeatedly fails clients. Reject safely, isolate, and avoid a fleet-wide discard storm.

### Complete cluster loss

Use progressive recovery, hot-set warming, bounded fallback, stale local data where safe, feature degradation, and load shedding.

## Trade-offs

- More shards add capacity but increase topology and rebalancing complexity.
- More replicas improve read/failure capacity but consume memory and may serve stale data.
- Client routing removes a hop but distributes correctness logic.
- Proxy routing centralizes behavior but adds a shared component.
- Persistence can reduce warm-up or data loss but increases cost and does not replace clear authority.
- Multi-region caches reduce read distance but complicate coherence and security.

## Production Gotchas

- Monitor p50/p95/p99 by shard, errors, timeouts, CPU, memory, fragmentation, network, connections, replication lag, failovers, hot keys, and rebalance progress.
- Separate cache hit, miss, bypass, and source fallback telemetry at the application.
- Encrypt traffic and authenticate clients; isolate tenants and avoid secrets in keys.
- Version payloads and keys across rolling deploys.
- Maintain headroom for one-node or one-shard failure.
- Test add/remove node, failover, network isolation, cold restart, and origin protection.

## Alternatives

- Local L1 for a small hot set.
- CDN or HTTP caching for public content.
- Read replicas for low-reuse database reads.
- Durable projections for data that is expensive to rebuild.
- Direct source access when distributed-cache complexity is not justified.

## Interview Questions

### Why consistent hashing?

It reduces key remapping when membership changes compared with simple modulo hashing. It does not eliminate rebalancing cost or traffic skew.

### Does replication prevent data loss?

Only according to the replication and acknowledgment protocol. Asynchronous failover can expose older state or lose recent acknowledged cache writes.

### What happens when a node is added?

Some key ownership moves, causing transfer or misses. Origin protection and controlled warm-up are part of the scaling plan.

## 2-Minute Interview Answer

> “I use a distributed cache when applications need shared reuse or more working-set capacity than local memory provides. Keys are partitioned with a minimal-disruption scheme such as consistent or rendezvous hashing, but I still measure per-shard traffic because hashing key count does not solve hot keys. Replicas provide defined read or failover capacity; I do not claim strong consistency or no data loss without the actual acknowledgment protocol. Membership changes are controlled cold-cache events, so rebalancing and recovery are rate-limited and origin calls have a strict concurrency budget. I size client pools, version serialized values, keep per-shard headroom, and define whether the cache is disposable, reconstructable but critical, or truly authoritative.”

## Senior-Level Follow-ups

- How do clients agree on membership during a partition?
- What happens to multi-key operations across shards?
- How much origin traffic does adding 20% cache capacity create?
- Can a promoted replica return an older value?
- How does region failover warm the new cache safely?
- When should the routing layer be client-side versus proxied?

## References

- [Redis Cluster Specification](https://redis.io/docs/latest/operate/oss_and_stack/reference/cluster-spec/)
- [Redis documentation — Replication](https://redis.io/docs/latest/operate/oss_and_stack/management/replication/)
- [Memcached documentation](https://docs.memcached.org/)
- [Karger et al. — Consistent Hashing and Random Trees](https://people.csail.mit.edu/karger/Papers/web.pdf)
- [Nishtala et al. — Scaling Memcache at Facebook](https://www.usenix.org/conference/nsdi13/technical-sessions/presentation/nishtala)
- [Internal: Redis](../databases/nosql/redis.md)
- [Internal: Hot Keys](hot-keys.md)

