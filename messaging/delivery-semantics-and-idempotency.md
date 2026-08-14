# Delivery Semantics and Idempotency

## TL;DR

Delivery labels describe a boundary, not an end-to-end business guarantee. At-most-once may lose work; at-least-once may duplicate it. “Exactly once” is credible only when its scope, transaction boundary, state store, and external side effects are stated. Most production consumers should assume duplicates and make the business transition idempotent.

## The acknowledgement gap

Consider a consumer that writes the database and then acknowledges:

```text
read M -> commit business effect -> crash -> broker redelivers M
```

Acknowledging first avoids the duplicate but creates a loss window:

```text
read M -> acknowledge -> crash -> business effect never happens
```

No acknowledgement order makes two independent systems atomic. The solution is a shared transaction, a protocol that scopes atomicity, or application-level idempotency and reconciliation.

## Semantics

| Model | Mechanism | Failure result | Appropriate when |
|---|---|---|---|
| At-most-once | acknowledge before work, or do not retry | possible loss, no broker-driven duplicate | stale telemetry or hints are disposable |
| At-least-once | retry until acknowledged | duplicates possible | durable business work with idempotent handling |
| Effectively-once | at-least-once plus deduplicated transition | one visible effect inside a declared scope | most transactional consumers |
| Transactional exactly-once | atomically consume, mutate supported state, and publish | one committed result within that platform boundary | broker-native pipelines whose sinks participate |

A broker transaction cannot stop a payment provider from accepting a request and the consumer crashing before recording the response. External effects still need an idempotency key, queryable operation state, or reconciliation.

## Idempotency patterns

### Natural idempotency

`set order.status = 'CANCELLED'` is naturally repeatable if state transitions are validated. `increment balance by 10` is not. Prefer commands that name the intended state or a unique business operation.

### Inbox or processed-message record

Within one database transaction:

1. Insert `(consumer, message_id)` under a unique constraint.
2. If it already exists, treat the message as completed.
3. Apply the business mutation.
4. Commit both.

This closes the consumer/database gap. The deduplication retention must cover the broker's maximum redelivery and replay horizon. If records are pruned sooner, an old replay becomes new again.

### Business uniqueness

Use a stable operation key such as `capture:{payment_id}:{attempt_generation}` and enforce it in the domain table. This is stronger than a generic inbox when the same intent can arrive through several channels.

### Idempotent external call

Pass a stable idempotency key to a provider and persist the provider operation ID. On an ambiguous timeout, query status before issuing a new logical operation. Never generate a new key for a retry of the same intent.

## Producer duplicates

A producer can time out after a broker accepted a record. Retrying may publish twice. Broker producer IDs and sequence numbers can suppress some transport duplicates, but business deduplication is still needed across producer restarts, migrations, manual replay, or events created by two code paths.

Use a globally unique event ID and a separate business key. The event ID identifies this envelope; the business key identifies the operation whose effect must occur once.

## Common traps

- Treating an HTTP timeout as proof that the remote operation failed.
- Deduplicating in an in-memory cache that can evict or restart.
- Acknowledging after sending email but before recording that it was sent.
- Reusing one idempotency key for materially different requests.
- Keeping dedupe records forever without sizing or privacy policy.
- Calling a queue “exactly once” without defining publication, processing, and sink boundaries.

## Interview prompts

- A consumer charged a card and crashed before acknowledgement. What happens next?
- Where is the unique constraint and how long is it retained?
- Can two different messages represent the same business intent?
- Which side effects cannot join the local transaction?

## Two-minute answer

Assume at-least-once delivery. Acknowledge only after the local durable effect. In the same transaction, insert a processed-message or business-operation key under a unique constraint and apply a valid state transition. Carry the same idempotency key to external providers; on ambiguous outcomes, query and reconcile rather than blindly repeat. State that any exactly-once claim is bounded to participating broker/state systems and does not automatically include external side effects.

## References

- [RabbitMQ — Consumer acknowledgements and publisher confirms](https://www.rabbitmq.com/docs/confirms)
- [Amazon SQS — Queue types](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-queue-types.html)
- [Apache Kafka — Documentation](https://kafka.apache.org/documentation/)

