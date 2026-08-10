# Trade-off Analysis

System design is all about choosing a design whose **benefits, costs, failure modes, and operational burden match the requirements**.

A senior engineer should be able to explain not only:

> “What would I use?”

but:

> “Why this, under these assumptions, and what does it make worse?”

---

## Interview TL;DR

For every major decision, answer six questions:

1. **Driver** — What requirement creates the decision?
2. **Options** — What realistic alternatives exist?
3. **Gain** — What improves?
4. **Cost** — What becomes worse or more complex?
5. **Failure mode** — How can this decision fail?
6. **Mitigation / trigger** — How do we control the downside, and when would we revisit it?

Use this structure:

```text
Requirement
    ↓
Options
    ↓
Decision
    ↓
Benefit
    ↓
Cost / Risk
    ↓
Mitigation
    ↓
Metric or evolution trigger
```

---

## Trade-offs Are Not Always Binary

Interview answers often become too simplistic:

```text
SQL vs NoSQL
Monolith vs Microservices
Consistency vs Availability
Vertical vs Horizontal Scaling
```

Most production decisions are multidimensional.

A database choice may depend on:

- access patterns
- transaction boundaries
- consistency model
- indexing
- partition key
- query flexibility
- write throughput
- read latency
- replication topology
- operational maturity
- cost
- recovery requirements

Do not turn an architecture decision into a slogan.

---

# Decision Framework

Use this table during an interview.

| Question | What to ask |
|---|---|
| Requirement | Which user/business goal matters? |
| Scale | QPS, storage, concurrency, regions, growth |
| Correctness | Which invariants must never break? |
| Latency | p50/p95/p99 target and geography |
| Availability | Which operations may fail or degrade? |
| Durability | What data loss is acceptable? |
| Freshness | How stale may reads be? |
| Operations | Who runs this and how complex is recovery? |
| Cost | What resource or vendor cost appears? |
| Evolution | What future scale changes the decision? |

---

# 1. Consistency vs Availability

This trade-off is most precise when discussed **during communication failure**.

Do not say:

> “Banking is CP and social media is AP.”

Instead identify the operation.

Example:

| Operation | Consistency need |
|---|---|
| Ledger mutation | Strong |
| Account activity projection | Can lag |
| Fraud analytics | Eventual |
| Notification | Eventual |
| Recommendation | Eventual |

Strong consistency improves correctness but may require coordination and reduce availability or increase latency.

Availability-first behavior keeps serving requests but requires a divergence/convergence model.

See [CAP Theorem](cap-theorem.md).

---

# 2. Latency vs Freshness / Consistency

A local replica can often respond faster than a globally coordinated read.

```text
Local read
  ↓
10-30 ms
  ↓
may be stale
```

versus:

```text
Coordinate with authoritative replica/quorum
  ↓
higher latency
  ↓
stronger freshness guarantee
```

Useful questions:

- Does the user need read-your-writes?
- Is bounded staleness acceptable?
- Can the UI show “last updated”?
- Is this an invariant or just a preference?

Do not use invented latency numbers as universal facts. State them as assumptions.

---

# 3. Read Optimization vs Write Optimization

### Read-optimized techniques

- caches
- replicas
- denormalization
- materialized views
- secondary indexes
- precomputation

Costs may include:

- invalidation
- replication lag
- write amplification
- storage duplication
- complex rebuilds

### Write-optimized techniques

- append-oriented storage
- batching
- partitioned logs
- asynchronous indexing
- fewer synchronous secondary indexes

Costs may include:

- slower reads
- delayed visibility
- compaction
- more complex query serving

The right answer depends on the read/write shape, not the product name.

---

# 4. SQL vs NoSQL

This should **not** be taught as:

```text
SQL = ACID
NoSQL = scalability
```

Both families contain systems with very different consistency, transaction, and scaling behavior.

## Relational databases are attractive when

- relationships and constraints matter
- transactions span related records
- ad hoc querying matters
- schema semantics are valuable
- mature tooling is useful

## Non-relational models are attractive when

- access patterns fit key-value/document/wide-column semantics
- partition-local operations dominate
- schema flexibility is valuable
- a specialized storage model materially improves workload fit

## Real decision criteria

Ask:

```text
What are my access patterns?
What is the transaction boundary?
What is the partition key?
Do I need joins or flexible queries?
What consistency is required?
What is the largest hot partition?
How will the database scale operationally?
What happens during rebalancing?
```

Modern distributed SQL systems can scale horizontally. Modern NoSQL systems can provide transactions and strong consistency.

The label alone does not answer the design question.

---

# 5. Monolith vs Microservices

A modular monolith is often the best starting point when the organization does not yet need independent operational boundaries.

## Monolith / modular monolith

**Benefits**

- one deployment
- local calls
- simpler debugging
- easier transactions
- lower operational overhead

**Costs**

- larger failure/deploy blast radius
- independent scaling can be harder
- ownership can deteriorate without module boundaries

## Microservices

**Benefits**

- independent deployment
- clearer service ownership
- independent scaling for real hotspots
- stronger runtime boundaries

**Costs**

- network failure
- distributed tracing
- contract/version management
- data ownership complexity
- retries and idempotency
- distributed workflows
- more infrastructure

The key question is:

> Does the organization need independent operational ownership enough to justify distributed-system complexity?

“Large company = microservices” is not a design rule.

---

# 6. Vertical vs Horizontal Scaling

## Vertical scaling

Increase resources on a node.

```text
8 CPU / 32 GB
      ↓
32 CPU / 128 GB
```

**Benefits**

- simple
- no partitioning
- fewer network hops

**Limits**

- hardware ceilings
- larger failure domain
- cost curve
- maintenance/failover implications

## Horizontal scaling

Add nodes.

```text
Node A
Node B
Node C
Node D
```

**Benefits**

- aggregate capacity
- can improve fault tolerance
- independent partition processing

**Costs**

- coordination
- load balancing
- partitioning
- replication
- distributed transactions
- rebalancing
- hot shards
- operational complexity

Do **not** call horizontal scaling unlimited.

Eventually bottlenecks move to:

- the database
- coordination metadata
- a leader
- a partition key
- network bandwidth
- shared storage
- downstream dependencies

---

# 7. Cache Speed vs Freshness and Complexity

Caching can reduce latency and database load.

It also adds a second copy of data.

```text
Database = source of truth
Cache = derived copy
```

New questions appear:

- cache key
- TTL
- invalidation
- race conditions
- stampede
- hot keys
- memory limit
- eviction
- cache outage
- stale reads

A cache is worthwhile when:

```text
read frequency × recomputation/read cost
```

is high enough to justify the consistency and operational complexity.

---

# 8. Normalization vs Denormalization

## Normalization

**Gain**

- fewer duplicate facts
- easier invariant enforcement
- lower update fan-out

**Cost**

- joins
- more read-time composition
- potentially more round trips in distributed ownership models

## Denormalization

**Gain**

- faster/precomputed reads
- access-pattern-specific data shape

**Cost**

- duplicate state
- propagation lag
- reconciliation
- rebuild strategy

Denormalization is often a deliberate read model, not “bad schema design.”

---

# 9. Synchronous vs Asynchronous Work

## Synchronous

```text
Request
  ↓
perform work
  ↓
return final result
```

**Benefits**

- simple semantics
- immediate failure visibility
- easier read-your-writes

**Costs**

- longer request latency
- dependency failures propagate
- limited tolerance for slow work

## Asynchronous

```text
Request
  ↓
persist intent/event
  ↓
acknowledge
  ↓
worker processes later
```

**Benefits**

- protects request path
- absorbs bursts
- supports retries
- independent consumer scaling

**Costs**

- eventual completion
- idempotency
- ordering
- duplicate delivery
- status tracking
- DLQ and replay operations

Good candidates specify what must be durable before acknowledging the user.

---

# 10. Durability vs Latency

A write can be acknowledged after different durability points:

```text
memory
local log
local fsync
replicated log
quorum commit
cross-region durable copy
```

Moving acknowledgement later can reduce data-loss risk but increase latency.

Use RPO/RTO and business impact.

Example:

| Data | Possible durability posture |
|---|---|
| Analytics event | small loss may be tolerable |
| Shopping cart | modest loss is painful |
| Payment ledger | very low tolerance for acknowledged-write loss |

Avoid “disk = durable” as the whole answer; hardware, filesystems, replication, and recovery policy matter.

---

# 11. Cost vs Reliability

Reliability improvements are not free.

Examples:

- more replicas
- more regions
- reserved headroom
- duplicate queues
- larger caches
- cross-region traffic
- warm standby
- more observability retention

A senior answer asks:

> Which failure are we buying protection against?

Do not deploy multi-region active-active simply because it sounds highly available.

---

# 12. Simplicity vs Flexibility

Abstractions and generic platforms often improve reuse but create:

- configuration surface
- debugging indirection
- migration cost
- ownership ambiguity

Prefer the simplest design that leaves a credible evolution path.

---

# 13. Push vs Pull

## Push

Useful when freshness matters.

Examples:

- WebSocket events
- notifications
- change streams

Costs:

- connection state
- fan-out
- backpressure
- delivery tracking

## Pull

Useful when consumers control pace.

Examples:

- polling
- queue consumption
- batch processing

Costs:

- freshness delay
- repeated empty reads
- polling load

Hybrid systems are common.

---

# 14. Partitioning vs Replication

These solve different problems.

### Replication

Copies data.

Primary goals:

- availability
- read distribution
- disaster recovery

### Partitioning/sharding

Splits data.

Primary goals:

- capacity
- write distribution
- storage growth

A system often needs both.

---

# 15. Build vs Buy

Evaluate:

- strategic differentiation
- team expertise
- operational burden
- compliance
- vendor lock-in
- data portability
- SLA
- pricing at projected scale
- exit strategy

A managed service can be the more scalable engineering decision if it removes undifferentiated operational work.

---

# Decision Matrix Example — URL Shortener

Suppose:

```text
100M new URLs/day
redirect path is latency-sensitive
reads greatly exceed writes
short-code lookup is by primary key
```

Do not jump directly to “NoSQL + Redis.”

Evaluate:

| Concern | Question |
|---|---|
| Primary key lookup | Can a relational KV/index lookup already meet the load? |
| Write scale | Is a single primary actually a bottleneck? |
| Partitioning | What key distributes writes evenly? |
| Cache | What percentage of redirects are hot/repeated? |
| Durability | Can an acknowledged URL mapping ever disappear? |
| Availability | What happens when the DB is unavailable? |

A reasonable early design might be:

```text
Application
    ↓
Relational database with indexed short code
```

and evolve to:

```text
Application
    ↓
Cache
    ↓
Partitioned/replicated mapping store
```

when measurements justify it.

The interview signal is the evolution reasoning, not the database brand.

---

# Strong Interview Language

Weak:

> “I’ll use Cassandra because it scales.”

Better:

> “The workload is write-heavy and partition-local by user ID. I do not require cross-partition transactions, so a partitioned wide-column model is plausible. The downside is query flexibility and operational complexity, so I would keep access patterns explicit and validate the partition distribution before committing.”

Weak:

> “Use Redis to make it fast.”

Better:

> “The product-read endpoint has high temporal locality and the database read is expensive, so cache-aside should reduce p95 latency and DB load. I’ll use bounded TTL plus invalidation, protect cache misses from stampede, and make sure Redis failure cannot overload the database.”

Weak:

> “Microservices scale better.”

Better:

> “I would keep a modular monolith until independent deployment or ownership becomes a real constraint. Splitting now would introduce distributed transactions and operational overhead without a demonstrated scaling benefit.”

---

# Common Interview Mistakes

- Treating technologies as architecture goals
- Saying “scalable” without naming the bottleneck
- Optimizing before estimating the workload
- Ignoring operational burden
- Ignoring failure modes
- Claiming horizontal scaling is unlimited
- Treating SQL/NoSQL as consistency/scalability opposites
- Using microservices as a default
- Adding queues without delivery semantics
- Adding caches without invalidation
- Choosing multi-region without defining write ownership

---

# Trade-off Checklist

For each major box in the architecture, ask:

```text
Why is it here?
What metric does it improve?
What state does it own?
What happens if it fails?
What new complexity does it introduce?
Can it be removed in the first version?
At what scale would the design change?
```

---

# Key Takeaways

1. Architecture decisions are conditional on requirements.
2. Avoid false binary comparisons.
3. Explain the downside of your own choice.
4. Name the failure mode and mitigation.
5. Use metrics or scale thresholds to justify evolution.
6. Operational simplicity is a first-class quality attribute.
7. A strong interview answer optimizes for the system's constraints, not for architectural fashion.
