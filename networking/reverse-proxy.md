# Reverse Proxy

A reverse proxy accepts client traffic on behalf of one or more upstream servers and forwards requests to those upstreams.

“Reverse proxy,” “load balancer,” and “API gateway” overlap in product implementations. Treat them as **roles/capabilities**, not mutually exclusive appliance categories.

## Interview TL;DR

1. Reverse proxy is the fronting/intermediation role; it can also load-balance, cache, terminate TLS, or enforce policy.
2. Do not distinguish reverse proxy vs load balancer using rigid product labels—NGINX/Envoy/cloud products can perform multiple roles.
3. Trust forwarded client headers only from known proxy hops.
4. Define timeouts, buffering, body-size limits, and backpressure.
5. TLS termination creates a new trust boundary.
6. Proxy retries can duplicate unsafe operations and amplify incidents.
7. Proxy buffering can protect slow clients/backends but consumes memory/disk and changes streaming behavior.
8. Upstream connection reuse/pooling is a major performance concern.
9. Keep upstreams private when proxy policy is security-critical.
10. Monitor proxy latency separately from upstream latency.

## Request Path

```text
client
  ↓
reverse proxy
  ↓
upstream service
```

Possible proxy responsibilities:

- TLS termination;
- routing;
- upstream selection;
- caching;
- compression;
- buffering;
- header normalization;
- request-size limits;
- WAF/rate limit integration.

## Routing

Examples:

```text
/api/users   → user service
/api/orders  → order service
/static      → object storage
```

This does not automatically make the reverse proxy an API gateway; the distinction is organizational/feature scope.

## Forwarded Headers

A proxy may add:

```http
Forwarded:
X-Forwarded-For:
X-Forwarded-Proto:
X-Request-ID:
```

Backends should trust them only when:

- direct public access is blocked or controlled;
- the header was sanitized/replaced by trusted infrastructure.

## TLS Termination

```text
client --TLS--> proxy --TLS/HTTP--> upstream
```

Choose internal encryption based on the trust/security requirement.

## Buffering

### Request buffering

Proxy may read request body before sending upstream.

Useful for:

- protecting upstream from slow uploads;
- applying size limits.

Cost:

- memory/disk;
- latency;
- unsuitable for some streaming.

### Response buffering

Can decouple a slow client from upstream.

For realtime/streaming, buffering may need to be disabled/tuned.

## Timeouts

Define separately:

- client/header timeout;
- connect timeout;
- upstream response timeout;
- idle timeout;
- total request deadline where platform supports it.

Timeout mismatch across proxies creates confusing failures.

## Upstream Reuse

Reuse connections to upstream services.

Without reuse:

```text
proxy → TCP/TLS setup → upstream
for every request
```

Connection pooling/keep-alive reduces handshake overhead but must be bounded.

## Retry Policy

Retry selected safe failures only.

If the upstream may have committed work before the connection broke, an automatic proxy retry can duplicate the operation.

## Caching

Reverse proxies can cache public/read-heavy responses.

Follow HTTP cache semantics and privacy-aware keys.

## Security

Use:

- upstream network isolation;
- request limits;
- header sanitation;
- TLS policy;
- rate limiting/WAF as required.

Hiding an IP address alone is not a security boundary.

## Reverse Proxy vs Load Balancer vs API Gateway

### Reverse proxy

Fronts upstreams.

### Load balancer

Selects a backend/target.

### API gateway

Adds API-specific policy, product lifecycle, auth/quota/transformation/analytics.

One product can perform all three roles.

## Common Mistakes

- rigid “proxy vs LB” feature tables treated as protocol law;
- trusting client-supplied `X-Forwarded-For`;
- retrying POST/mutations automatically;
- buffering realtime streams unexpectedly;
- plaintext internal traffic without an explicit trust decision;
- public upstreams bypassing proxy policy.

## 2-Minute Interview Answer

> “I use a reverse proxy as the controlled front door to upstream services. I define TLS termination, trusted forwarding headers, upstream connection reuse, buffering, body limits, and timeout budgets. I treat retries as a reliability risk for non-idempotent operations. Whether the same proxy also acts as load balancer or API gateway is an implementation choice, not a strict conceptual boundary.”
