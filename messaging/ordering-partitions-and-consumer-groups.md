# Ordering, Partitions, and Consumer Groups

## TL;DR

Order is purchased with serialization. Define the smallest business key that requires order, route that key consistently to one partition, and let unrelated keys run in parallel. A consumer group normally assigns each partition to one active member, so partition count sets a practical ceiling on parallel consumption.

## Scope order before promising it

Global order forces all traffic through one sequencer or requires expensive coordination. Most domains need narrower invariants:

- Changes for one account must be applied in version order.
- Messages in one chat conversation should appear in a stable order.
- Inventory reservations for one SKU must serialize at the authority.
- Events for unrelated accounts, chats, or SKUs may proceed concurrently.

Choose a partition key from that invariant, not from whichever identifier is easiest to hash.

## Partitioned log model

```text
key -> partitioner -> P0: 10,11,12
                   -> P1: 21,22
                   -> P2: 30,31,32

consumer group G: worker A owns P0; worker B owns P1 and P2
```

Records are ordered by position inside a partition. There is no inherent comparison between `P0:12` and `P1:22`. Adding consumers beyond the number of active partitions does not increase group parallelism. Adding partitions later may change key mapping unless the platform preserves existing assignments; plan for that effect.

## Consumer group behavior

A group represents one logical subscription. Each partition has one active owner within that group, while a different group receives the same retained records independently. Membership changes cause reassignment. During a rebalance:

- old owners must stop before new owners process the same partition concurrently;
- in-flight work may be repeated;
- a slow handler can look dead and trigger churn;
- offsets committed ahead of business state can lose work;
- offsets committed after business state can repeat work.

Keep processing time bounded, separate polling from long work when the client model permits it, and make ownership changes observable.

## Preserving causal order

Transport order is insufficient when producers race. Include an entity version or sequence number. A consumer can then:

- apply `version = current + 1`;
- ignore an already-applied version;
- buffer a short gap;
- fetch authoritative state or send the record to repair when the gap persists.

Multiple producer instances need a single authoritative version assignment, database commit order, or another sequencer. Wall-clock timestamps are not a safe total order across machines.

## Hot partitions

A celebrity, tenant, or popular SKU can dominate one key and cap throughput at one partition. Options include:

- accept serialization because the invariant requires it;
- split the aggregate into independently ordered subkeys;
- add a two-stage design: shard ingestion, then merge at a scoped authority;
- aggregate or coalesce updates;
- isolate a heavy tenant into dedicated partitions.

Randomly salting the key improves spread but destroys per-key order unless there is a merge protocol.

## Failure modes and tradeoffs

| Decision | Benefit | Cost |
|---|---|---|
| One global partition | simple total order | low throughput and large blast radius |
| Partition by entity | scalable scoped order | hot keys; no cross-entity order |
| More partitions | more parallelism | metadata, open files, rebalance, and cost |
| Commit offset before work | fewer duplicates | possible loss |
| Commit after durable work | no silent loss | duplicates require idempotency |

## Interview prompts

- What exact invariant requires ordering?
- How is a partition key assigned, and can it become hot?
- What happens to in-flight work during reassignment?
- How does the consumer detect missing or stale entity versions?

## Two-minute answer

Define order per business aggregate, such as account or conversation, and hash that key to a partition. State that order exists only inside a partition and that a consumer group has one active owner per partition. Commit progress after durable, idempotent processing, expecting repeats on failure or rebalance. Include entity versions to detect gaps and stale updates, monitor lag per partition, and explain how the design handles a hot key without casually discarding the invariant.

