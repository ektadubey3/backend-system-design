# System Design Decision Frameworks

Use the same structure for important decisions:

```text
Requirement → Options → Decision → Benefit → Cost → Mitigation → Reconsideration trigger
```

## SQL vs NoSQL

Do not reduce this to “ACID vs scalability.” Evaluate:

- data model;
- query patterns;
- transaction boundaries;
- consistency;
- partition key;
- secondary indexes;
- write amplification;
- operational expertise;
- horizontal scaling model;
- cost.

## Cache or No Cache

Cache when repeated reads, expensive computation, or origin protection justify stale-data and invalidation complexity.

Decide:

- cache key;
- TTL;
- invalidation path;
- stampede protection;
- hot-key strategy;
- memory/eviction policy;
- acceptable stale window.

## Queue or Synchronous Call

Prefer asynchronous processing when work can complete later and decoupling, smoothing bursts, or retries matter.

Costs:

- eventual consistency;
- ordering complexity;
- duplicate delivery;
- harder debugging;
- backlog management.

## Replication vs Sharding

- Replication primarily improves availability and can improve read capacity.
- Sharding primarily increases dataset/write capacity.

They solve different problems and are often combined.

## Strong vs Eventual Consistency

Classify consistency **per operation**, not per company or entire product.

Example:

```text
ledger write       → strong correctness requirement
account dashboard  → may read a projection
analytics          → eventual consistency
notification       → asynchronous
```

## Monolith vs Microservices

Evaluate organizational and operational constraints, not trendiness:

- team boundaries;
- deployment independence;
- data ownership;
- failure isolation;
- observability maturity;
- network overhead;
- transaction complexity.

A modular monolith is often the lower-risk starting point.
