# Autoscaling, Capacity, and Deployments

## TL;DR

Autoscaling is delayed feedback control, not infinite capacity. Scale from a signal causally related to work, bound it by downstream limits and quotas, retain failure headroom, and combine it with admission control. Deploy gradually with readiness, compatibility, health gates, and a data rollback/forward plan.

## Capacity model

Load-test sustainable useful throughput and the saturation knee. Estimate arrival rate, service time, in-flight concurrency, queue age, memory per request, dependency capacity, and one-zone/instance loss.

```text
required replicas ~= peak work rate / tested sustainable work per replica
then add rollout + failure + uncertainty headroom
```

CPU can be a useful proxy for compute-bound stateless work. It is weak when bottlenecks are queue age, connections, memory, I/O, provider quota, or per-key serialization.

## Scaling layers

- **Workload horizontal scaling:** add replicas based on utilization or custom/work metrics.
- **Workload vertical scaling:** change requests/resources; may require restart or platform support.
- **Node scaling:** add/remove cluster capacity for pending demand.
- **Data/partition scaling:** split storage or broker ownership; usually slower and stateful.
- **Event-driven scaling:** map backlog/arrival to consumers while respecting partition and sink ceilings.

These loops have different delays. A workload scaler cannot place Pods until nodes exist; scaling consumers beyond partitions yields no progress; scaling frontends can overload a fixed database.

## Stable feedback control

Choose target, measurement window, cooldown/stabilization, min/max, and startup time. Rapid scale-down can evict warm caches and cause oscillation. Keep a minimum for latency and sudden traffic. Predictable scheduled events may justify pre-scaling.

For a queue, scale from arrival/completion rate and oldest age, not depth alone. A huge backlog with an exhausted downstream should trigger backpressure, not unlimited consumers.

## Deployment strategies

| Strategy | Benefit | Cost |
|---|---|---|
| rolling | resource-efficient gradual replacement | old/new coexist; rollback depends on compatibility |
| canary | small traffic cohort validates behavior | routing and representative cohort complexity |
| blue/green | fast traffic switch and rollback | double capacity; data changes still shared |
| shadow | compare new reads/decisions without serving | duplicated load; effectful operations must be suppressed |

Use health gates based on SLO/error/saturation and domain correctness, not Pod-running count. Correlate version/config with telemetry. Stop or roll back automatically only for well-understood thresholds; preserve operator control.

## Database and contract compatibility

Use expand-migrate-contract: add compatible schema/API/event fields, deploy code that handles both, backfill under limits, switch ownership/read path, then remove after old versions and retained messages expire. A binary rollback may be impossible after destructive data change; plan roll-forward and restore separately.

## Failure modes

- HPA sees low CPU while requests wait on an exhausted DB pool.
- New Pods start slowly, scaler adds more, and all stampede dependencies/cache.
- Scale-down terminates active workers without checkpoint/idempotency.
- Canary receives only low-risk traffic and misses the hot tenant/path.
- Deployment plus zone loss exceeds surge capacity and stalls.
- Old consumer encounters a newly emitted incompatible event during rollback.

## Interview prompts

- Which scaling signal predicts user latency and has what delay?
- What fixed downstream ceiling limits replicas?
- Is there capacity for canary/surge during a zone loss?
- Can schema/events support old and new versions simultaneously?

## Two-minute answer

Benchmark sustainable per-replica work at the saturation knee and provision peak plus failure/rollout margin. Autoscale on a causal bounded signal—CPU for compute, concurrency/queue age/work rate for others—with stable windows, min/max, pre-scaling, and downstream quotas. Keep admission control because scaling is delayed. Roll out through compatible versions and a representative canary, gate on SLO/saturation/domain metrics, drain safely, and use expand-migrate-contract so rollback or roll-forward does not corrupt shared data.

## References

- [Kubernetes — Autoscaling workloads](https://kubernetes.io/docs/concepts/workloads/autoscaling/)

