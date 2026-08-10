# Query Optimization

Query optimization is not “add an index until EXPLAIN looks green.” It is a loop:

```text
measure
  ↓
identify dominant cost
  ↓
change query/schema/index
  ↓
measure again
```

Optimize the workload, not isolated SQL screenshots.

---

## Interview TL;DR

1. Start with **frequency × latency × resource cost**, not the single slowest query.
2. Compare estimated rows with actual rows; cardinality-estimation errors often cause bad plans.
3. A sequential scan is not automatically bad.
4. An index is not automatically good.
5. Optimize row count and bytes moved before micro-optimizing syntax.
6. Keyset pagination is usually better than deep offset for large ordered streams, but it requires deterministic ordering.
7. Read replicas move load but introduce staleness.
8. Caching can mask a query problem and create invalidation problems.
9. N+1 is an application access-pattern issue, not only a SQL issue.
10. Keep analytical scans away from latency-sensitive OLTP when they compete for the same resources.

---

# 1. Define the Problem

Measure:

- query fingerprint
- calls/sec
- p50/p95/p99
- rows examined
- rows returned
- CPU
- I/O
- temporary spill
- lock wait
- network bytes
- cache hit ratio

A 2-second query run once a night may matter less than a 40 ms query run 50,000 times/sec.

---

# 2. Execution Plan

Use the database's plan tools.

PostgreSQL:

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT ...
```

MySQL:

```sql
EXPLAIN
SELECT ...
```

Inspect:

```text
access path
join order
estimated rows
actual rows
loops
sort
temporary work
index condition
filter
```

---

# 3. Sequential Scan Is Not Automatically Bad

A full scan may be right when:

- table is small
- query returns a large fraction
- analytical query needs most rows
- random index access would cost more
- data is already memory-resident and scan is cheaper

Do not fight the optimizer solely because you see `Seq Scan`.

---

# 4. Select Fewer Rows

The highest-leverage optimization is often:

```text
touch less data
```

Use:

- more selective predicate
- tenant/partition key
- bounded date range
- pagination
- pre-aggregation

---

# 5. Select Fewer Columns

Avoid `SELECT *` on hot APIs when large payload columns exist.

Benefits:

- less table/index I/O
- less network
- less serialization
- possible covering/index-only access

But do not obsess over three tiny fixed-width columns in a query dominated by another bottleneck.

---

# 6. Correct Composite Index

Query:

```sql
SELECT id, status, total_amount, created_at
FROM orders
WHERE customer_id = ?
  AND status = ?
ORDER BY created_at DESC
LIMIT 20;
```

Candidate index:

```sql
(customer_id, status, created_at DESC)
```

Validate it against real distribution and workload.

---

# 7. Cardinality Estimation

Bad planner estimates can come from:

- stale stats
- correlated columns
- skew
- rapidly changing data
- expressions
- unusual predicates

Symptom:

```text
estimated rows: 100
actual rows: 2,000,000
```

That can change join choice and memory strategy.

Before adding hints/forcing behavior, fix the information problem when possible.

---

# 8. Join Strategy

Common algorithms:

- nested loop
- hash join
- merge join

No algorithm is universally best.

A nested loop can be excellent:

```text
10 outer rows × indexed inner lookup
```

and terrible:

```text
1,000,000 outer rows × expensive inner scan
```

Inspect row counts and loops.

---

# 9. N+1

Application:

```text
1 query for 100 orders
100 queries for customers/items
```

Fix with:

- join
- batched `IN`
- eager loading
- request-scoped DataLoader
- denormalized read model

Do not replace N+1 with one giant Cartesian join that duplicates huge payloads. Measure both.

---

# 10. Keyset Pagination

Deep offset:

```sql
LIMIT 20 OFFSET 1000000
```

often requires skipping/processing a large prefix.

Keyset:

```sql
WHERE (created_at, id) < (?, ?)
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

Use a unique tie-breaker.

Important:

```text
ORDER BY created_at alone
```

is not deterministic if many rows share the same timestamp.

Cursor should encode the complete ordering position.

---

# 11. Offset Pagination Still Has a Place

Good for:

- small admin datasets
- direct page-number navigation
- low-depth results
- stable snapshot/report

The choice is UX + workload, not dogma.

---

# 12. Functions on Indexed Columns

A function can prevent use of a normal index if the index does not match the expression.

Example:

```sql
WHERE lower(email) = ?
```

Possible fixes:

- expression/functional index
- normalized stored value
- case-insensitive collation/type as supported

Validate with plan.

---

# 13. Sort and Spill

Large sorts/aggregates can spill from memory to disk.

Look for:

- external sort
- temp files
- large hash batches
- memory grant/work-memory problems

Possible fixes:

- better index/order path
- reduce rows first
- bounded result
- more appropriate memory configuration
- pre-aggregation

---

# 14. Read Replicas

A query router may send stale-tolerant reads to replicas.

Do not assume this solves every read bottleneck.

New problems:

- replication lag
- read-after-write violation
- connection routing
- failover
- replica saturation
- long report queries delaying replay in some systems

---

# 15. Cache

Use cache when:

```text
high repeat rate
×
expensive source read
×
acceptable staleness
```

A cache adds:

- key correctness
- TTL
- invalidation
- stampede
- memory policy
- failure fallback

Fix an obviously bad query before building a complex cache around it.

---

# 16. Precompute

Useful for expensive repeated aggregates:

- materialized view
- summary table
- event-driven projection
- OLAP warehouse

Trade-off:

```text
write/update pipeline complexity
vs
fast reads
```

Define freshness SLO.

---

# 17. OLTP vs Analytics

OLTP wants:

- short transactions
- selective queries
- predictable latency

Analytics wants:

- large scans
- aggregations
- columnar/parallel execution
- high throughput

At scale, isolate them using:

- warehouse
- analytical replica
- CDC pipeline
- lakehouse
- materialized projections

Do not let a dashboard scan starve checkout.

---

# 18. Query Timeouts and Resource Guardrails

Protect the database with:

- statement timeout
- lock timeout
- page-size limit
- API complexity limit
- connection-pool queue
- workload isolation
- cancellation

A query should not run forever because a client disconnected.

---

# 19. Partition Pruning / Shard Routing

If a large table or distributed DB is partitioned, include the partition/routing key where practical.

Good routing:

```text
tenant_id + time range
    ↓
few partitions/shards
```

Bad routing:

```text
broadcast every shard
```

---

# 20. Query Regression

A query can regress without SQL changing because:

- table grew
- distribution changed
- stats changed
- cache warmed/cooled
- index changed
- database upgraded
- parameter values differ

Monitor query fingerprints over time.

---

# 21. Common Mistakes

### “Index scan always beats table scan”

False.

### “Select * is always the main problem”

Sometimes irrelevant compared with row count/join/spill.

### “Read replica makes reads scalable”

Only stale-tolerant reads, and replicas are finite resources.

### “Cursor pagination fixes all pagination”

It sacrifices arbitrary page jumps and requires stable ordering.

### “Caching fixes slow SQL”

It may only hide it.

### “Add database CPU”

If the query does 10,000× too much work, larger hardware delays the incident.

---

# Query-Optimization Workflow

```text
1. Find top query fingerprints.
2. Rank by frequency × latency/resource.
3. Inspect actual plan.
4. Compare estimated vs actual rows.
5. Check I/O, sort/spill, locks, loops.
6. Reduce rows/bytes.
7. Add/change index if justified.
8. Re-run with production-like parameters/data.
9. Watch write cost and regressions.
10. Deploy with before/after telemetry.
```

---

# Interview Questions

## Why might the database ignore an index?

The optimizer may estimate that a scan is cheaper, the predicate may return too many rows, the index order/expression may not match, or statistics may be misleading.

## Why can keyset pagination duplicate/skip incorrectly?

If the ordering is not deterministic or the cursor does not include all order-by tie-breakers.

## What is the first optimization for N+1?

Change the access pattern—batch or join related reads—then validate payload and plan costs.

## When should analytics leave the primary?

When large scans/aggregations materially interfere with latency-sensitive transactional work.

---

## References

- PostgreSQL EXPLAIN: https://www.postgresql.org/docs/current/using-explain.html
- PostgreSQL indexes: https://www.postgresql.org/docs/current/indexes.html
- MySQL optimization: https://dev.mysql.com/doc/refman/8.4/en/optimization.html
