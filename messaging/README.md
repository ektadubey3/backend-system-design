# Messaging Systems

Messaging moves work and facts across time, processes, and failure boundaries. The interview skill is not naming a broker; it is defining what a message means, where ordering matters, who owns retries, and how duplicates are made harmless.

## Learning path

1. [Messaging fundamentals](messaging-fundamentals.md) — queues, pub/sub, streams, and the asynchronous boundary.
2. [Delivery semantics and idempotency](delivery-semantics-and-idempotency.md) — at-most-once, at-least-once, scoped exactly-once claims, and safe side effects.
3. [Ordering, partitions, and consumer groups](ordering-partitions-and-consumer-groups.md) — the unit of parallelism and the cost of order.
4. [Retries, dead letters, and backpressure](retries-dead-letters-and-backpressure.md) — failure policy without retry storms.
5. [Transactional outbox and CDC](transactional-outbox-and-cdc.md) — closing the database/broker dual-write gap.
6. [Schema evolution and replay](schema-evolution-and-replay.md) — compatibility, retention, and rebuilding consumers.
7. [Broker selection](broker-selection.md) — Kafka, RabbitMQ, and SQS as different operating models.
8. [Messaging design framework](messaging-design-framework.md) — an interview-ready decision sequence.

## Boundary with existing material

The canonical deep treatment of database transactions, idempotency records, transactional outbox, CDC, saga, and two-phase commit remains in [SQL transactions](../databases/sql/transactions.md). This section explains how those patterns shape a messaging architecture and links back rather than maintaining a second competing explanation.

## What strong answers make explicit

- Whether a record is a command, event, notification, or durable log entry.
- The required ordering key and whether unrelated keys may progress independently.
- Where acknowledgement occurs relative to the business transaction.
- Retry ownership, delay, attempt budget, poison-message handling, and operator workflow.
- Idempotency key, deduplication scope, retention window, and side-effect policy.
- Retention, replay, schema compatibility, and consumer lag objectives.
- Broker failure assumptions and what the application must still guarantee.

## Fast review

Use the [design framework](messaging-design-framework.md) for a two-minute answer, then revisit delivery semantics and partitioning: those two choices drive most downstream tradeoffs.

