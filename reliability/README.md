# Reliability Engineering

Reliability is the probability that a system delivers the intended result under stated conditions. The goal is controlled behavior when dependencies slow, capacity saturates, instances fail, or regions disappear—not merely recovery after a crash.

## Learning path

1. [Deadlines, timeouts, and retries](deadlines-timeouts-and-retries.md) — bound work and avoid retry amplification.
2. [Overload, backpressure, and load shedding](overload-backpressure-and-load-shedding.md) — protect useful throughput at saturation.
3. [Circuit breakers, bulkheads, and degradation](circuit-breakers-bulkheads-and-degradation.md) — contain dependency and resource failures.
4. [SLIs, SLOs, and error budgets](slis-slos-and-error-budgets.md) — turn reliability into measurable product policy.
5. [Disaster recovery](disaster-recovery.md) — RPO, RTO, backup, failover, and restore evidence.
6. [Reliability design framework](reliability-design-framework.md) — interview-ready sequence and failure review.

## Existing foundations

This section deepens rather than replaces [Fault tolerance](../fundamentals/fault-tolerance.md), [Availability](../fundamentals/availability.md), [Reliability](../fundamentals/reliability.md), and [Latency versus throughput](../fundamentals/latency-vs-throughput.md). Use [Messaging retries, DLQs, and backpressure](../messaging/retries-dead-letters-and-backpressure.md) for asynchronous failure policy.

## Reliability vocabulary

- **Fault:** underlying defect or failed component.
- **Error:** incorrect internal state caused by a fault.
- **Failure:** user-visible service behavior violates its contract.
- **Resilience:** ability to absorb and recover from faults while controlling impact.
- **Redundancy:** extra capacity or copies; useful only across independent failure domains.
- **Graceful degradation:** preserving a smaller, explicitly correct product during stress.

## Core review questions

- What is the user's critical journey and its measurable success condition?
- Which resource saturates first and how is admission bounded?
- Where are deadlines propagated and retries budgeted?
- Which dependencies and tenants share a failure pool?
- What degraded response is honest and useful?
- What evidence proves RPO, RTO, and restore capability?

