# Locks and Coordination

“Use a lock” is incomplete system-design reasoning.

First identify **where the shared state lives** and choose the narrowest concurrency mechanism that the authoritative resource can enforce.

There are several different things engineers call locks:

- in-process mutexes
- database row/table locks
- database advisory locks
- optimistic version checks
- distributed leases
- coordination-service locks

They do not provide interchangeable guarantees.

---

## Interview TL;DR

1. Prefer an invariant/atomic operation at the authoritative datastore over an external lock when possible.
2. Database lock behavior is vendor-specific.
3. In MVCC databases, ordinary reads do not necessarily take blocking shared row locks.
4. PostgreSQL row locks block writers/lockers on the same rows but ordinary `SELECT` can still read.
5. InnoDB locking statements can lock every scanned index record/range; index design therefore affects contention.
6. Do not claim that all databases escalate many row locks into a table lock; lock escalation is product-specific.
7. Deadlocks are expected; use consistent ordering, short transactions, good indexes, and retry.
8. Lock timeout and deadlock are different failure modes.
9. A distributed lease can expire while the old worker is still executing.
10. Use fencing/version tokens when stale workers could corrupt an external resource.
11. Locks and idempotency solve different problems.
12. Use `SKIP LOCKED` for queue-like work distribution only when intentionally skipping locked rows is acceptable.

---

# 1. Choose the Concurrency Primitive

For each invariant, consider:

```text
constraint
  ↓
atomic conditional update
  ↓
optimistic version
  ↓
database row/range lock
  ↓
Serializable transaction
  ↓
distributed coordination
```

Do not start with Redis/ZooKeeper if the SQL row itself can enforce the rule.

---

# 2. In-Process Mutex

Useful when:

```text
one process
one memory space
one shared object
```

It does not coordinate:

- another process
- another pod
- another region

A local mutex is insufficient for horizontally scaled service instances.

---

# 3. PostgreSQL Row Locks

Example:

```sql
SELECT *
FROM inventory
WHERE product_id = ?
FOR UPDATE;
```

PostgreSQL row-level locks prevent conflicting writers/lockers on the same row until transaction end.

Important:

> Ordinary reads are not blocked by row locks in the way a simplistic “exclusive lock blocks readers” explanation suggests.

MVCC readers can read the appropriate version.

---

# 4. PostgreSQL Row Lock Modes

Common modes:

- `FOR UPDATE`
- `FOR NO KEY UPDATE`
- `FOR SHARE`
- `FOR KEY SHARE`

Use the weakest mode that protects the operation.

Do not memorize the conflict matrix for interviews unless asked; understand that key-changing updates require stronger exclusion than ordinary non-key updates.

---

# 5. PostgreSQL Table Locks

DDL and maintenance can acquire table-level modes.

Some operations can strongly block normal workload.

Example classes:

```text
CREATE INDEX          → stronger table coordination
CREATE INDEX CONCURRENTLY → reduced blocking, extra work/constraints
ALTER TABLE variants  → lock depends on operation
VACUUM FULL           → strong exclusive behavior
```

Schema migration design is therefore a system-design concern.

---

# 6. PostgreSQL Advisory Locks

Advisory locks use application-defined lock IDs.

Useful for:

- one logical tenant job
- migration coordination
- scheduler singleton
- logical resources without a natural row

PostgreSQL offers:

- session-level advisory locks
- transaction-level advisory locks

Session-level locks survive transaction rollback until explicitly released/session end.

Transaction-level advisory locks release with transaction end.

Use the right lifecycle.

---

# 7. InnoDB Record and Range Locks

InnoDB locking reads and writes operate through indexes.

A critical detail:

> A locking statement generally locks the index records/ranges it scans, not only rows that the application conceptually thinks are “the answer.”

So:

```sql
UPDATE reservations
SET status = 'HELD'
WHERE room_id = ?
  AND starts_at >= ?
  AND starts_at < ?;
```

with a poor index can scan and lock a broader region.

Indexing is therefore also concurrency design.

---

# 8. Gap and Next-Key Locks

InnoDB can use next-key locking:

```text
record lock
+
gap before record
```

This helps protect ranges from conflicting inserts under relevant isolation/access patterns.

Trade-off:

- stronger range protection
- more blocking

Do not describe MySQL locking using only “row lock vs table lock.”

---

# 9. Lock Escalation Is Not Universal

Some database systems can escalate many fine-grained locks into coarser locks.

Do **not** teach:

```text
many row locks automatically become table lock
```

as a universal database rule.

For example, PostgreSQL does not use that generic automatic row-to-table escalation model.

Always name the database when discussing escalation.

---

# 10. Pessimistic Locking

Use when:

- conflict probability is high
- waiting is cheaper than repeated failed work
- one transaction must inspect mutable state then change it

Example:

```sql
BEGIN;

SELECT balance
FROM accounts
WHERE id = ?
FOR UPDATE;

-- validate + update

COMMIT;
```

Cost:

- queueing
- deadlocks
- timeout
- connection occupancy
- tail latency

---

# 11. Optimistic Concurrency

Version check:

```sql
UPDATE documents
SET body = ?,
    version = version + 1
WHERE id = ?
  AND version = ?;
```

Result:

```text
1 row → success
0 rows → conflict
```

Good when conflicts are uncommon.

Under high contention:

```text
many optimistic retries
```

can waste more work than a queue/lock/serialized owner.

---

# 12. Atomic Update Instead of Lock

Counter:

```sql
UPDATE counters
SET value = value + 1
WHERE id = ?;
```

Inventory:

```sql
UPDATE inventory
SET stock = stock - 1
WHERE id = ?
  AND stock > 0;
```

Often better than:

```text
SELECT FOR UPDATE
then update
```

when no intermediate read-dependent logic is required.

---

# 13. Deadlocks

Example:

```text
Tx A: lock account 1
Tx B: lock account 2

Tx A: wants account 2
Tx B: wants account 1
```

Mitigation:

- lock in consistent order
- keep transactions short
- use selective indexes
- avoid unnecessary resources
- retry aborted transaction

Deadlock handling is part of normal application behavior on busy systems. MySQL/InnoDB explicitly documents retrying transactions that are rolled back as deadlock victims. 

---

# 14. Lock Timeout vs Deadlock

### Deadlock

A dependency cycle exists.

Database may detect and abort a victim quickly.

### Lock timeout

No cycle is required.

A request simply waits too long for a resource.

These need different observability and sometimes different retry policy.

---

# 15. `NOWAIT`

Some databases support immediate failure instead of waiting.

Use when:

```text
resource busy
      ↓
return conflict / try another resource
```

is better than queueing.

---

# 16. `SKIP LOCKED`

Useful for worker queues:

```sql
SELECT id
FROM jobs
WHERE status = 'READY'
FOR UPDATE SKIP LOCKED
LIMIT 10;
```

Multiple workers can claim different unlocked jobs.

Do not use `SKIP LOCKED` for a query that is supposed to represent a complete consistent business set; it intentionally omits locked rows.

---

# 17. Distributed Locks

A distributed lock/lease can coordinate service instances.

Possible resource:

```text
job:month-end-close
```

Need:

- unique owner
- TTL/lease
- atomic acquisition
- safe release
- renewal strategy
- failure semantics
- fencing/version where correctness requires it

---

# 18. Lease Expiry Problem

Timeline:

```text
Worker A gets lease
      ↓
A pauses for 60 seconds
      ↓
lease expires
      ↓
Worker B gets lease
      ↓
A resumes
```

Now A and B may both act.

TTL alone does not prove exclusive execution.

---

# 19. Fencing Tokens

Issue monotonically increasing epochs:

```text
A gets token 41
B later gets token 42
```

Authoritative resource stores/accepts latest token.

If A resumes:

```text
token 41 < 42
      ↓
reject stale write
```

Fencing moves safety to the protected resource.

---

# 20. Distributed Lock vs Leader Election

Related but different.

Leader election:

```text
one process owns a role for an epoch
```

Lock:

```text
one owner temporarily controls one resource
```

A leader still needs fencing/term numbers if stale leaders can continue writing.

---

# 21. Lock vs Idempotency

Lock:

```text
prevent/serialize concurrent execution
```

Idempotency:

```text
make repeated logical request safe
```

Example checkout may need:

- atomic stock update
- idempotency key
- no distributed lock at all

Do not add a lock when a unique key solves the actual duplicate-request problem.

---

# 22. Hot Lock / Hot Row

A single popular counter or resource can serialize throughput.

Symptoms:

- rising lock wait
- low CPU but high latency
- one row dominates write traffic

Options:

- sharded counters
- batching
- partitioned ownership
- queue/serialized worker
- redesign invariant
- reduce critical section

---

# 23. Observability

Track:

- lock acquisition latency
- lock hold duration
- waiting sessions
- deadlock count
- lock timeout count
- aborted/retried transactions
- hot resources
- advisory lock count
- distributed lease expiration
- stale fencing rejection
- p95/p99 mutation latency

For PostgreSQL inspect `pg_locks` and blocking relationships.

For MySQL inspect lock/deadlock instrumentation and InnoDB status/performance schema as appropriate.

---

# 24. Common Mistakes

### “FOR UPDATE blocks all readers”

Not a portable statement.

### “Every SQL SELECT gets a shared row lock”

False in MVCC systems.

### “Row locks always become table locks at scale”

Vendor-specific, not universal.

### “Distributed lock guarantees exactly one execution”

Lease expiry can allow stale work.

### “Use Redis lock for database inventory”

Prefer DB invariant/atomic update when the DB is authoritative.

### “Deadlock means retry one statement”

The database may have rolled back the whole transaction; follow product semantics.

---

# Interview Answer Template

> “The authoritative state is the inventory row, so I would first avoid an external distributed lock and use an atomic conditional decrement. If the operation requires reading several mutable fields before deciding, I’d use a row lock or Serializable transaction depending on the invariant. I’ll keep the transaction short, index the locking predicate so the lock footprint is narrow, and retry deadlock/serialization failures according to the database semantics. A distributed lease is only for coordination outside that DB boundary, and correctness-critical external resources need fencing tokens.”

---

## References

- PostgreSQL explicit locking: https://www.postgresql.org/docs/current/explicit-locking.html
- MySQL InnoDB locks: https://dev.mysql.com/doc/refman/8.4/en/innodb-locks-set.html
- MySQL deadlocks: https://dev.mysql.com/doc/refman/8.4/en/innodb-deadlocks.html
