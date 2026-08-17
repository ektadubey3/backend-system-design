# Distributed Systems

Distributed systems fail partially: one participant may be slow, unreachable, partitioned, duplicated, or observing stale state while another is healthy. Senior-level design starts by naming the authority, consistency boundary, and recovery behavior—not by drawing more services.

## Learning path

1. [Distributed systems fundamentals](distributed-systems-fundamentals.md) — partial failure, uncertainty, invariants, and failure domains.
2. [Time and logical clocks](time-and-logical-clocks.md) — why wall clocks do not establish causality.
3. [Replication and quorums](replication-and-quorums.md) — copies, lag, read/write quorums, and repair.
4. [Partitioning and consistent hashing](partitioning-and-consistent-hashing.md) — ownership, movement, hot shards, and resharding.
5. [Consensus and leader election](consensus-and-leader-election.md) — a replicated decision, terms, quorum, and safety.
6. [Coordination, leases, and fencing](coordination-leases-and-fencing.md) — safe ownership when processes pause or partitions occur.
7. [Multi-region data ownership](multi-region-data-ownership.md) — latency, sovereignty, failover, and conflict policy.
8. [Distributed-systems design framework](distributed-systems-design-framework.md) — interview sequence and failure tests.

## Canonical material reused

- [CAP theorem](../fundamentals/cap-theorem.md) and [PACELC](../fundamentals/pacelc.md) remain the consistency/latency foundations.
- [SQL transactions](../databases/sql/transactions.md) remains the canonical treatment of 2PC, idempotency, transactional outbox, saga, orchestration, and choreography.
- [SQL locks](../databases/sql/locks.md) remains the implementation-oriented treatment of distributed locks, lease expiry, and fencing tokens.
- [Distributed caching](../caching/distributed-caching.md) covers cache-specific consistent hashing and failure behavior.

## Questions every design should answer

- Which component is authoritative for each invariant?
- What can proceed when replicas cannot communicate?
- What does a timeout mean, and how is an ambiguous result resolved?
- Which operations require consensus, and which tolerate convergence?
- How is ownership proven after pause, failover, or network partition?
- How are replicas repaired and how is recovery progress observed?
