# Backend System Design

> An interview-focused backend system design handbook for experienced engineers: fundamentals, networking, databases, distributed systems, production trade-offs, and complete design exercises.

This repository is a living knowledge base for **Senior Backend Engineer and System Design interviews**. The goal is not to memorize products or architecture diagrams. The goal is to reason from requirements to APIs, data models, consistency boundaries, scaling decisions, failure handling, observability, security, operations, and trade-offs.

## Start Here

If you are preparing for interviews, use this order:

1. Review [System Design Fundamentals](fundamentals/) and the [45-minute interview framework](interview/45-minute-framework.md).
2. Practice [capacity estimation](interview/capacity-estimation.md) and [decision frameworks](interview/decision-frameworks.md).
3. Learn the core backend stack: [Networking](networking/), [Databases](databases/), [Caching](caching/README.md), and [Messaging](messaging/README.md).
4. Add distributed and production reasoning: [Distributed Systems](distributed-systems/README.md), [Reliability](reliability/README.md), [Security](security/README.md), and [Observability](observability/README.md).
5. Connect the choices through [Architecture Patterns](architecture-patterns/README.md) and [Cloud Patterns](cloud-patterns/README.md).
6. Attempt the [case studies](case-studies/) without reading the model answers, then score yourself with the [rubric](interview/scoring-rubric.md).
7. Repeat with the [question bank](interview/question-bank.md) and each curriculum's design framework.

## What Strong System Design Answers Demonstrate

A strong answer connects each architecture choice to a requirement:

```text
Requirements
    ↓
Scale assumptions
    ↓
API + data model
    ↓
High-level architecture
    ↓
Critical flows
    ↓
Scaling + consistency
    ↓
Failure modes
    ↓
Observability + security
    ↓
Trade-offs + evolution
```

The repository therefore emphasizes **why** a design is chosen, what it costs, how it fails, and what would cause it to evolve.

## Curriculum Status

Every available area has a landing page or established section index. The documentation-site tooling remains intentionally deferred in the [roadmap](ROADMAP.md).

| Area | Status | Entry point |
|---|---|---|
| Fundamentals | ✅ Available | [Browse](fundamentals/) |
| Networking | ✅ Available | [Browse](networking/) |
| Databases | ✅ Available | [Browse](databases/) |
| Interview preparation | ✅ Available | [Browse](interview/) |
| Case studies | ✅ Available | [Browse](case-studies/) |
| Caching | ✅ Available | [Browse](caching/README.md) |
| Messaging | ✅ Available | [Browse](messaging/README.md) |
| Distributed systems | ✅ Available | [Browse](distributed-systems/README.md) |
| Reliability engineering | ✅ Available | [Browse](reliability/README.md) |
| Security | ✅ Available | [Browse](security/README.md) |
| Observability | ✅ Available | [Browse](observability/README.md) |
| Architecture patterns | ✅ Available | [Browse](architecture-patterns/README.md) |
| Cloud & infrastructure | ✅ Available | [Browse](cloud-patterns/README.md) |

## Existing Fundamentals

- [Functional vs Non-functional Requirements](fundamentals/function-vs-non-functional-requirements.md)
- [Scalability](fundamentals/scalability.md)
- [Availability](fundamentals/availability.md)
- [Reliability](fundamentals/reliability.md)
- [Fault Tolerance](fundamentals/fault-tolerance.md)
- [Latency vs Throughput](fundamentals/latency-vs-throughput.md)
- [Horizontal vs Vertical Scaling](fundamentals/horizontal-vs-vertical-scaling.md)
- [CAP Theorem](fundamentals/cap-theorem.md)
- [PACELC](fundamentals/pacelc.md)
- [Bottleneck Identification](fundamentals/bottleneck-identification.md)
- [Trade-off Analysis](fundamentals/trade-off-analysis.md)
- [High-Level Design](fundamentals/hld.md)
- [Low-Level Design](fundamentals/lld.md)

## Existing Networking

- [HTTP & HTTPS](networking/http&https.md)
- [HTTP/2 & HTTP/3](networking/http2&http3.md)
- [REST](networking/rest.md)
- [GraphQL](networking/graphql.md)
- [gRPC](networking/grpc.md)
- [DNS](networking/dns.md)
- [CDN](networking/cdn.md)
- [Load Balancers](networking/load-balancer.md)
- [Reverse Proxy](networking/reverse-proxy.md)
- [API Gateways](networking/api-gateways.md)
- [Connection Pooling](networking/connection-pooling.md)
- [Keep-Alive](networking/keep-alive.md)
- [Sidecar](networking/sidecar.md)

## Existing Databases

### SQL

- [PostgreSQL](databases/sql/postgresql.md)
- [MySQL](databases/sql/mysql.md)
- [Indexing](databases/sql/indexing.md)
- [Query Optimization](databases/sql/query-optimization.md)
- [Transactions](databases/sql/transactions.md)
- [ACID](databases/sql/acid.md)
- [Isolation Levels](databases/sql/isolation-levels.md)
- [Locks](databases/sql/locks.md)
- [Joins](databases/sql/joins.md)

### NoSQL

- [MongoDB](databases/nosql/mongodb.md)
- [Redis](databases/nosql/redis.md)

## Interview Preparation

The interview layer is deliberately separate from encyclopedic topic notes.

- [45-Minute Framework](interview/45-minute-framework.md)
- [Requirements Checklist](interview/requirements-checklist.md)
- [Capacity Estimation](interview/capacity-estimation.md)
- [Decision Frameworks](interview/decision-frameworks.md)
- [Question Bank](interview/question-bank.md)
- [Scoring Rubric](interview/scoring-rubric.md)
- [Rapid Review](interview/rapid-review.md)

## Case Studies

Use the [case-study index](case-studies/README.md) for ten complete model designs, from URL shortening and rate limiting through payments, booking, analytics, and video streaming. Each follows the same requirements-to-evolution answer shape.

## Advanced Curriculum

- [Caching](caching/README.md)
- [Messaging Systems](messaging/README.md)
- [Distributed Systems](distributed-systems/README.md)
- [Reliability Engineering](reliability/README.md)
- [Security Architecture](security/README.md)
- [Observability](observability/README.md)
- [Architecture Patterns](architecture-patterns/README.md)
- [Cloud Architecture Patterns](cloud-patterns/README.md)

## Content Standard

Every new technical chapter should answer:

1. What problem does this solve?
2. What is the mental model?
3. How does it work?
4. When should it be used?
5. When should it not be used?
6. How does it scale?
7. How does it fail?
8. What are the important trade-offs?
9. What are the production gotchas?
10. What would an interviewer ask next?

See [CONTRIBUTING.md](CONTRIBUTING.md) for the full chapter template.

## Books

Recommended foundations:

- *Designing Data-Intensive Applications* — Martin Kleppmann
- *Database Internals* — Alex Petrov
- *Building Microservices* — Sam Newman
- *System Design Interview* — Alex Xu
- *Domain-Driven Design* — Eric Evans

## Contributing

Corrections and high-quality additions are welcome. Prefer material that improves architecture reasoning rather than increasing topic count. See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT.
