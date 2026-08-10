# Transaction Isolation and Concurrency Control

Isolation levels are not a four-row memorization table. The difficult part is connecting a **business invariant** to the actual concurrency behavior of the chosen database.

The same isolation-level name can have implementation-specific behavior across PostgreSQL, MySQL/InnoDB, SQL Server, Oracle, and other systems.

---

## Interview TL;DR

1. Start with the invariant: **what invalid final state must be impossible?**
2. Dirty read, non-repeatable read, and phantom read are only part of the anomaly model.
3. Senior interviews frequently hinge on **lost update, write skew, duplicate allocation, and stale decision-making**.
4. MVCC reduces read/write blocking but does not automatically make concurrent application logic correct.
5. PostgreSQL and InnoDB implement Repeatable Read differently.
6. Higher isolation is not the only tool: use constraints, atomic updates, row locks, optimistic version checks, and idempotency where appropriate.
7. Serializable transactions may abort; retry the **whole transaction**, not only the last statement.
8. Deadlocks and serialization failures are expected operational events.
9. Keep transactions short and do not hold locks/snapshots across slow network calls.
10. Measure lock waits, aborts, retries, and tail latency in production.

---

# 1. Start With the Invariant

Example:

```text
available seats = 1
```

Invalid final state:

```text
two successful bookings for the same seat
```

This is more useful than saying:

> “We need Repeatable Read.”

Possible protections include:

- unique constraint
- atomic conditional update
- row lock
- optimistic version
- Serializable transaction
- partition ownership

Choose the narrowest mechanism that reliably protects the rule.

---

# 2. Standard Phenomena

## Dirty read

Read data another transaction has not committed.

## Non-repeatable read

Read one row twice and observe a different committed value.

## Phantom

Repeat a predicate/range query and observe a different matching row set.

## Serialization anomaly

The final committed behavior cannot be explained by any serial one-at-a-time execution.

---

# 3. Lost Update

Unsafe read-modify-write:

```text
stock = 10

Tx A reads 10
Tx B reads 10

A writes 9
B writes 9
```

Expected logical result: 8.

Possible fix:

```sql
UPDATE inventory
SET stock = stock - 1
WHERE product_id = ?
  AND stock > 0;
```

Check affected rows.

---

# 4. Write Skew

Rule:

```text
at least one doctor must remain on call
```

Concurrent snapshot-based transactions:

```text
A sees B on-call → A goes off-call
B sees A on-call → B goes off-call
```

They update different rows, so ordinary row-write conflict detection may not catch the cross-row invariant.

Possible protections:

- Serializable isolation
- explicit locking of an invariant/guard row
- redesign state so the invariant is enforced by a constraint
- serialized ownership of the decision

---

# 5. Read Uncommitted

Weakest standard level.

Whether dirty reads truly occur depends on the database.

For example, PostgreSQL maps Read Uncommitted to Read Committed.

Do not describe vendor behavior from the generic ANSI name alone.

---

# 6. Read Committed

Typical model:

```text
statement 1 → snapshot A
other transaction commits
statement 2 → snapshot B
```

Useful for many short CRUD operations.

It can be perfectly safe when state transitions use:

- atomic predicates
- constraints
- explicit locks
- version checks

Read Committed is not “unsafe by definition.”

---

# 7. Repeatable Read

The name hides implementation differences.

## PostgreSQL

Repeatable Read uses snapshot-isolation-style behavior.

It prevents:

- dirty reads
- non-repeatable reads
- phantom reads

but can still permit serialization anomalies/write-skew-style problems.

Updating transactions can receive serialization failures and require retry.

## InnoDB

Repeatable Read is the default.

Ordinary consistent reads use a stable snapshot.

Locking reads/writes can use record, gap, and next-key locking.

Therefore a PostgreSQL Repeatable Read mental model should not be blindly applied to MySQL.

---

# 8. Serializable

Serializable means successfully committed concurrent transactions behave like some serial ordering.

Implementation strategies vary.

## PostgreSQL

Serializable Snapshot Isolation detects dangerous dependency structures and aborts transactions when required.

## Lock-heavy implementations

Other systems may achieve serializable behavior through stronger locking/range protection.

Trade-offs can include:

- retries
- blocking
- lock footprint
- tail latency
- lower contention tolerance

---

# 9. MVCC Is Not Isolation

MVCC is an implementation technique for keeping multiple row versions and serving snapshots.

It can support multiple isolation levels.

MVCC helps with concurrency, but business logic can still suffer from:

- write skew
- stale decisions
- lost update depending on the statement pattern
- long-lived snapshots
- version cleanup pressure

---

# 10. Pessimistic Locking

```sql
SELECT *
FROM inventory
WHERE product_id = ?
FOR UPDATE;
```

Use when:

- conflicts are likely
- the transaction needs to inspect then update the same state
- serialized access is acceptable

Costs:

- blocking
- deadlocks
- tail latency
- lock management

---

# 11. Optimistic Concurrency

Use a version:

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

Good for:

- low-contention data
- editors/profile updates
- APIs exposing an ETag/version

The conflict path is part of the product design.

---

# 12. Unique Constraints as Concurrency Control

Suppose a username must be unique.

Do not:

```text
SELECT username
if missing:
   INSERT
```

and assume the check is safe.

Use:

```sql
UNIQUE(username)
```

Then handle the conflict.

The database invariant closes the race.

---

# 13. Atomic State Transition

Example:

```sql
UPDATE jobs
SET status = 'RUNNING'
WHERE id = ?
  AND status = 'PENDING';
```

Checking affected rows can implement compare-and-set-style transitions.

Useful for:

- inventory decrement
- worker claim
- one-way state machines
- idempotent transitions

---

# 14. Deadlocks

Deadlock:

```text
Tx A owns lock X, waits Y
Tx B owns lock Y, waits X
```

Databases typically resolve it by aborting a victim.

Mitigate:

- consistent lock order
- short transactions
- selective indexes
- fewer rows/ranges locked
- retry policy

Do not attempt to “eliminate all deadlocks” with fragile application sequencing.

---

# 15. Serialization Failure Retry

Correct:

```text
attempt:
  BEGIN
  reread current state
  validate
  write
  COMMIT

if serialization/deadlock retryable:
  rollback
  backoff/jitter
  rerun entire transaction
```

Incorrect:

```text
retry only failed UPDATE using old application values
```

The old reads may no longer be valid.

Bound retries; sustained retry storms indicate contention or bad transaction design.

---

# 16. Idempotency Is Different

Isolation protects **concurrent database state**.

Idempotency protects **repeated business requests**.

Example:

```text
client sends payment
response times out
client retries
```

Even a serializable transaction can create two payments if the repeated request is treated as two distinct operations.

Use an idempotency key with a durable uniqueness constraint/state record.

---

# 17. Transaction Boundaries

Keep network calls outside DB transactions when possible.

Bad:

```text
BEGIN
lock inventory row
call payment provider for 8 seconds
COMMIT
```

This holds locks and a connection across unbounded external latency.

Better architectures often use:

- reservation state
- durable workflow
- transactional outbox
- compensation
- idempotent external requests

---

# 18. Isolation and Replicas

A transaction isolation level on the primary does not automatically solve stale reads from asynchronous replicas.

You must separately define:

- read-after-write routing
- replica lag tolerance
- causal/session policy
- primary fallback

Isolation and replication consistency are separate dimensions.

---

# 19. Isolation Selection Framework

| Requirement | Candidate approach |
|---|---|
| Simple independent CRUD | Read Committed |
| Stable report snapshot | Repeatable Read / snapshot |
| Conditional decrement | Atomic update |
| Edit conflict detection | Optimistic version |
| Hot single-row mutation | Row lock / atomic update |
| Cross-row invariant | Serializable or explicit invariant lock |
| Unique allocation | Unique constraint + transaction |
| Duplicate API retry | Idempotency key |

The final choice depends on the database implementation.

---

# 20. Observability

Track:

- transaction duration
- lock wait time
- lock timeout
- deadlocks
- serialization failures
- retry count
- rows locked/touched
- connections `idle in transaction`
- p95/p99 mutation latency
- aborted transaction rate

A design without a contention metric is incomplete.

---

# 21. Common Interview Mistakes

### “Serializable means one transaction at a time”

Not necessarily.

### “Repeatable Read is identical everywhere”

False.

### “MVCC means reads never block”

Locking reads, DDL, and implementation details still matter.

### “Pessimistic locking is always safer”

It can create deadlocks and throughput collapse under contention.

### “Optimistic locking is faster”

Only when conflicts are sufficiently rare.

### “Higher isolation fixes duplicate API requests”

That is usually an idempotency problem.

---

# Interview Answer Template

> “The invariant is that only one successful reservation can exist for this seat. I would enforce that in the database with a unique constraint/state transition rather than relying on a prior application read. If I need to inspect mutable capacity before updating it, I’ll use an atomic conditional update or lock the relevant row. For a cross-row invariant I’d consider Serializable and implement whole-transaction retry. I’ll keep the transaction short and monitor lock waits, deadlocks, and serialization retries.”

---

## References

- PostgreSQL isolation: https://www.postgresql.org/docs/current/transaction-iso.html
- MySQL/InnoDB isolation: https://dev.mysql.com/doc/refman/8.4/en/innodb-transaction-isolation-levels.html
- InnoDB locking: https://dev.mysql.com/doc/refman/8.4/en/innodb-locks-set.html
