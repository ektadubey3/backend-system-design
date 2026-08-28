# Cloud Design Framework

## TL;DR

Translate requirements into primitives in this order: workload, state, network, identity, failure domains, capacity/deployment, observability/recovery, then provider selection. This keeps service names from replacing architecture reasoning.

## Framework

### 1. Classify workloads

For each component state duration, concurrency, latency, CPU/memory/accelerator, network, startup, scheduling, and isolation needs. Choose VM, managed container, function, batch, or Kubernetes from this shape and team capability.

### 2. Place authoritative and derived state

Define consistency, partition key, replication, durability acknowledgement, backup/restore, and ownership. Keep object/blob payloads, transactional metadata, queues, caches, and indexes in primitives matching their access and recovery models.

### 3. Design connectivity

Map edge, load balancing, DNS/service discovery, private/public paths, egress/NAT/proxy, TLS identity, connection pools, and quotas. Avoid unbounded fan-out and cross-region critical calls.

### 4. Establish identity and policy

Use scoped workload identities and separate environment/cell accounts. Define secrets/key lifecycle, network and resource policy, audit, administrative access, and recovery authority.

### 5. Map failure domains

Place replicas across zones and size survivors. Separate serving data plane from management control plane. State region-loss behavior, data ownership epoch, RPO/RTO, and hidden global dependencies.

### 6. Bound capacity and change

Load-test bottlenecks, reserve failure/rollout headroom, autoscale on causal signals within downstream quotas, and retain admission control. Deploy through compatible canary/rolling/blue-green stages with expand-migrate-contract data changes.

### 7. Operate and recover

Define user SLOs, saturation and quota signals, cost attribution, runbooks, backup/restore and regional drills, provider maintenance, and incident ownership.

### 8. Select managed services and portability

Compare semantic fit, operational transfer, limits, regional availability, recovery/export, ecosystem skill, and cost. Portability means documented semantic dependencies and a tested data/traffic exit—not hiding every API behind a lowest-common-denominator wrapper.

## Decision table

| Area | Required statement |
|---|---|
| Compute | workload shape and why this execution model fits |
| Data | authority, consistency, partitioning, acknowledgement, restore |
| Network | ingress/egress/service identity and bounded connections |
| Isolation | account/cell/tenant boundaries and credentials |
| Resilience | zone/region/control-plane behavior and survivor capacity |
| Scale | bottleneck, signal, delay, max, quota, admission policy |
| Delivery | compatibility, health gates, drain, rollback/roll-forward |
| Operations | SLO, telemetry, cost, runbook, drill, owner |

## Two-minute answer template

“The workloads are [interactive service/job/stream/stateful], so I use [execution primitive] because [shape/operations]. [Store] is authoritative with [consistency/partition/replication], while [cache/index/object] is derived or immutable. Traffic enters through [edge] and services authenticate with scoped workload identity. Replicas and capacity span [domains], and the data plane can continue through [control-plane failure]. Autoscaling uses [signal] within [downstream/quota], with admission control and zone-loss headroom. Releases use [strategy] plus compatible data changes. RPO/RTO are [objectives] proven by [drill], and provider-specific semantics/exit are documented.”

## Follow-up questions

- Why Kubernetes rather than a managed container or job service?
- Which managed-service guarantee is assumed but unverified?
- What happens when autoscaling and the control plane are unavailable together?
- Can the destination region handle traffic and quotas immediately?
- How is data exported and validated if the selected service must be replaced?

## Further study

- [Networking](../networking/README.md)
- [Distributed-systems design framework](../distributed-systems/distributed-systems-design-framework.md)
- [Reliability design framework](../reliability/reliability-design-framework.md)
- [Security design framework](../security/security-design-framework.md)

