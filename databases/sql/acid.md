# ACID: What It Guarantees — and What It Does Not

ACID is frequently explained as four definitions and then over-applied.

For system design, the important skill is understanding the **scope** of those guarantees.

ACID protects transaction processing inside the database's transactional boundary. It does not automatically make a distributed workflow correct.

---

## Interview TL;DR

1. **Atomicity**: a transaction's database changes commit or abort as a unit.
2. **Consistency**: transactions should preserve defined invariants; the database only enforces the invariants actually encoded or correctly implemented.
3. **Isolation**: concurrent transactions behave according to the selected isolation model; it does not mean “transactions never see each other.”
4. **Durability**: once the database reports commit success under its configured durability semantics, the committed state should survive covered failures.
5. ACID consistency is **not CAP consistency**.
6. Atomicity does not roll back an email, payment provider, Kafka publish, or another database unless those systems participate in a suitable distributed protocol.
7. Isolation is implementation-specific; the same isolation-level name can behave differently across databases.
8. Durability is not backup, and replication is not backup.
9. Constraints are first-class consistency mechanisms.
10. ACID is necessary for many workflows, but application correctness still requires idempotency, validation, and recovery design.

---

# 1. Atomicity

Atomicity means:

```text
all transaction changes commit
or
none of them become committed
```

Example:

```sql
BEGIN;

INSERT INTO orders(...);

UPDATE inventory
SET stock = stock - 1
WHERE product_id = ?
  AND stock > 0;

INSERT INTO outbox_events(...);

COMMIT;
```

If the inventory update fails business validation and the transaction rolls back, the other uncommitted DB changes are discarded.

---

# 2. Atomicity Is Not “Instant”

A transaction can take time.

Atomicity concerns committed outcome, not zero-duration execution.

Other transactions may:

- wait on locks
- see an older MVCC snapshot
- run concurrently
- later encounter a conflict

---

# 3. Atomicity Stops at the Transaction Boundary

This is not atomic:

```text
BEGIN database transaction
      ↓
update DB
      ↓
send email
      ↓
COMMIT
```

If commit fails, the email is still sent.

Likewise:

```text
charge card
+
write database
```

is not automatically one ACID action.

For cross-system reliability, use:

- outbox
- idempotency
- durable workflow
- compensation
- 2PC only when supported and justified

---

# 4. Consistency

ACID “C” is often misunderstood.

A transaction should take the database from one state satisfying the defined rules to another valid state.

Examples:

```text
balance >= 0
email is unique
order references valid customer
seat allocated at most once
```

But the database does not invent your business rules.

If an invariant is neither:

- encoded as a constraint,
- protected by transaction logic,
- nor enforced through correct concurrency control,

“ACID database” does not magically preserve it.

---

# 5. Constraints as Consistency

Examples:

```sql
CHECK (amount > 0)
```

```sql
UNIQUE (seat_id, show_id)
```

```sql
FOREIGN KEY (customer_id)
REFERENCES customers(id)
```

Prefer a database constraint when it naturally expresses the invariant.

It is more authoritative than:

```text
SELECT to check
then INSERT
```

which has a race.

---

# 6. Cross-Row Business Rules

Some rules do not fit a simple constraint.

Example:

```text
at least one doctor remains on call
```

Protect with:

- Serializable isolation
- explicit invariant/guard locking
- redesigned state representation
- constraint technique where possible

A normal snapshot transaction can let individually valid writes combine into invalid state.

---

# 7. Isolation

Isolation controls the observable effects of concurrent transactions.

It can involve:

- MVCC snapshots
- row locks
- range/predicate protection
- conflict detection
- transaction abort/retry

Isolation is a spectrum of semantics and implementation choices.

See `isolation-levels.md`.

---

# 8. Isolation Does Not Mean No Concurrency

Serializable does not necessarily execute transactions one-by-one.

For example, PostgreSQL can execute Serializable transactions concurrently and abort dangerous combinations.

So:

```text
stronger isolation
```

does not always mean:

```text
global mutex
```

The actual database algorithm matters.

---

# 9. Durability

Durability means a successful commit survives the class of failures covered by the database's configured durability guarantees.

Mechanisms may include:

- WAL/redo log
- fsync/storage flush
- replication
- checksums/recovery
- checkpoints

But durability depends on:

- configuration
- storage behavior
- replication acknowledgement
- failover topology

---

# 10. Durability Has Levels in Real Architectures

Possible acknowledgement points:

```text
process memory
      ↓
local log buffer
      ↓
local durable log
      ↓
replica received
      ↓
replica durable
      ↓
cross-region durable
```

These have different latency and RPO.

Ask:

> What data loss is acceptable after the client received success?

---

# 11. Durability Is Not Backup

A durable replicated database can durably preserve:

```text
DROP TABLE
```

Replication may copy the mistake everywhere.

Backups/PITR protect against:

- operator error
- bad deployment
- malicious deletion
- logical corruption
- some disaster scenarios

Restore testing is part of correctness.

---

# 12. ACID vs CAP Consistency

ACID consistency:

```text
preserve database/application invariants
```

CAP consistency:

```text
distributed reads/writes behave like a strongly ordered single logical copy
```

They are different meanings of the word “consistency.”

Never use them interchangeably in an interview.

---

# 13. ACID vs BASE

“BASE” is often used as shorthand for systems accepting weaker/immediate consistency in exchange for availability or distributed flexibility.

Do not present:

```text
ACID = SQL
BASE = NoSQL
```

Modern systems mix:

- ACID transactions
- eventual projections
- strongly consistent key-value operations
- asynchronously replicated search indexes

Classify semantics per operation.

---

# 14. Example: Checkout

Local DB transaction can atomically:

```text
create order
reserve local inventory row
create payment-attempt record
create outbox event
```

It cannot atomically:

```text
capture Stripe payment
send email
reserve stock in another independent service
create FedEx shipment
```

Those need a workflow.

This is the ACID boundary an experienced interviewer expects you to identify.

---

# 15. Failure Matrix

| Failure | ACID helps? | Additional design |
|---|---:|---|
| SQL statement fails before commit | Yes | Rollback/error handling |
| Process crashes before commit | Yes | DB recovery |
| Client times out after commit | Partly | Idempotency/result lookup |
| Kafka publish lost after DB commit | No | Outbox |
| Event delivered twice | No | Idempotent consumer |
| Payment API succeeds then app crashes | No | Idempotency + reconciliation |
| Accidental production delete | No | Backup/PITR |
| Replica read stale | No | Read-routing consistency policy |

---

# 16. Common Mistakes

### “ACID means no race conditions”

Wrong isolation or application logic can still create anomalies.

### “Consistency means replicas agree”

That is a different consistency concept.

### “Commit means backed up”

No.

### “A transaction can include any business operation”

Only participating transactional resources share atomic commit.

### “Serializable removes need for idempotency”

Client retries are a separate problem.

### “Rollback can undo an external API”

No.

---

# Interview Answer Template

> “ACID applies inside the database transaction boundary. Atomicity ensures the order, inventory mutation, and outbox row commit together. Consistency is the set of invariants I actually enforce, such as non-negative inventory and unique idempotency keys. Isolation protects concurrent decisions; I’ll choose the narrowest mechanism that prevents the relevant anomaly. Durability defines what a reported commit survives, but backup and multi-region RPO are separate concerns. Remote payment and messaging are outside this ACID boundary, so they need idempotency, outbox, workflow state, and reconciliation.”

---

## References

- PostgreSQL transaction isolation: https://www.postgresql.org/docs/current/transaction-iso.html
- PostgreSQL transaction processing: https://www.postgresql.org/docs/current/transactions.html
- MySQL InnoDB transaction model: https://dev.mysql.com/doc/refman/8.4/en/innodb-transaction-model.html
