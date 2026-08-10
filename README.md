# Backend System Design

> An interview-focused backend system design handbook for experienced engineers: fundamentals, networking, databases, distributed systems, production trade-offs, and complete design exercises.

This repository is a living knowledge base for **Senior Backend Engineer and System Design interviews**. The goal is not to memorize products or architecture diagrams. The goal is to learn how to reason from requirements to APIs, data models, scaling decisions, failure handling, consistency guarantees, observability, and trade-offs.

## Start Here

If you are preparing for interviews, use this order:

1. Review the [System Design Fundamentals](fundamentals/).
2. Learn the [45-minute interview framework](interview/45-minute-framework.md).
3. Practice [capacity estimation](interview/capacity-estimation.md) and [decision frameworks](interview/decision-frameworks.md).
4. Attempt the [URL Shortener](case-studies/01-url-shortener.md) and [Rate Limiter](case-studies/02-rate-limiter.md) without reading the solutions first.
5. Score yourself using the [interview rubric](interview/scoring-rubric.md).
6. Repeat with the [question bank](interview/question-bank.md).

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

The README distinguishes between content that exists today and content that is planned. Planned sections are not represented as completed material.

| Area | Status | Entry point |
|---|---|---|
| Fundamentals | ✅ Available | [Browse](fundamentals/) |
| Networking | ✅ Available | [Browse](networking/) |
| Databases | 🚧 In progress | [Browse](databases/) |
| Interview preparation | ✅ Available | [Browse](interview/) |
| Case studies | 🚧 In progress | [Browse](case-studies/) |
| Caching | 📋 Planned | [Roadmap](ROADMAP.md) |
| Messaging | 📋 Planned | [Roadmap](ROADMAP.md) |
| Distributed systems | 📋 Planned | [Roadmap](ROADMAP.md) |
| Reliability engineering | 📋 Planned | [Roadmap](ROADMAP.md) |
| Security | 📋 Planned | [Roadmap](ROADMAP.md) |
| Observability | 📋 Planned | [Roadmap](ROADMAP.md) |
| Architecture patterns | 📋 Planned | [Roadmap](ROADMAP.md) |
| Cloud & infrastructure | 📋 Planned | [Roadmap](ROADMAP.md) |

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

Start with a small number of complete designs instead of many shallow examples.

- [URL Shortener](case-studies/01-url-shortener.md)
- [Distributed Rate Limiter](case-studies/02-rate-limiter.md)

Future case studies are prioritized in [ROADMAP.md](ROADMAP.md).

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
