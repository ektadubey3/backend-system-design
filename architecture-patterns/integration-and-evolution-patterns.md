# Integration and Evolution Patterns

## TL;DR

Gateways and BFFs adapt external clients; sidecars/agents handle uniform local infrastructure concerns; stranglers replace systems incrementally; sagas coordinate multi-service business workflows. Each introduces routing or workflow state that needs ownership, observability, and rollback.

## Gateway and Backend for Frontend

An API gateway centralizes edge routing, authentication enforcement, coarse quotas, protocol termination, and request limits. Keep domain authorization and business orchestration in owning services. A gateway that accumulates product logic becomes a critical distributed monolith.

A Backend for Frontend (BFF) adapts APIs to one client experience, reducing mobile/web chatiness and allowing separate evolution. It should not become a second source of domain truth. Budget fan-out and partial responses; one BFF request can create many downstream calls.

## Sidecar and node agent

A sidecar shares a deployment boundary with an application instance and can provide proxying, credential refresh, log shipping, or adapters without language-specific code. It adds resource overhead, startup/readiness coupling, another version, and a local network hop. A node-level agent is cheaper for host-wide concerns but provides weaker per-workload isolation/customization.

Use the canonical [Sidecar Pattern](../networking/sidecar.md) chapter for mechanics and failure analysis.

## Strangler migration

A facade routes selected capabilities or cohorts from a legacy system to a new implementation:

1. Establish observability and a routing seam.
2. Extract one bounded capability/read path.
3. Backfill or synchronize data with a declared authority.
4. Shadow and compare without effectful double execution.
5. Shift a small cohort and preserve rollback.
6. Advance ownership; stop legacy writes.
7. Remove translation and old code after verification/retention windows.

Dual writes create divergence. Prefer one authority plus CDC/outbox replication, and make cutover a versioned ownership state.

## Saga, orchestration, and choreography

A saga is a sequence of local transactions with persisted progress and compensating business actions. Orchestration uses one process manager to command steps and handle timeouts; choreography has services react to events.

Orchestration makes workflow state and recovery visible but centralizes coordination. Choreography reduces one central component but can hide cycles, ownership, and end-to-end state. Use choreography for independent reactions; use an orchestrator when the product needs one queryable multi-step outcome, deadlines, and explicit compensation.

Compensation is not rollback. Refunding a payment or releasing a seat can fail and may not erase external consequences. Make compensations idempotent, retryable, and operator-visible.

## Anti-corruption layer

Translate legacy or partner concepts into the domain's own model at the boundary. This prevents external naming, error semantics, and unstable schemas from spreading. The layer adds mapping/version logic and needs contract tests and metrics.

## Failure modes

- Gateway is the only place that checks authorization.
- Sidecar readiness prevents application recovery or doubles resource needs unexpectedly.
- Strangler routes reads to new data while writes still mutate legacy with unbounded lag.
- Saga status exists only in messages and cannot be queried/repaired.
- Choreography loop repeatedly triggers the same business action.
- BFF fan-out causes a tail-latency and retry storm.

## Interview prompts

- Which component owns the workflow state and terminal outcome?
- How does traffic roll back during strangler cutover?
- Is this concern truly uniform enough for a sidecar?
- Which compensation cannot restore the original world?

## Two-minute answer

Keep gateways/BFFs at adaptation and policy enforcement boundaries, leaving domain authority in services. Use a sidecar only for uniform independently operated local infrastructure concerns and include its lifecycle/resource failure. Migrate with a strangler seam, one data authority, shadow comparison, cohort routing, and versioned ownership cutover. For cross-service workflows, persist a saga: orchestrate when one outcome/deadline/compensation needs visibility, and use choreography for independent reactions. Every compensation and routing step is idempotent, observable, and reversible where possible.

