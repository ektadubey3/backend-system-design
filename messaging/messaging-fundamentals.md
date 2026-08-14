# Messaging Fundamentals

## TL;DR

A queue distributes work, pub/sub fans a fact out to subscribers, and a durable stream retains an ordered log that consumers can revisit. These shapes overlap in products, so select the semantic model before the broker. Messaging reduces temporal coupling, but introduces lag, duplicates, reordering, versioning, and operational state.

## Mental model: asynchronous ownership transfer

A synchronous caller retains responsibility until a response arrives. A producer that durably publishes transfers responsibility to the messaging system and, eventually, a consumer. That transfer is real only after the broker has accepted the record under its durability policy.

```text
producer -> durable acceptance -> stored record -> delivery -> processing -> acknowledgement
             publication SLO                       consumer responsibility
```

Distinguish four common meanings:

| Meaning | Example | Natural shape | Primary concern |
|---|---|---|---|
| Command | `ChargePayment` | queue | one logical handler owns the work |
| Domain event | `PaymentCaptured` | pub/sub or stream | multiple independent reactions |
| Notification | `WakeWorker` | lossy or durable queue | freshness may matter more than history |
| Change log | row mutation or audit fact | durable stream | retention, order, and replay |

A topic name does not establish meaning. Put message intent, owner, and compatibility policy in the contract.

## Queue, pub/sub, and stream

In a work queue, competing consumers divide records. Acknowledged work leaves the active set, although it may remain in internal storage. This is a good fit for independent jobs whose order is weak or scoped.

In pub/sub, each subscription receives its own logical copy. A slow analytics consumer should not block a notification consumer. Fan-out therefore multiplies storage, delivery, and schema-compatibility obligations.

A durable stream appends records and tracks consumer positions. Retention is independent of acknowledgement, which enables replay and new consumers. Partitioned streams trade global order for parallelism: order is normally meaningful only within one partition.

## Why messaging helps

- **Temporal decoupling:** producer and consumer need not be healthy simultaneously.
- **Load leveling:** a bounded backlog absorbs a short burst while consumers drain at a sustainable rate.
- **Independent evolution:** subscribers can deploy and scale separately when contracts are compatible.
- **Failure isolation:** non-critical work can fail without extending the request path.
- **Audit and reconstruction:** retained facts can rebuild projections or support investigation.

It does not make work free. If arrival rate exceeds service rate for long enough, lag grows without bound. Queueing moves the overload signal; it does not remove the capacity requirement.

## Core design choices

### Durability and acknowledgement

Define when a producer considers publication successful: leader memory, leader disk, or replicated durable acknowledgement. On the consumer, acknowledge after the durable business effect, not merely after deserialization. If processing and acknowledgement cannot be atomic, duplicates are normal.

### Retention and expiry

Short-lived task queues optimize for completion. Streams often retain by time or size. Retention must exceed the longest planned outage plus replay time, with margin. Expiry is a product rule too: sending a two-day-old typing indicator is worse than dropping it.

### Payload versus reference

An event containing the required immutable facts is replayable and avoids a read-after-notify race. A thin message containing only an entity ID is smaller but couples the consumer to the producer's current database state. Large binary payloads usually belong in object storage with a stable reference and integrity metadata.

## Failure modes

- Producer commits local state but fails to publish: use an [outbox or CDC](transactional-outbox-and-cdc.md).
- Consumer performs a side effect then crashes before acknowledgement: make handling [idempotent](delivery-semantics-and-idempotency.md).
- One poison record loops forever: bound attempts and provide a dead-letter workflow.
- Consumers fall behind: expose oldest-message age and lag, then shed, scale, or reduce work.
- A hot key caps partition throughput: revisit the ordering scope or split the key deliberately.
- Schema change breaks an offline consumer: enforce compatibility in the release process.

## Tradeoffs

| Choice | Gain | Cost |
|---|---|---|
| Synchronous call | immediate result, simple debugging | availability and latency coupling |
| Work queue | load leveling, competing consumers | weaker replay model, duplicate work |
| Pub/sub | independent fan-out | per-subscriber operations and compatibility |
| Durable stream | replay, ordered history | partition planning and retention cost |

## Interview prompts

- Why can a queue reduce peak pressure but not solve sustained overload?
- When is a message an event rather than a command?
- What business work happens before acknowledgement?
- Which messages become harmful when delivered late?

## Two-minute answer

Start with message meaning and the producer-to-consumer contract. Choose a queue for distributed work, pub/sub for independent reactions, or a retained stream when replay and ordered history matter. Define durable publication, the ordering key, acknowledgement after business commit, idempotency, retry/dead-letter policy, retention, schema compatibility, and lag objectives. Then show the failure path: duplicates, poison records, slow consumers, and broker unavailability.

