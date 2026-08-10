# SQL Joins for System Design

Knowing `INNER JOIN` versus `LEFT JOIN` is table stakes.

For senior system design, joins matter because they expose:

- data ownership,
- cardinality,
- index strategy,
- query-plan behavior,
- network boundaries,
- and the point where relational composition should move to a projection/search/analytics system.

---

## Interview TL;DR

1. A join is not inherently slow; bad cardinality, missing access paths, large intermediate results, and poor estimates are the real problem.
2. Join performance depends on **rows entering the join**, not only total table size.
3. Nested loop, hash join, and merge join are algorithms, not “good/bad” labels.
4. Join order is commonly optimizer-chosen for inner joins; outer joins constrain legal reorderings because semantics differ.
5. Index join/filter columns when the access pattern justifies it, but do not index every foreign key blindly without workload evidence.
6. Joining two tables in the same database is very different from an application “joining” data across services.
7. Avoid cross-service synchronous fan-out masquerading as a distributed join.
8. Denormalize intentionally when the read model needs bounded, stable duplication.
9. Use pagination before a one-to-many join explodes response size.
10. Check actual execution plans and row estimates.

---

# 1. Join Semantics

## INNER JOIN

Return matching pairs.

```sql
SELECT o.id, u.email
FROM orders o
JOIN users u
  ON u.id = o.user_id;
```

## LEFT JOIN

Return all left rows plus matches.

Useful for optional related state.

## FULL OUTER JOIN

Useful for:

- reconciliation
- migration validation
- diff/audit workflows

Support varies by database.

## CROSS JOIN

Cartesian product.

```text
A rows × B rows
```

Useful intentionally, dangerous accidentally.

---

# 2. Cardinality First

Suppose:

```text
100 users
×
1,000 orders/user
×
5 items/order
```

A naive multi-table result can become:

```text
500,000 rows
```

before returning one page to a client.

The main question is:

> How many rows exist at every stage of the plan?

---

# 3. One-to-Many Result Explosion

Query:

```sql
users
JOIN orders
JOIN order_items
JOIN product_tags
```

One logical user can appear hundreds or thousands of times.

Consequences:

- network payload
- ORM object duplication
- sort/hash memory
- serialization cost

Sometimes use:

- two-phase fetch
- aggregation
- JSON aggregation
- precomputed view
- separate endpoint
- pagination at correct entity level

---

# 4. Nested Loop Join

Conceptually:

```text
for each outer row:
    find matching inner rows
```

Excellent when:

```text
outer rows = small
inner lookup = indexed
```

Example:

```text
20 orders
×
PK lookup customer
```

Bad when:

```text
1,000,000 outer rows
×
expensive inner scan
```

Look at loops and row count.

---

# 5. Hash Join

Conceptually:

```text
build hash table on one input
      ↓
probe using other input
```

Strong for many equality joins where inputs fit available memory reasonably.

Risk:

- large memory
- spill/batching
- huge build side

A hash join is not a substitute for reducing unnecessary input rows.

---

# 6. Merge Join

Conceptually:

```text
two ordered streams
      ↓
advance through keys
```

Useful when:

- inputs are already ordered by join key
- sorting is affordable
- large equality/range-compatible workloads fit the engine's strategy

Do not memorize “merge join for large tables” as an unconditional rule.

---

# 7. Join Order

For inner joins, optimizers can often reorder joins.

Goal:

```text
filter/selective work early
      ↓
smaller intermediate sets
```

Outer joins constrain reordering because preserving unmatched rows changes semantics.

Example:

```sql
A LEFT JOIN B ...
```

is not freely interchangeable with arbitrary reordered joins.

---

# 8. Statistics and Cardinality Estimation

A bad estimate:

```text
estimated 100 rows
actual 2,000,000 rows
```

can lead to:

- wrong join order
- nested loop where hash would be better
- undersized memory
- spill
- bad parallelism choices

Fix:

- statistics
- data skew understanding
- correlated-column stats/features where supported
- query shape

before forcing a plan.

---

# 9. Indexing Join Paths

Foreign-key relationship:

```text
orders.user_id → users.id
```

Parent PK is already indexed.

The child FK may need an index when queries frequently:

- join from user to orders
- delete/update parent and must check children
- filter child by parent
- lock child sets

But “every foreign key must always have an index” is too absolute; workload and database semantics matter.

---

# 10. Filter Before Joining

Prefer:

```sql
SELECT ...
FROM (
  SELECT ...
  FROM orders
  WHERE user_id = ?
  ORDER BY created_at DESC
  LIMIT 20
) o
JOIN ...
```

when semantics permit, rather than joining all historical orders before limiting.

Be careful: optimizer transformations and outer-join semantics can change what is legal.

---

# 11. `ON` vs `WHERE` With Outer Joins

Classic correctness bug.

```sql
SELECT ...
FROM users u
LEFT JOIN orders o
  ON o.user_id = u.id
WHERE o.status = 'PAID';
```

The `WHERE` predicate removes rows where `o` is `NULL`, often making behavior equivalent to requiring a match.

If the intent is:

```text
all users
+
paid orders when present
```

use:

```sql
LEFT JOIN orders o
  ON o.user_id = u.id
 AND o.status = 'PAID'
```

Semantics before performance.

---

# 12. N+1 Is an Application Join Problem

```text
1 query: orders
100 queries: user
500 queries: item
```

Solutions:

- SQL join
- batch query
- eager load
- DataLoader
- cached reference data
- denormalized read model

But one massive join is not always the answer.

Measure:

```text
query count
rows returned
bytes transferred
duplicate parent data
DB CPU
```

---

# 13. Join vs Denormalization

Join when:

- data is independently mutable
- relational correctness matters
- query frequency/latency is acceptable
- database can efficiently compose it

Denormalize when:

- read path is extremely hot
- data changes less often
- duplication semantics are clear
- stale-window/rebuild strategy is defined

Example:

```text
order item stores product_name_at_purchase
```

This is domain history, not merely a performance shortcut.

---

# 14. Join vs Materialized Projection

For an expensive dashboard:

```text
orders
JOIN users
JOIN payments
JOIN refunds
GROUP BY ...
```

executed thousands of times,

consider:

```text
transactional source
      ↓
CDC/event pipeline
      ↓
materialized read model
```

Now the trade-off is:

```text
freshness
vs
query cost
```

---

# 15. Joins Across Services

Bad distributed composition:

```text
Order API
   ↓
User Service
   ↓
Product Service × 20
   ↓
Pricing Service × 20
   ↓
Inventory Service × 20
```

This is a latency/reliability fan-out problem, not a SQL join.

At service boundaries prefer:

- API composition with bounded fan-out
- batch endpoints
- BFF/aggregator
- cached reference data
- replicated projection
- event-driven read model

---

# 16. Data Ownership

A cross-service join often reveals unclear ownership.

Ask:

```text
Who owns the authoritative field?
Who may cache/project it?
What freshness is required?
What happens if owner is unavailable?
```

Do not duplicate mutable data without defining propagation and reconciliation.

---

# 17. Distributed Join vs Co-Location

In sharded systems:

```text
join keys co-located
```

can be far cheaper than:

```text
shuffle both datasets across network
```

Shard/partition key design may be influenced by frequent join locality.

But overfitting all data to one join can destroy other access patterns.

---

# 18. Pagination and Joins

If the API returns 20 orders:

paginate orders first where semantics allow.

Do not:

```text
join 10 million order-items
      ↓
then LIMIT 20 rows
```

and accidentally return only part of several orders.

Entity-level pagination must align with the UX entity.

---

# 19. Join Query Observability

Track:

- query fingerprint
- p95/p99
- actual rows
- loops
- temp/spill
- rows examined
- bytes returned
- lock wait
- join algorithm
- plan changes

The right plan can change as data distribution changes.

---

# 20. Common Mistakes

### “Joins do not scale”

Too vague.

### “NoSQL avoids joins, so it is faster”

It may move composition into application/network or duplicate state.

### “Index every join column”

Not automatically.

### “Hash join is faster than nested loop”

Depends on row counts and access path.

### “One SQL query is always better than multiple”

A gigantic duplicative result can be worse than two targeted queries.

### “Service calls are just joins over HTTP”

They have independent latency/failure/consistency semantics.

---

# Interview Answer Template

> “The order detail query is relational and bounded by one order ID, so a join is appropriate. I expect one order, tens of items, and one payment/shipment row, with PK/FK access paths. I’ll inspect the actual plan rather than assume the algorithm. For the order-history list I’ll paginate orders before loading child detail to avoid row explosion. If this became a cross-service page, I would not fan out one call per item; I’d use batching or a materialized projection with an explicit freshness contract.”

---

## References

- PostgreSQL EXPLAIN: https://www.postgresql.org/docs/current/using-explain.html
- MySQL join optimization: https://dev.mysql.com/doc/refman/8.4/en/nested-join-optimization.html
