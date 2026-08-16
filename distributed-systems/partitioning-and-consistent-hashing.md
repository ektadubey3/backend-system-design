# Partitioning and Consistent Hashing

## TL;DR

Partitioning assigns each key or range to an authority so data and work scale horizontally. The hard parts are choosing a key, preserving invariants, detecting hot shards, moving ownership safely, and routing during transition. Consistent hashing reduces movement for hash-based membership changes; it does not solve skew, replication, consensus, or resharding correctness.

## Partitioning strategies

| Strategy | Strength | Weakness |
|---|---|---|
| Range | efficient scans and locality | sequential/hot ranges and uneven growth |
| Hash | even spread for ordinary keys | poor range scans; hot individual keys remain hot |
| Directory | flexible placement and tenant isolation | routing metadata becomes critical state |
| Geographic/tenant | locality, sovereignty, blast-radius control | uneven tenants and cross-partition workflows |

The partition key should align with the most important access path and invariant. If an order and all its line items transact together, `order_id` is a useful boundary. If a transfer spans two account partitions, the design needs a coordinator, an escrow model, or a changed invariant.

## Capacity and skew

Average load hides the partition that fails first. Track per-partition QPS, bytes, storage, queue age, throttling, and key distribution. Heavy-hitter detection should identify hot keys, not only hot nodes.

Mitigations include splitting a large tenant, isolating it on dedicated capacity, write sharding with a read aggregate, cached fan-out, request coalescing, and changing the product operation. Salting spreads one key only if the application can merge results without violating order or uniqueness.

## Consistent hashing

A basic modulo hash remaps most keys when node count changes. Consistent hashing places nodes and keys on a ring and moves primarily the ranges adjacent to changed nodes. Virtual nodes or many small tokens smooth capacity and allow weighted ownership.

Real systems still need:

- an agreed membership/configuration version;
- replication placement across failure domains;
- bounded-load or token-balancing logic for heterogeneous nodes;
- safe handoff and dual-routing during movement;
- repair of missed updates;
- protection from a single hot key.

For cache-specific application, see [Distributed caching](../caching/distributed-caching.md).

## Online resharding

A safe state machine is more useful than “move the shard”:

1. Create target ownership under a new configuration version.
2. Copy a snapshot while source remains authoritative.
3. Stream changes after the snapshot position.
4. Verify count/checksum and lag.
5. Enter a bounded cutover using forwarding, dual write, or brief write pause.
6. Atomically publish the new routing version.
7. Keep source forwarding/read fallback for a grace period.
8. Delete old data only after clients and repair windows have expired.

Requests carry or discover a routing epoch. A stale router must be redirected rather than silently writing the old owner.

## Failure modes

- Low-cardinality partition key creates a few enormous shards.
- Time-based range makes the newest shard hot.
- Rebalancing saturates the same disk/network needed for live traffic.
- Both source and target accept authoritative writes after a partial cutover.
- Metadata service is unavailable and new clients cannot route.
- Cross-shard query fan-out turns one user request into hundreds of calls.

## Interview prompts

- What is the partition key and which invariants cross it?
- What is the largest tenant/key, not just the average?
- How does a stale client learn the new owner?
- Can resharding be paused and resumed safely?

## Two-minute answer

Choose a partition key that co-locates the dominant access path and hard invariant. Use range partitioning for scans, hash partitioning for distribution, or a directory for explicit placement. Track skew per shard and isolate or split heavy keys. Consistent hashing reduces remapping but still requires agreed membership, failure-domain-aware replicas, versioned routing, and safe handoff. Explain snapshot-plus-change-stream resharding, cutover, forwarding, verification, and rollback.

## References

- [Karger et al. — Consistent Hashing and Random Trees](https://www.cs.princeton.edu/courses/archive/fall09/cos518/papers/chash.pdf)
- [Amazon Dynamo paper](https://www.amazon.science/publications/dynamo-amazons-highly-available-key-value-store)

