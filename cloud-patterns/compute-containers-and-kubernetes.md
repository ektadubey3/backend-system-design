# Compute, Containers, and Kubernetes

## TL;DR

Choose execution by workload shape and operational ownership. Virtual machines provide flexible hosts, containers package a process and dependencies, serverless functions/jobs trade control for managed scaling, and Kubernetes offers a declarative reconciliation platform for container workloads. Kubernetes does not supply application durability, exactly-once processing, or correct service boundaries.

## Execution choices

| Model | Good fit | Costs/limits |
|---|---|---|
| Virtual machine | custom OS/runtime, legacy or specialized host | patching, placement, slower scaling |
| Managed container service | long-running stateless services with low platform burden | provider model and fewer customization points |
| Serverless function | event-driven bursty short work | cold start, duration/concurrency/quota, platform coupling |
| Batch service | queued finite jobs, compute fleets | startup and data movement; retry/idempotency |
| Kubernetes | many container workloads needing shared scheduling/policy/platform APIs | control-plane/ecosystem/on-call complexity |

Do not select Kubernetes because the system is “large.” Select it when workload diversity, portability requirements, and platform-team capability justify owning its abstractions.

## Kubernetes mental model

Kubernetes stores desired state in an API. Controllers reconcile actual state toward it.

- **Pod:** smallest scheduled unit; one or more tightly coupled containers sharing network/storage namespaces.
- **Deployment:** manages stateless replicated Pods through ReplicaSets and rollout.
- **StatefulSet:** stable identity and ordered lifecycle for stateful Pods; does not make the application data safe.
- **Job/CronJob:** finite or scheduled work; handlers must tolerate repeat execution.
- **Service:** stable virtual endpoint selecting ready Pods.
- **Ingress/Gateway:** routes external/application traffic according to installed controller behavior.
- **ConfigMap/Secret:** configuration objects mounted or exposed to Pods; require access and rollout policy.
- **Namespace:** naming/policy scope, not a hard tenant security boundary by itself.

## Scheduling and resources

Resource requests inform placement and reserve capacity; limits constrain use according to resource/runtime behavior. Bad requests cause overcommit, unschedulable Pods, or poor autoscaling. Use node pools and affinity/topology spread for hardware/failure-domain needs. Taints/tolerations allow selected workloads onto specialized nodes but do not guarantee isolation alone.

Containers are disposable; durable data belongs in a state system designed for replication, backup, and fencing. Persistent volumes attach storage but do not define database consistency or multi-zone recovery.

## Platform boundaries

A platform should standardize deployment, identity, network policy, telemetry, secrets delivery, safe defaults, and golden paths while leaving domain behavior to services. Admission policies can reject unsafe configurations. Keep extension/controllers limited: each is privileged control-plane software with its own version and failure modes.

## Failure modes

- One Pod per service is mistaken for high availability.
- StatefulSet ordinal identity is mistaken for consensus/replication.
- Resource requests omitted, so scheduler/autoscaler decisions are meaningless.
- Job retry repeats a non-idempotent external effect.
- Service mesh/sidecar added fleet-wide without resource and failure budget.
- Too many controllers and custom resources create an unmaintainable control plane.

## Interview prompts

- Why does this workload need Kubernetes instead of managed containers/jobs?
- Which state survives Pod/node/zone loss and how?
- How do resource requests reflect measured usage and failover headroom?
- What application guarantees remain outside the platform?

## Two-minute answer

Choose the simplest execution service matching workload duration, scaling, customization, isolation, and team operations. Kubernetes is a declarative controller platform: Deployments reconcile stateless Pods, Services route to ready endpoints, Jobs may repeat, and StatefulSets provide identity—not data correctness. Set measured requests/limits, spread replicas across failure domains, use workload identity and policy, and put durable state in an explicitly replicated/restorable system. Own platform controllers and sidecars like production dependencies.

## References

- [Kubernetes — Concepts](https://kubernetes.io/docs/concepts/)
- [Kubernetes — Workload management](https://kubernetes.io/docs/concepts/workloads/controllers/)

