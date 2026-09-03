# Databases

Database choice should follow access patterns, consistency requirements, transactions, indexes, partitioning, and operational constraints.

## SQL

- [PostgreSQL](sql/postgresql.md)
- [MySQL](sql/mysql.md)
- [Indexing](sql/indexing.md)
- [Query Optimization](sql/query-optimization.md)
- [Transactions](sql/transactions.md)
- [ACID](sql/acid.md)
- [Isolation Levels](sql/isolation-levels.md)
- [Locks](sql/locks.md)
- [Joins](sql/joins.md)

## NoSQL / Specialized Stores

- [MongoDB](nosql/mongodb.md)
- [Redis](nosql/redis.md)

## Interview Rule

Do not answer “SQL or NoSQL?” based on category labels. Start with the operations the system must perform and the guarantees those operations require.

## Cross-System Consistency

- [Data Consistency Boundaries](consistency-boundaries.md)

## Related Curricula

- [Messaging Systems](../messaging/README.md) applies transactions, outbox, CDC, and idempotency across asynchronous boundaries.
- [Distributed Systems](../distributed-systems/README.md) generalizes replication, quorums, partitioning, consensus, and ownership.
- [Caching](../caching/README.md) treats databases as authoritative origins and makes staleness/fallback explicit.
- [Payment System](../case-studies/07-payment-system.md) and [Ticket Booking](../case-studies/08-ticket-booking.md) apply transactional invariants under external failure and contention.
