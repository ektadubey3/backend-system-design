# Architecture Decision Framework

## TL;DR

Choose patterns from forces, not fashion. Record the current context, hard constraints, options, failure/operations cost, measurable fitness functions, migration, and reversal. Prefer the simplest architecture that meets present requirements while preserving a credible evolution seam.

## Decision sequence

### 1. State the pressure

Examples: a specific team cannot deploy independently; one tenant's workload harms others; read shape cannot meet its SLO; audit requires temporal reconstruction; legacy release risk blocks product changes. Avoid “we need microservices/events/Kubernetes” as the problem statement.

### 2. Identify invariants and owners

Name domain authority, transaction boundary, schema owner, and critical journey. Patterns that split a hard invariant add coordination; patterns that keep it local simplify correctness.

### 3. Compare at least two viable options

Include “improve the current shape.” Compare latency, availability coupling, consistency, scalability, security/isolation, team ownership, testability, data migration, observability, cost, and on-call load.

### 4. Expose new state and failures

Every queue, gateway, sidecar, read model, workflow engine, routing directory, and control plane becomes production state. Describe backlog, version skew, recovery, and what happens when it is unavailable.

### 5. Define fitness functions

Examples:

- no cross-module database access in build checks;
- p99 critical path stays below target under one-zone loss;
- one cell failure affects at most 2% of tenants;
- projection freshness remains under 30 seconds;
- service can deploy independently with backward-compatible contracts;
- restore and rollback complete inside measured objectives.

### 6. Plan incremental migration

Create a seam, copy/shadow, compare, route a cohort, advance ownership, retain rollback, and remove old state after evidence. Avoid flag-day rewrites and unbounded dual writes.

## Pattern selection guide

| Pressure | Consider | First challenge |
|---|---|---|
| codebase coupling | modular monolith | are module APIs/data enforceable? |
| independent operational need | microservice | is authority cohesive and team-operated? |
| tenant blast radius | cells | are dependencies truly isolated? |
| read/write shape divergence | CQRS | can projection lag/rebuild be owned? |
| temporal audit as core domain | event sourcing | can events remain semantically replayable? |
| gradual legacy replacement | strangler | where is the routing/data authority seam? |
| multi-service business workflow | saga/orchestrator | what are compensation and unknown states? |

## Two-minute answer template

“The pressure is [measurable constraint], while invariant [X] is owned by [Y]. Options are [current improvement, pattern A, pattern B]. I choose [option] because it meets [fitness functions] with acceptable costs [new state/failures/operations]. Data and release ownership become [boundary]. I will migrate through [seam, shadow, cohort, ownership cutover], monitor [evidence], and roll back by [mechanism]. I would revisit the choice when [trigger] occurs.”

## Follow-up questions

- What team will operate the new state at 3 a.m.?
- Could a module boundary solve this without a network boundary?
- Which user journey becomes worse under the chosen pattern?
- What assumption makes the choice reversible—or irreversible?
- What evidence would cause you to remove the pattern?

## Further study

- [Trade-off analysis](../fundamentals/trade-off-analysis.md)
- [Interview scoring rubric](../interview/scoring-rubric.md)
- [Distributed-systems design framework](../distributed-systems/distributed-systems-design-framework.md)

