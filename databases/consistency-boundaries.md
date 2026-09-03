# Data Consistency Boundaries

This chapter connects PostgreSQL, MySQL, MongoDB, Redis, caches, and messaging into one system-design model.

The key question is:

> **Which system is authoritative for each invariant, and which other copies are projections that may lag?**

---

## Interview TL;DR

1. Every piece of mutable data needs an authoritative owner.
2. A cache is usually a projection, not the authority.
3. A search index is usually a projection.
4. A read replica is a copy with replication semantics, not an independent source of truth.
5. MongoDB/PostgreSQL/MySQL transactions only protect their own transaction boundary.
6. Redis atomic operations do not create a transaction with a SQL database.
7. Cross-system consistency needs versions, events, idempotency, reconciliation, or a distributed commit protocol.
8. “Eventually consistent” is incomplete until you define convergence, maximum acceptable lag, and failure repair.
9. Read-after-write requirements should be defined per user flow.
10. Reconciliation is a first-class component, not an admission of failure.

---

## 1. Authoritative vs Derived State

Example commerce system:

```text
Product DB        → authoritative product definition
Inventory DB      → authoritative available stock
Payment ledger    → authoritative payment state
Redis cache       → derived fast copy
Search index      → derived discovery copy
Analytics store   → derived reporting copy
```

A system can have many copies but should have a clear answer to:

> Which copy wins after disagreement?

---

## 2. Cache Consistency

Typical cache-aside:

```text
DB update
  ↓
cache invalidation
```

Failure modes:

- invalidation event lost
- old reader repopulates stale value
- Redis outage
- TTL too long
- replication lag inside cache topology

Use:

- TTL
- versioned values
- outbox/CDC invalidation
- request coalescing
- source fallback
- reconciliation where required

---

## 3. SQL DB + Redis Transaction

This is **not** one transaction:

```text
BEGIN SQL
UPDATE row
SET Redis key
COMMIT SQL
```

If Redis succeeds and SQL rolls back, or SQL commits and Redis fails, copies diverge.

Prefer:

```text
SQL = authority
commit SQL
then invalidate/refresh Redis
```

and design bounded staleness.

For cache data, eventual repair is often acceptable.

---

## 4. SQL DB + Kafka / Broker

Do not dual-write:

```text
commit DB
publish event
```

without handling the crash window.

Use transactional outbox:

```text
business row
+
outbox row
```

in one DB transaction.

Then publish asynchronously.

---

## 5. SQL DB + Search Index

Architecture:

```text
SQL source
  ↓
CDC/outbox
  ↓
indexer
  ↓
search
```

Search document includes:

```text
entity ID
source version
```

Indexer rejects old out-of-order updates.

Critical actions revalidate against source:

```text
search says product available
      ↓
checkout reads authoritative inventory
```

---

## 6. MongoDB + Search / Cache

Same principle.

MongoDB may be authoritative for a document aggregate.

Redis/search are derived views.

Change Streams can feed downstream views, but consumers still need:

- resume/recovery
- idempotency
- versioning
- delete handling
- reconciliation

---

## 7. Read-After-Write

User flow:

```text
POST /profile
      ↓
GET /profile
```

If GET routes to an asynchronous replica/cache, user may see old data.

Possible policies:

- route session to primary briefly
- cache/update local result from write response
- causal consistency/session semantics where supported
- version token and wait-until-applied
- accept stale UX with explicit contract

Define which flows require it.

---

## 8. Monotonic Reads

Problem:

```text
read version 20
then read version 18
```

after routing to a more-lagged replica.

For user-facing state, this can be worse than consistently stale reads.

Possible controls:

- sticky replica/session
- version watermark
- primary fallback
- causal session semantics

---

## 9. Versioned Projection

Source:

```text
product version = 42
```

Event:

```json
{
  "productId": "p1",
  "version": 42
}
```

Search currently has 43.

Then event 42 arrives late:

```text
42 < 43
      ↓
ignore
```

This is essential for asynchronous projections.

---

## 10. Reconciliation

No distributed pipeline is perfect forever.

Run periodic checks:

```text
source count/version/checksum
      ↓
compare projection
      ↓
repair drift
```

Examples:

- search documents missing
- cache records impossible to invalidate
- payment/order mismatch
- orphaned workflow state
- outbox stuck

Reconciliation is a safety net for failures outside happy-path retry windows.

---

## 11. Consistency SLO

Do not say only:

```text
eventually consistent
```

Define:

```text
99.9% of product updates searchable within 5 seconds
99.99% within 60 seconds
```

or:

```text
payment confirmation must be current immediately
```

This turns consistency into an observable requirement.

---

## 12. Failure Matrix

| Copy/System | Authority? | May lag? | Repair |
|---|---:|---:|---|
| Primary order DB | Yes | No by definition | backup/PITR |
| Read replica | No | Yes | replication |
| Redis cache | No | Yes | TTL/invalidation |
| Search index | No | Yes | replay/reindex |
| Analytics warehouse | No | Yes | ETL/CDC replay |
| Workflow projection | Depends | Yes | event replay/reconciliation |

---

## Interview Answer Template

> “I’ll name one authoritative owner for each invariant. Inventory is authoritative in the transactional inventory store; Redis and search may contain projections. The inventory write commits locally and records an outbox event. Cache invalidation and search updates happen asynchronously with entity versions so stale events cannot overwrite newer state. Search is allowed a five-second freshness SLO, but checkout always revalidates stock against the authority. A reconciliation job detects missing projections and stuck outbox events.”
