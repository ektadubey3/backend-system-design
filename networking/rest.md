# REST

REST is an architectural style centered on resources, representations, stateless interactions, cacheability, and a uniform interface. It is commonly implemented over HTTP.

The interview question is not “REST or not?” It is whether resource-oriented HTTP semantics fit the client and workflow.

## Interview TL;DR

1. REST is an architectural style, not a wire protocol.
2. Model stable business resources rather than RPC verbs in every URL.
3. Use HTTP method, caching, conditional-request, and status semantics correctly.
4. Statelessness means the server can interpret each request without relying on hidden conversational connection state; it does not mean the system stores no user state.
5. Use idempotency keys for retried create/action operations when required.
6. Pagination, filtering, versioning, and error contracts are part of API design.
7. `202 Accepted` is useful when work is durable but not complete.
8. REST is not ideal for every use case: highly interactive client-shaped aggregation, strict typed RPC, or long-lived bidirectional streaming may fit other styles better.
9. Avoid chatty APIs and N+1 client workflows.
10. Backward-compatible evolution is usually more important than URL-version aesthetics.

## Resource Modeling

Prefer:

```http
GET  /v1/orders/123
POST /v1/orders
GET  /v1/orders/123/items
```

over RPC-like naming everywhere:

```http
POST /getOrder
POST /createOrder
POST /getOrderItems
```

Actions that do not map cleanly to CRUD can still be modeled explicitly:

```http
POST /v1/orders/123/cancellation
```

Do not force awkward purity.

## Stateless Requests

The request should include enough context to be processed without affinity to a specific application process.

State can still live in:

- database;
- cache/session store;
- token;
- object store.

## Idempotency

For create/action endpoints that clients may retry:

```http
POST /v1/payments
Idempotency-Key: order-123-payment
```

Store the key durably with the result.

HTTP `POST` does not become protocol-idempotent, but the application operation can be made safe to retry.

## Pagination

### Offset

Simple, supports page numbers, but deep offsets may be expensive and changing data can shift results.

### Cursor/keyset

Better for large ordered streams.

Use a deterministic ordering:

```text
(created_at, id)
```

not timestamp alone when ties are possible.

## Filtering and Sorting

Bound complexity:

- allowed filter fields;
- maximum page size;
- date-range limits;
- allowed sort keys;
- expensive relationship expansion.

Public query flexibility can become a database DoS vector.

## Async Operations

If work is accepted but continues asynchronously:

```http
POST /v1/exports
```

Response:

```http
202 Accepted
Location: /v1/exports/job-123
```

The job status becomes a resource.

## Caching

REST can benefit from standard HTTP caching:

- `Cache-Control`;
- `ETag`;
- `Last-Modified`;
- conditional requests.

Do not cache user-specific responses in shared caches without a deliberate cache key/privacy policy.

## API Evolution

Prefer additive changes when possible:

- add optional fields;
- add endpoints;
- preserve old enum values/semantics;
- use tolerant clients.

Version when semantics cannot remain backward compatible.

## Error Contract

Return machine-readable errors with:

- stable code/type;
- human message;
- field violations where relevant;
- correlation/request ID.

Do not make clients parse free-form strings.

## REST vs Alternatives

### REST

Strong for:

- public APIs;
- resource-oriented CRUD/workflows;
- HTTP caching;
- broad tooling.

### GraphQL

Useful for client-shaped connected-data queries.

### gRPC

Useful for typed internal RPC/streaming.

### WebSockets

Useful for long-lived bidirectional application messaging.

## Common Mistakes

- “REST = JSON over HTTP”;
- “stateless = no database/session state”;
- `200 OK` for every error;
- deep offset pagination at arbitrary scale;
- versioning every additive field change;
- chatty client flows that require dozens of endpoints;
- retrying POST without application idempotency.

## 2-Minute Interview Answer

> “I choose REST when resource-oriented HTTP semantics fit the client. I use correct method/status/cache semantics, make retried create operations idempotent at the application layer, and bound filtering/pagination complexity. For async work I return a job/status resource. I would choose GraphQL for highly client-shaped aggregation or gRPC for strongly typed internal RPC rather than forcing REST into every communication pattern.”

## References

- [RFC 9110 — HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html)
- Roy Fielding, *Architectural Styles and the Design of Network-based Software Architectures*.
