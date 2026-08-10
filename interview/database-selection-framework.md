# Database Selection Framework for System Design Interviews

Do not answer:

> “SQL for ACID, NoSQL for scale.”

Choose a database from the workload.

---

## 1. Start With the Invariant

Ask:

```text
What can never be wrong?
```

Examples:

- one seat allocated once
- ledger balances reconcile
- username unique
- document update conflicts detected

This determines transaction/concurrency needs.

---

## 2. Natural Data Shape

### Relational

Good signal for PostgreSQL/MySQL:

- many meaningful relationships
- constraints
- joins
- flexible query combinations
- multi-row transaction boundaries

### Document

Good signal for MongoDB:

- self-contained aggregate
- nested data read/written together
- evolving category-specific attributes
- single-document atomicity fits invariant

### In-memory structures

Good signal for Redis:

- cache
- counter
- sorted ranking
- TTL state
- rate-limit state
- ephemeral coordination

Redis is not automatically the durable authority.

---

## 3. Access Patterns

List the top reads/writes before choosing technology.

Example:

```text
Get order by ID
List latest orders for user
Update order status
Reserve inventory
Aggregate monthly revenue
Full-text product search
```

One database may not optimally serve all of them.

Possible architecture:

```text
OLTP DB
+
Redis
+
search index
+
warehouse
```

with explicit ownership and replication.

---

## 4. Transaction Boundary

Ask:

```text
single row/document?
multiple rows?
multiple tables?
multiple shards?
multiple services?
```

The wider the invariant boundary, the more coordination is required.

Prefer data co-location when it materially simplifies correctness.

---

## 5. Consistency Requirement

Choose actual semantic:

- linearizable/strong decision
- read-your-writes
- snapshot
- bounded staleness
- eventual consistency

Do not classify the entire product as strong/eventual.

---

## 6. Query Flexibility

Need:

```text
ad hoc relational filtering
joins
aggregations
multiple secondary access paths
```

Relational SQL is a strong fit.

Need:

```text
known document access patterns
aggregate-local reads
```

document model may simplify storage.

Need:

```text
text relevance
fuzzy matching
faceting
```

use a search engine/index rather than forcing the OLTP database to be every system.

---

## 7. Scale Dimensions

Do not say only “millions of users.”

Estimate:

```text
read QPS
write QPS
bytes/write
dataset
working set
growth/day
largest tenant
largest partition
regions
fan-out
```

Scaling bottleneck may be:

- CPU
- connections
- storage IOPS
- WAL/binlog
- hot key
- hot shard
- cross-shard query
- memory
- replication bandwidth

---

## 8. Partition Key

If horizontal partitioning/sharding may be required, ask early:

```text
What key distributes writes?
What key appears in common reads?
Can one tenant dominate?
Is the key monotonic?
Will queries scatter/gather?
```

A database with built-in sharding does not rescue a bad shard key.

---

## 9. Index Model

Ask:

- primary access key
- secondary indexes
- compound indexes
- write amplification
- full-text needs
- index rebuild/migration

A write-heavy workload with 20 secondary indexes may fail regardless of database brand.

---

## 10. Availability and Failover

Define:

```text
RPO
RTO
regional failure requirement
read/write behavior during failover
```

Then choose:

- asynchronous replica
- synchronous replica
- quorum
- multi-region ownership
- managed HA topology

---

## 11. Operational Maturity

Evaluate:

- team expertise
- managed-service quality
- backup/restore
- observability
- schema migration
- online index build
- failover tooling
- cost
- compliance

A technically elegant database that the team cannot operate is a poor system-design choice.

---

# Decision Examples

## Payments ledger

Likely priorities:

```text
strong invariants
transactions
auditability
durability
```

Strong relational fit.

Redis/search may be projections, not authority.

---

## Product catalog

Possible priorities:

```text
document-like attributes
category variation
read-heavy
search
```

Could use:

- PostgreSQL + JSONB
- MongoDB
- relational catalog + search index

Decide from query/transaction shape, not schema-flexibility slogans.

---

## Social feed

Likely split:

```text
authoritative posts → durable DB
fan-out/feed materialization → key-value/wide-column/cache
search → search index
analytics → warehouse
```

One database does not need to do everything.

---

## Rate limiter

State:

```text
small
ephemeral
high-frequency
atomic counter/TTL
```

Redis is often a strong fit.

But define fail-open/fail-closed and multi-region semantics.

---

# Comparison Checklist

| Dimension | PostgreSQL/MySQL | MongoDB | Redis |
|---|---|---|---|
| Core model | Relational | Document | Data structures / in-memory |
| Multi-row relations | Excellent | Possible, less natural | Not primary role |
| Single aggregate document | Good with JSON/relations | Natural | Possible but memory-oriented |
| Transactions | Mature | Supported | Limited/local semantics |
| Secondary queries | Rich | Rich document indexes | Structure/key dependent |
| Horizontal distribution | Architecture/product dependent | Built-in sharding | Cluster sharding |
| Durable system of record | Common | Common | Possible but requires deliberate durability design |
| Cache/TTL | External/limited use | TTL features possible | Excellent |
| Full-text relevance | Moderate DB features | Text/search options vary | Not primary fit |

This table is only a starting point.

---

# Interview Answer Template

> “The invariant is transactional across orders and inventory, and the main access patterns require indexed relational queries, so I’ll start with PostgreSQL/MySQL rather than choose a sharded store prematurely. I expect roughly X write QPS and Y GB/year, which one primary can plausibly handle with proper indexing and pooling. Redis is only for hot derived reads. Search is a separate projection if relevance/faceting becomes important. If write throughput later exceeds a single-primary limit, I’ll revisit partition ownership using the dominant tenant/customer key rather than claiming horizontal scaling is free.”

---

# Red Flags

Avoid these answers:

```text
SQL cannot scale
NoSQL has no transactions
MongoDB is schema-less
Redis is always sub-millisecond
microservices need one DB each
read replicas are strongly consistent
sharding is unlimited scaling
```

State workload-specific guarantees instead.
