# Architecture Patterns

Patterns are reusable arrangements of responsibility and communication, not solutions to copy blindly. A strong design names the pressure that makes a pattern useful, the failure and operational state it introduces, and the trigger for evolving away from the simpler shape.

## Learning path

1. [Service boundaries and deployment shapes](service-boundaries-and-deployment-shapes.md) — monolith, modular monolith, microservices, cells, and data ownership.
2. [Event-driven architecture, CQRS, and event sourcing](event-driven-cqrs-and-event-sourcing.md) — asynchronous facts, separate models, and logs as authority.
3. [Integration and evolution patterns](integration-and-evolution-patterns.md) — gateway, BFF, sidecar, strangler, saga, choreography, and orchestration.
4. [Architecture decision framework](architecture-decision-framework.md) — forces, fitness functions, and interview answers.

## Existing canonical chapters

- [Sidecar pattern](../networking/sidecar.md) covers sidecar mechanics and networking tradeoffs.
- [SQL transactions](../databases/sql/transactions.md) covers saga, orchestration/choreography, transactional outbox, and 2PC.
- [Trade-off analysis](../fundamentals/trade-off-analysis.md) provides the repository-wide comparison method.

## Pattern discipline

For every pattern document:

- the concrete problem and constraints;
- responsibility and data ownership;
- new state, queues, control planes, and failure modes;
- correctness and operability implications;
- evidence/fitness function that says it helps;
- the simplest migration and rollback path.

