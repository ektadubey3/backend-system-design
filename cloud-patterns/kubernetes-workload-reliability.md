# Kubernetes Workload Reliability

## TL;DR

Readiness controls traffic, liveness requests restart, and startup delays the other probes while initialization completes. Probes must be cheap and express distinct facts. Combine them with graceful termination, disruption budgets, topology spread, resource sizing, bounded dependency behavior, and an application-level recovery model.

## Probe semantics

- **Startup probe:** has initialization completed enough to begin ordinary probing?
- **Readiness probe:** should this endpoint receive new traffic now?
- **Liveness probe:** is restart likely to repair a stuck process?

Do not make liveness depend on a remote database; an outage could restart every otherwise healthy Pod and worsen recovery. Readiness may reflect inability to serve, but overly broad dependency checks can remove all endpoints. Serve probe handlers from reserved capacity so overload does not look like process death.

Tune thresholds from measured startup, pause, and recovery behavior. A startup probe prevents slow initialization from being killed by liveness.

## Graceful lifecycle

On termination:

1. become unready and stop accepting new work;
2. allow routing propagation;
3. drain requests/messages within the termination deadline;
4. stop renewing ownership/leases;
5. checkpoint or safely abandon idempotent work;
6. exit before forced kill.

Applications still need deadlines and client retry/idempotency because connections can drop during any rollout.

## Availability and disruption

Run replicas across zones/nodes using topology spread or anti-affinity. A PodDisruptionBudget limits simultaneous voluntary disruption, but it cannot stop node crashes and does not create spare capacity. Ensure rollout and one-zone-loss headroom, and test cluster/node autoscaler interaction.

Deployments need `maxUnavailable`/`maxSurge` compatible with capacity and SLO. A rollout with no surge cannot proceed if the cluster has no spare node resources. DaemonSets and StatefulSets have different disruption constraints.

## Configuration and secrets

Use immutable/versioned configuration where possible and include config version in telemetry. Mounted/configured values do not necessarily restart applications; define reload or rollout behavior. Secrets need storage encryption, narrow RBAC/workload access, external rotation, and redaction. Avoid secrets in environment dumps and command lines.

## Networking and dependencies

Service discovery and load balancing do not eliminate stale connections. Set connection age/drain, timeouts, retry budgets, and pool limits. NetworkPolicy is additive isolation only when the network implementation enforces it; start deny-by-default and allow required flows. Protect DNS capacity and understand node/zone locality.

## Stateful workloads

Before running a database on Kubernetes, define replica protocol, quorum placement, persistent-volume failure/attachment, backup/restore, operator upgrade behavior, fencing, and who owns incidents. An operator automates a protocol; inspect its assumptions and recovery paths.

## Failure modes

- Liveness checks database and crash-loops the fleet during DB outage.
- Readiness turns true before caches/connections/config are safe for peak traffic.
- Termination deadline is shorter than request/job deadline.
- All replicas scheduled in one zone or on one node pool.
- PDB blocks maintenance while there is no spare capacity to reschedule.
- Secret rotates, but long-lived connection/client never reloads it.

## Interview prompts

- What exactly can restart repair that readiness cannot?
- Can the rollout proceed during one-zone loss?
- How does an in-flight message remain safe on termination?
- Which platform component enforces the declared network policy?

## Two-minute answer

Use startup to protect initialization, readiness to control new traffic, and liveness only for locally detectable stuck states that restart can repair. On termination become unready, drain within a bounded deadline, stop leases, and rely on idempotent retry for interrupted work. Spread replicas and capacity across zones, align PDB and rollout surge with real headroom, version configuration, rotate secrets safely, and bound connections/dependencies. Stateful workloads still need an explicit consensus, fencing, storage, backup, and operator recovery design.

## References

- [Kubernetes — Liveness, readiness, and startup probes](https://kubernetes.io/docs/concepts/workloads/pods/probes/)

