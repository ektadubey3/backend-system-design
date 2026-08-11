# API Gateway

An API gateway is an API-facing policy and routing layer between clients and backend services.

It is useful when multiple APIs need consistent cross-cutting controls, but it can become a latency bottleneck and an accidental home for business logic.

## Interview TL;DR

1. Gateway centralizes **edge/API policy**, not domain ownership.
2. Good responsibilities: routing, authentication verification, quotas/rate limits, request limits, observability, protocol adaptation, version/tenant routing.
3. Services should still enforce business authorization and invariants.
4. Avoid putting large orchestration/business workflows into the gateway.
5. Aggregation can reduce client chattiness but creates fan-out/tail-latency coupling.
6. Gateway retries must be deadline-aware and safe for the operation.
7. Rate limiting needs explicit identity/key, atomicity, failure policy, and hot-tenant handling.
8. Protect the gateway itself with horizontal capacity, admission control, and dependency isolation.
9. API gateway, reverse proxy, ingress, and load balancer may be implemented by the same product.
10. A BFF is client-experience-specific composition; a gateway is usually broader shared policy.

## Request Path

```text
client
  ↓
API gateway
  ├─ auth verification
  ├─ quota/rate limit
  ├─ route
  └─ telemetry
       ↓
backend service
```

## Authentication vs Authorization

Gateway can verify:

- token signature;
- issuer/audience;
- API key;
- client certificate.

Backend must still enforce:

```text
Can this authenticated principal perform this domain action on this object?
```

Do not centralize every authorization rule at the gateway.

## Rate Limiting

Define:

```text
key = tenant/user/API key/IP/route
algorithm
window/bucket
atomic update
fail-open/fail-closed
multi-region policy
```

See the rate-limiter case study.

## Routing

Can use:

- host;
- path;
- header;
- tenant;
- version;
- canary percentage.

Keep routing policy observable and version-controlled.

## Aggregation

Example:

```text
mobile request
   ↓
gateway/BFF
 ├─ user
 ├─ orders
 └─ recommendations
```

Benefits:

- fewer client round trips;
- tailored response.

Costs:

- fan-out tail latency;
- partial failure handling;
- gateway complexity.

For complex client-specific composition, a dedicated BFF may be clearer.

## Transformation

Useful for:

- legacy protocol adaptation;
- header normalization;
- simple schema mapping.

Avoid heavy business data transformations that make the gateway a monolith.

## Timeouts and Retries

Gateway has only part of the end-to-end deadline.

Propagate remaining budget.

Retry selected safe failures only; do not create hidden duplicate writes.

## Overload

Protect:

- max connections;
- max request/body size;
- per-tenant quota;
- concurrency;
- queue length;
- route priorities.

A gateway outage can affect every API, so keep the critical path small.

## Observability

Track:

- request rate by route/tenant;
- gateway-added latency;
- auth failures;
- rate-limit decisions;
- upstream errors;
- retry count;
- rejected/overload requests;
- config/deployment version.

## Common Mistakes

- domain authorization only at gateway;
- business workflows inside gateway;
- aggregation without partial-failure policy;
- unlimited request transformation;
- retries at gateway and service layers;
- rate limiting by IP only for authenticated multi-tenant APIs;
- treating gateway as mandatory for a simple monolith.

## 2-Minute Interview Answer

> “I add an API gateway when shared edge/API policies justify a centralized layer. It verifies identity, applies quotas and request limits, routes traffic, and emits telemetry, while services keep domain authorization and invariants. I keep gateway business logic small, propagate deadlines, and avoid hidden retries on unsafe writes. If client-specific aggregation grows complex, I use a BFF rather than turning the shared gateway into an orchestration service.”
