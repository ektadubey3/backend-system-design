# Load Balancers

A load balancer selects a backend target for incoming network traffic.

The important system-design questions are **what layer it operates at, what signal it uses, how health/draining work, and how it behaves during overload and partial failure**.

## Interview TL;DR

1. L4 balances connections/transport; L7 can route using HTTP/application metadata.
2. “Round robin” is not enough when requests or connections have very different cost.
3. Health checks should control routing without creating cascading removal.
4. Readiness and liveness solve different problems.
5. Long-lived connections can make distribution uneven even when request-rate balancing looks fair.
6. Connection draining is required for safe deploy/scale-down.
7. Slow start/warm-up prevents a cold backend from receiving full load immediately.
8. Retry policy at the balancer can multiply load and duplicate non-idempotent work.
9. Load shedding and priority isolation can preserve critical traffic under overload.
10. A managed load-balancing service may already provide internal redundancy; do not prescribe “deploy two LBs” without knowing the platform.

## L4 vs L7

### Layer 4

Routes using transport/network information such as:

- destination IP/port;
- connection state.

Useful for:

- arbitrary TCP/UDP protocols;
- high-throughput transport balancing.

### Layer 7

Understands application protocol such as HTTP.

Can route on:

- host;
- path;
- header;
- cookie;
- method.

Can also implement:

- redirects;
- auth integration;
- rate limits;
- request limits;
- canary traffic.

## Algorithms

### Round Robin

Good when backend capacity and request cost are similar.

### Weighted Round Robin

Good when backend capacities differ.

### Least Connections

Useful for long-lived/variable-duration connections.

### Least Request / Response-Time-Informed

Useful when active load varies, but metrics can be noisy or lagging.

### Consistent Hashing / Affinity

Useful when a key should tend to reach the same backend/cache shard.

Trade-off: hotspots and uneven load.

## Health Checks

A good readiness check answers:

> Can this instance accept more production traffic?

Avoid expensive health checks that overload dependencies.

Avoid making every optional dependency mandatory for readiness.

## Passive Failure Signals

Balancers may also observe:

- connection failures;
- resets;
- timeouts;
- selected response codes.

Be careful with automatically ejecting backends during a shared downstream outage: every backend can appear unhealthy.

## Connection Draining

Deployment:

```text
mark backend draining
      ↓
stop new traffic
      ↓
finish/timeout existing requests or streams
      ↓
terminate
```

For WebSockets/gRPC streams, draining can take much longer than an ordinary HTTP request.

## Slow Start / Warm-Up

New instances may have:

- cold caches;
- empty DB pools;
- JIT/runtime warm-up;
- lazy model/config load.

Ramp traffic gradually where supported.

## TLS Termination

A load balancer may terminate TLS.

Define whether traffic to backends is:

- plaintext inside a trusted network;
- re-encrypted;
- mTLS.

## Retry Ownership

Bad:

```text
client retries
  × load balancer retries
  × service retries
```

Define one clear retry budget and preserve request deadlines.

Never retry unsafe mutations blindly.

## Overload

The load balancer can enforce:

- connection limits;
- request rate;
- queue limits;
- priority routing;
- load shedding.

A system is healthier when excess work is rejected early rather than queued until timeout.

## Global vs Regional Balancing

Global layer:

```text
choose region
```

Regional layer:

```text
choose instance/zone
```

Global traffic steering must align with the data/write-ownership model.

## Session Affinity

Sticky sessions can be practical for legacy/stateful applications but create:

- uneven load;
- difficult failover;
- deployment constraints.

Prefer explicit external state where feasible, but do not externalize state blindly just to avoid affinity.

## Metrics

Track:

- RPS/connection rate;
- active connections;
- backend p95/p99;
- healthy target count;
- retry count;
- rejected/load-shed traffic;
- connection errors;
- TLS handshake errors;
- backend distribution skew.

## Common Mistakes

- round robin assumed sufficient for WebSockets;
- liveness used as readiness;
- retries at every layer;
- no draining;
- all health checks fail because one optional DB/cache is down;
- global failover without safe data authority;
- assuming a managed LB is a single VM/SPOF.

## 2-Minute Interview Answer

> “I choose L4 or L7 based on whether routing needs application semantics. I use readiness-based health, draining, and slow start for safe lifecycle management. The algorithm follows workload shape—least-connections may fit long-lived streams better than round robin. I define retry ownership so the balancer does not amplify failures, and I shed excess work before queues destroy p99 latency. Global routing is tied to the data failover model.”

## References

- [RFC 9110 — HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html)
