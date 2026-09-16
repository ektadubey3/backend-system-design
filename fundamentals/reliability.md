# Reliability

Reliability is the ability of a system to produce the intended outcome over time despite expected faults, retries, partial failures, and operational change.

A service can be reachable and still be unreliable if it duplicates a payment, loses accepted work, corrupts state, or returns inconsistent results.

## Interview TL;DR

1. Define the **correct outcome** before selecting reliability mechanisms.
2. Reliability includes correctness, durability, controlled failure, and recoverability.
3. Retries are safe only when the operation and retry policy are safe.
4. Idempotency protects repeated business requests/events.
5. Redundancy helps only with independent failure domains and safe failover.
6. Backups and restore tests protect against failures replicas may copy.
7. Reconciliation repairs cross-system states outside one transaction.
8. Reliability requires observability of stuck, duplicate, stale, and partial work.
9. Recovery time matters as much as prevention.
10. Prefer simple local invariants over distributed coordination.

## Reliability Starts With Invariants

Examples:

```text
one payment request creates at most one charge
accepted notification is not silently lost
inventory never violates its reservation rule
an order eventually reaches a terminal/recoverable state
```

## Failure Categories

### Crash failure

Process/node stops.

Defenses:

- restart;
- replication;
- failover;
- durable logs.

### Timeout / uncertain outcome

Caller may not know whether remote work happened.

Defenses:

- deadlines;
- idempotency;
- status lookup;
- retry;
- reconciliation.

### Overload

System is alive but cannot keep up.

Defenses:

- admission control;
- load shedding;
- backpressure;
- priorities.

### Logical failure

Software performs the wrong action.

Replicas may reproduce it.

Defenses:

- invariants;
- validation;
- testing;
- audit trail;
- backup/PITR;
- repair.

## Retries

Retry only when:

- failure is plausibly transient;
- operation is safe to repeat or protected by idempotency;
- deadline permits it;
- attempts are bounded.

Use backoff and jitter.

## Idempotency

```text
Idempotency-Key: payment-order-5001
```

Persist a scoped key (for example tenant + operation + client key), request fingerprint, and operation status/result. Reject reuse with a different payload. Enforce uniqueness and commit the key plus the local business mutation atomically; a separate “check then insert” races under concurrency. A duplicate still in progress needs a documented pending/status-lookup response.

A local transaction does not include a payment provider. Carry a stable provider key, persist the provider operation ID, and reconcile ambiguous outcomes. Retention must cover the supported retry/replay horizon; after expiry, duplicate protection may no longer hold. Memory-only deduplication does not survive process failure. See [delivery semantics and idempotency](../messaging/delivery-semantics-and-idempotency.md) for consumer transactions and external effects.

## Durable Async Work

For accepted async work:

```text
receive
  ↓
durably persist / enqueue
  ↓
acknowledge
```

Do not acknowledge first and hope persistence succeeds afterward. “Accepted” means durable responsibility for processing, not completed business work; expose status and a completion SLO. If acceptance updates a database and publishes an event, use a [transactional outbox](../messaging/transactional-outbox-and-cdc.md) or another explicit atomicity protocol instead of an unprotected dual write.

## Reconciliation

Example:

```text
payment provider = CAPTURED
order DB         = PAYMENT_PENDING
```

A reconciliation process compares authoritative systems and repairs or alerts.

Include:

- retry;
- stuck/dead-letter state;
- replay;
- reconciliation;
- manual recovery.

## Backups

Replication protects selected infrastructure failures.

Backup/PITR protects against:

- accidental delete;
- bad migration;
- malicious change;
- logical corruption.

Define RPO, RTO, and restore-test practice.

## Observability

Track:

- duplicate operations;
- retry rate;
- queue oldest age;
- stuck workflow age;
- reconciliation mismatch;
- failover count;
- restore-test success.

## Reliability vs Availability

Availability:

> Can I successfully serve the operation now?

Reliability:

> Does the system continue producing correct, recoverable outcomes over time?

## Common Mistakes

- retries without idempotency or a budget;
- replication mistaken for backup;
- acknowledging before durable acceptance;
- no handling for uncertain timeout outcomes;
- no reconciliation for external side effects;
- defining reliability only as MTBF.

## Interview Answer Template

> “I define reliability from the business invariant. Externally retried operations use durable idempotency. Accepted async work is persisted before acknowledgement and consumers tolerate duplicates. Calls use deadlines and bounded retries for transient faults, while reconciliation covers cross-system states that cannot share one transaction. Replication and backup solve different failure classes.”

## Continue Without Repeating the Fundamentals

This chapter owns the business-outcome model. Use [availability](availability.md) for eligible-operation success, [fault tolerance](fault-tolerance.md) for specified faults, and [reliability engineering](../reliability/README.md) for operational controls and recovery drills. Apply them together in the [scenario study guide](../interview/scenario-study-guide.md).

## References

- [Google SRE — Addressing Cascading Failures](https://sre.google/sre-book/addressing-cascading-failures/)
- [Google SRE — Data Integrity](https://sre.google/sre-book/data-integrity/)
