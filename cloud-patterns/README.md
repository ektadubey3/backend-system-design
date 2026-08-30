# Cloud Architecture Patterns

Cloud platforms provide programmable compute, networking, storage, identity, and managed control planes. Good designs reason from workload and failure semantics, then map them to provider primitives. A managed service transfers some operations; it does not transfer data modeling, quotas, security, observability, or recovery ownership.

## Learning path

1. [Cloud foundations and failure domains](cloud-foundations-and-failure-domains.md) — regions, zones, control/data planes, managed services, and shared responsibility.
2. [Compute, containers, and Kubernetes](compute-containers-and-kubernetes.md) — choosing execution models and understanding core Kubernetes primitives.
3. [Kubernetes workload reliability](kubernetes-workload-reliability.md) — probes, resources, disruption, networking, configuration, and state.
4. [Autoscaling, capacity, and deployments](autoscaling-capacity-and-deployments.md) — scaling signals, headroom, rollout, and rollback.
5. [Multi-region cloud architecture](multi-region-cloud-architecture.md) — regional independence, routing, data ownership, and evacuation.
6. [Cloud design framework](cloud-design-framework.md) — interview sequence and portability decisions.

## Boundaries with existing curricula

- [Networking](../networking/README.md) covers DNS, proxies, load balancing, service discovery, TLS, and connection management.
- [Distributed systems](../distributed-systems/README.md) covers replication, consensus, partitioning, ownership, and multi-region data semantics.
- [Reliability](../reliability/README.md), [Security](../security/README.md), and [Observability](../observability/README.md) define the production requirements that cloud services must satisfy.

## Principle

Use the least operationally expensive primitive that meets the workload's latency, consistency, isolation, portability, recovery, and team-capability requirements. Introduce Kubernetes after understanding those primitives, not as a substitute for them.

