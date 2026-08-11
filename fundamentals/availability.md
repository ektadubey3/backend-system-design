# Availability

Availability is the ability of a service to successfully serve an eligible operation when it is needed.

“Uptime” is a useful approximation, but modern services should measure availability through a **service-level indicator (SLI)** that reflects successful user-facing work.

## Interview TL;DR

1. Define availability per critical operation, not only per host or process.
2. Prefer an SLI such as `successful eligible requests / eligible requests`.
3. Separate **SLI**, **SLO**, and **SLA**.
4. Redundancy improves availability only when replicas do not share the same failure mode.
5. Failover needs detection, routing, state safety, and recovery—not only a spare server.
6. Overload is an availability failure mode.
7. Graceful degradation may preserve critical operations while dropping optional work.
8. Multi-region designs add data-consistency and operational complexity.
9. Replication is not backup.
10. Availability and reliability overlap but are not interchangeable.

## SLI, SLO, SLA

### SLI

What you measure.

```text
successful checkout requests
----------------------------
eligible checkout requests
```

### SLO

The engineering target.

```text
99.95% successful checkout operations over 30 days
```

### SLA

A customer/business commitment that may include contractual consequences.

## “Nines”

Approximate downtime budget if availability were measured purely by time:

| Availability | Approx. downtime/year |
|---|---:|
| 99% | 3.65 days |
| 99.9% | 8.76 hours |
| 99.99% | 52.6 minutes |
| 99.999% | 5.26 minutes |

Real service SLOs are often request-based.

## Failure Domains

Think in layers:

```text
process
host
rack
availability zone
region
provider/control plane
dependency
```

Redundancy must span the failure domains the requirement cares about.

## Dependency Availability

A synchronous path:

```text
API → Service A → Service B → Database
```

can be less available than any individual component because all required dependencies must succeed.

Reduce unnecessary synchronous dependencies.

## Failover

A complete failover includes:

```text
detect failure
      ↓
establish healthy authority
      ↓
route traffic
      ↓
prevent split brain
      ↓
recover capacity
      ↓
rejoin safely
```

A secondary merely existing is not enough.

## Active-Passive

Benefits:

- simpler write ownership;
- simpler conflict model.

Costs:

- unused capacity;
- failover time;
- cold-cache/warm-up risk.

## Active-Active

Benefits:

- serving capacity is already active;
- potentially faster traffic failover.

Costs:

- data ownership/conflict complexity;
- larger operational surface;
- harder testing.

Active-active application servers do not imply active-active database writes.

## Overload

A healthy process can still be unavailable if queues grow until requests time out.

Use:

- bounded queues;
- admission control;
- load shedding;
- deadlines;
- concurrency limits;
- priority classes.

## Graceful Degradation

```text
Preserve:
- login
- cart
- checkout

Degrade:
- recommendations
- analytics
- personalization
```

## Health Checks

Separate:

- **liveness**: should the process be restarted?
- **readiness**: should it receive traffic?

Avoid making readiness depend on every optional dependency.

## Multi-Region

Use only when the requirement justifies:

- extra cost;
- replication;
- consistency decisions;
- traffic steering;
- data residency;
- failover testing.

State write ownership explicitly.

## Availability vs Durability

A service can be available while losing acknowledged data.

A service can be unavailable while preserving every committed write.

Treat the properties separately.

## Common Mistakes

- measuring “process up” instead of successful operations;
- calling replicas backups;
- assuming redundant components fail independently;
- health checks that trigger cascades;
- retrying overload;
- multi-region without a data model.

## Interview Answer Template

> “I’ll define availability for the critical operation as a request-based SLI and give optional features lower criticality. I’ll isolate failure domains and protect overload with bounded queues and load shedding. Stateful failover needs explicit authority and fencing. I only add multi-region if the regional-failure requirement justifies the consistency and operational cost.”

## References

- [Google SRE — Service Level Objectives](https://sre.google/sre-book/service-level-objectives/)
- [Google SRE — Handling Overload](https://sre.google/sre-book/handling-overload/)
