# Distributed Systems Fundamentals

## TL;DR

Once state or work spans failure boundaries, success, failure, and time become observations rather than universal facts. Design around partial failure: use explicit deadlines, stable operation identities, bounded coordination, authoritative state, monotonic versions, and reconciliation. Minimize the consistency boundary before choosing a protocol.

## What distribution changes

Inside one process, a function either returns or throws under a mostly shared clock and memory model. Across a network:

- a request may arrive after its caller times out;
- the server may commit and its response may be lost;
- two healthy nodes may be unable to communicate;
- messages may be duplicated, delayed, or reordered;
- a paused process may resume believing it still owns a lease;
- replicas may disagree until repair completes.

Therefore, “the call failed” is often too strong. A timeout means the caller stopped waiting; the outcome can be unknown.

## Start with invariants and authority

An invariant is a rule that must remain true, such as “a seat has at most one active owner” or “captured amount never exceeds authorized amount.” Assign one serialization authority for each hard invariant: a database row/version, a partition leader, or a consensus group.

Avoid coordinating facts that do not share an invariant. User preferences and analytics counters can often converge independently; inventory ownership may not.

```text
business rule -> consistency boundary -> authority -> protocol -> failure recovery
```

## Safety and liveness

- **Safety:** nothing bad happens—two clients do not both obtain the same exclusive seat.
- **Liveness:** something good eventually happens—a valid buyer can eventually reserve a seat.

During a partition, a design may preserve safety by refusing writes, sacrificing liveness. A system that grants both sides ownership preserves local liveness but violates safety. State which property the business protects under each failure.

## Failure model

Define the failures the design claims to tolerate:

| Failure | Typical mechanism |
|---|---|
| Process crash | durable log, retry, idempotency |
| Node or zone loss | replicas across failure domains, failover |
| Network partition | quorum/ownership rule, bounded degradation |
| Slow dependency | deadlines, cancellation, bulkheads, load shedding |
| Duplicate/reordered message | IDs, versions, commutative operations |
| Disk/data corruption | checksums, backups, independent restore tests |
| Operator/configuration error | staged rollout, audit, blast-radius controls |

Replication does not replace backup: a bad delete or corrupt write can be faithfully copied.

## Coordination versus convergence

Coordination orders decisions before they are visible. It protects hard invariants but adds latency and can become unavailable without a quorum. Convergence allows replicas to accept work and reconcile later; it improves local availability but requires explicit conflict semantics.

Useful convergence techniques include last-writer rules only when clock/overwrite semantics are acceptable, per-field merges, version vectors, CRDTs for suitable data types, and application reconciliation. “Eventually consistent” is incomplete without a convergence trigger, conflict rule, and staleness bound or objective.

## Common failure modes

- Treating retries as new operations instead of the same idempotent intent.
- Using wall-clock timestamps as a universally correct order.
- Replicating synchronously across every region on a latency-sensitive path.
- Electing a new owner without fencing the old one.
- Claiming high availability while the only database or control plane is a single failure domain.
- Designing steady state but not resharding, replay, repair, or regional evacuation.

## Interview prompts

- What invariant requires coordination?
- Which node is authoritative, and how is that authority proven?
- What does the system do during a partition?
- How is an ambiguous timeout reconciled?

## Two-minute answer

Identify the hard invariants and make their consistency boundary as small as possible. Give each boundary an authority—often a leader or conditional database write—and use idempotent operation IDs for ambiguous retries. Replicate across named failure domains, state quorum and failover behavior, use versions rather than clocks for causality, and fence stale owners. Let non-critical derived state converge asynchronously with an explicit repair loop. Then cover overload, backup/restore, observability, and the exact behavior during partition.

