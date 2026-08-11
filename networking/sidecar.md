# Sidecar Pattern

A sidecar is a supporting process/container deployed alongside the primary application instance and sharing its deployment lifecycle and, commonly, local network/storage context.

This is primarily an **architecture/deployment pattern**, not a networking primitive. It remains in `/networking` here only to preserve the current repository structure.

## Interview TL;DR

1. Use a sidecar when support functionality benefits from per-instance locality and lifecycle coupling.
2. Common uses: proxying, telemetry collection, secret/config refresh, data synchronization.
3. Sidecars create a per-instance resource tax: CPU, memory, connections, startup time, logs, and failure modes.
4. A sidecar failure can make the main application effectively unavailable even when the app process is healthy.
5. Startup and shutdown ordering matter.
6. Network-proxy sidecars add latency and another retry/timeout layer.
7. Keep domain/business logic out of sidecars.
8. Compare with node-level agents, shared services, libraries, or newer service-mesh architectures before choosing a per-pod sidecar.
9. In Kubernetes, native sidecar containers are now a stable platform concept; distinguish that mechanism from the broader pattern.
10. Move this chapter to a future `architecture-patterns/` section when restructuring.

## Mental Model

```text
Pod / deployment unit
├── Application
└── Sidecar
```

Possible local communication:

```text
Application → localhost → Proxy Sidecar → Remote Service
```

## Good Use Cases

### Local proxy

- traffic policy;
- mTLS;
- telemetry;
- protocol mediation.

### Secret/config agent

- fetch/refresh credentials;
- write to shared volume/local endpoint.

### Local data synchronization

- copy/upload files;
- proxy to remote state.

### Telemetry collector

Useful when per-pod collection is required, though node-level/daemon approaches may be more efficient for some telemetry.

## Resource Tax

If every app pod gets a sidecar:

```text
1,000 app pods
+ 1,000 sidecars
```

Budget:

- CPU;
- RAM;
- file descriptors;
- network connections;
- image pulls;
- metrics/log volume.

The pattern can materially change cluster cost.

## Failure Coupling

Examples:

```text
sidecar proxy broken
→ app cannot reach dependencies

secret agent broken
→ credentials expire

logging sidecar blocks shared volume
→ app disk fills
```

Define whether sidecar failure should:

- make app unready;
- degrade a feature;
- restart the pod;
- alert only.

## Startup Ordering

Main app may need the sidecar ready before accepting traffic.

Likewise, shutdown may require the sidecar to remain alive while the app drains/flushes.

Do not rely on accidental container startup timing.

## Proxy Retry Layer

A network sidecar can add:

```text
application retry
× sidecar retry
× upstream retry
```

Coordinate deadlines and retry budgets.

## Alternatives

### Library

Pros: no extra process/hop.

Cons: language coupling and duplicated rollout.

### Shared service

Pros: fewer resources.

Cons: remote dependency and network latency.

### Node-level agent

Pros: one per node instead of per pod.

Cons: weaker per-workload isolation/locality.

### Sidecar

Pros: local, language-independent, per-workload lifecycle.

Cons: resource and operational overhead.

## Kubernetes Note

Kubernetes provides native sidecar-container lifecycle semantics. That implementation detail is useful, but the broader pattern also exists outside Kubernetes.

## Common Mistakes

- putting business logic into sidecar;
- ignoring sidecar CPU/memory at fleet scale;
- app readiness independent of a mandatory proxy sidecar;
- duplicate retries/timeouts across app and proxy;
- using sidecar where a node agent/shared service is cheaper;
- treating sidecar as a networking concept only.

## 2-Minute Interview Answer

> “I use a sidecar when support functionality needs to be local and versioned with each app instance, such as a service proxy or credential agent. I account for the fleet-wide resource tax and failure coupling, coordinate startup/draining and retry budgets, and keep domain logic in the application. I compare it with a library, node agent, or shared service before paying a sidecar per replica.”

## References

- [Kubernetes — Sidecar Containers](https://kubernetes.io/docs/concepts/workloads/pods/sidecar-containers/)
