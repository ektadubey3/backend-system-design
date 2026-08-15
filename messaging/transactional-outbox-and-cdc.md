# Transactional Outbox and Change Data Capture

## TL;DR

A transactional outbox writes the business change and a publishable record in one local database transaction. A relay or change-data-capture (CDC) connector then publishes that record at least once. This closes the “database committed but event was lost” gap, but it does not remove duplicates, ordering design, schema ownership, cleanup, or monitoring.

## The dual-write problem

Neither order of two independent writes is safe:

```text
database commit -> process dies -> broker publish missing
broker publish -> database rolls back -> consumers observe a fact that never committed
```

Distributed transactions are rarely available across every database, broker, and external service. The outbox reduces the atomic boundary to one database transaction.

## Outbox flow

```text
BEGIN
  mutate business rows
  insert outbox(event_id, aggregate_id, type, version, payload, occurred_at)
COMMIT

relay/CDC -> broker -> idempotent consumers
```

The committed outbox row means “this fact is eligible for publication,” not “every consumer has processed it.” The relay may publish and crash before marking progress, so repeat publication is expected.

Use an immutable event ID for deduplication and an aggregate ID as the partition key when per-aggregate order matters. If several outbox writers can update one aggregate, add an aggregate version assigned by the transaction authority.

## Polling relay versus log-based CDC

| Choice | Strength | Cost |
|---|---|---|
| Polling outbox table | easy to understand and debug; application-controlled | polling load, claim/lease logic, cleanup |
| Database-log CDC | low-latency capture without table scans; preserves commit-log position | connector operations, database-specific log retention and permissions |
| Direct dual write | initially simple | unavoidable split-brain outcome on partial failure |

A polling relay should claim batches safely, avoid one stuck row blocking all later rows, and expose oldest-unpublished age. CDC must monitor connector lag and ensure the database log is retained longer than the maximum outage; otherwise the capture position can fall off the log.

## Event contract ownership

Raw table-change events expose storage details and make consumers depend on internal columns. A curated outbox event is usually a more stable domain contract. CDC can transport that outbox efficiently without making the binlog itself the public API.

Include only facts consumers may retain. Redact secrets and minimize personal data. Define compatibility, tombstone/deletion semantics, and the owner who approves changes.

## Cleanup and recovery

Outbox growth is production state. Delete or archive only after the relay's durable checkpoint makes replay safe. Track:

- oldest unpublished row age;
- publish and failure rates;
- duplicate rate;
- table size or log-retention headroom;
- broker acknowledgement latency;
- gaps by aggregate version.

Recovery must be rehearsed: restore a relay checkpoint, replay a bounded range, verify idempotent consumers, and reconcile source state against derived state.

## Boundary with transactions and saga

For implementation details on idempotency tables, two-phase commit, saga compensation, and orchestration versus choreography, use the repository's canonical [SQL transactions chapter](../databases/sql/transactions.md). The outbox transports committed local facts; a saga coordinates several local transactions and their compensating actions. They solve different layers of the workflow.

## Interview prompts

- What exact failure does an outbox close, and which failures remain?
- How are records claimed, published, checkpointed, and cleaned up?
- Does CDC expose raw table changes or curated domain events?
- How do consumers detect a missing aggregate version?

## Two-minute answer

Avoid the database/broker dual write by committing the business mutation and an immutable outbox row together. A polling relay or CDC connector publishes it at least once, keyed by aggregate when scoped order matters. Consumers deduplicate by event or business operation ID. Monitor oldest unpublished age, connector/log retention, table growth, and gaps; retain curated event contracts instead of exposing the entire storage schema. Link multi-service business completion to a saga, not to the outbox itself.

## References

- [Debezium — Outbox event router](https://debezium.io/documentation/reference/stable/transformations/outbox-event-router.html)
- [PostgreSQL — Logical decoding concepts](https://www.postgresql.org/docs/current/logicaldecoding-explanation.html)

