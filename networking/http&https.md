# HTTP and HTTPS

HTTP defines application-level request/response semantics. HTTPS is HTTP carried over a secure transport using TLS.

For system design, the highest-value HTTP topics are **method semantics, caching, conditional requests, idempotency, timeouts/retries, intermediaries, and security boundaries**.

## Interview TL;DR

1. HTTP semantics are shared across HTTP/1.1, HTTP/2, and HTTP/3.
2. `GET`, `HEAD`, `OPTIONS`, and `TRACE` are defined as safe; safe methods are read-oriented semantics, not “nothing anywhere changes.”
3. `PUT`, `DELETE`, and safe methods are idempotent by HTTP semantics; `POST` is not inherently idempotent.
4. An idempotent HTTP method does not automatically protect a business operation from duplicate side effects if the server implementation violates the contract.
5. Caching behavior is driven by cache controls, validators, and cache keys—not merely by using `GET`.
6. Timeouts and retries must respect method/business idempotency and an end-to-end deadline.
7. HTTPS protects data in transit to the TLS termination point; the trust boundary continues behind the terminator.
8. Proxies/gateways are part of HTTP behavior and can affect headers, retries, caching, and client identity.

## Request and Response

Example:

```http
POST /v1/orders HTTP/1.1
Host: api.example.com
Content-Type: application/json
Idempotency-Key: checkout-abc

{"productId":"p1","quantity":1}
```

Response:

```http
HTTP/1.1 201 Created
Location: /v1/orders/o123
Content-Type: application/json

{"id":"o123","status":"created"}
```

## Method Semantics

| Method | Typical intent | Safe | Idempotent |
|---|---|---:|---:|
| GET | retrieve representation | yes | yes |
| HEAD | retrieve metadata | yes | yes |
| POST | submit/create/action | no | no by definition |
| PUT | replace target state | no | yes |
| PATCH | partial modification | no | not guaranteed |
| DELETE | remove target mapping/state | no | yes |
| OPTIONS | capabilities | yes | yes |

“Idempotent” means repeated identical requests have the same intended effect, not necessarily identical response codes or logs.

## Status Codes

Use status codes to communicate protocol-level outcome.

Examples:

- `200` successful retrieval/action;
- `201` created;
- `202` accepted for async processing;
- `204` successful with no response content;
- `400` malformed/invalid request;
- `401` authentication required/failed;
- `403` understood but not permitted;
- `404` target not found;
- `409` state conflict;
- `412` precondition failed;
- `429` rate limited;
- `503` temporarily unavailable.

Do not encode every business outcome as `200`.

## Conditional Requests

Validators allow safe optimistic/cache operations.

Example:

```http
ETag: "order-v7"
```

Client update:

```http
If-Match: "order-v7"
```

A mismatch can return:

```http
412 Precondition Failed
```

This can prevent lost updates in API workflows.

## Caching

Important response controls include:

```http
Cache-Control: public, max-age=300
ETag: "product-v42"
Vary: Accept-Encoding
```

Design:

- what is cacheable?
- cache key?
- private vs public?
- freshness?
- revalidation?
- sensitive headers/cookies?
- invalidation?

See [CDN](cdn.md).

## Timeouts and Retries

Use an end-to-end deadline.

Retry only when:

- failure is plausibly transient;
- remaining deadline permits;
- method/business operation is safe to repeat or protected by idempotency.

Beware retries at multiple layers.

## HTTPS and TLS Termination

```text
client
  ↓ TLS
edge / load balancer
  ↓ HTTP or TLS
backend
```

If TLS terminates at the edge, plaintext may exist behind it unless internal TLS is used.

Protect:

- forwarding headers;
- internal network;
- certificates/keys;
- service identity.

See [SSL/TLS](ssl&tls.md).

## Client Identity Behind Proxies

Headers such as:

```http
Forwarded:
X-Forwarded-For:
X-Forwarded-Proto:
```

must be trusted only from known proxy infrastructure. Untrusted clients can spoof forwarding headers.

## Common Mistakes

- “POST cannot be retried” instead of discussing idempotency;
- treating `DELETE` idempotence as “same response every time”;
- assuming HTTPS means every internal hop is encrypted;
- cache sensitive responses without considering authorization/cookies;
- retrying after deadline expiry;
- treating HTTP/2/3 as different application semantics.

## 2-Minute Interview Answer

> “I use HTTP semantics as part of the reliability design: correct method semantics, conditional requests, cache controls, and status codes matter. Retries are bounded by a deadline and only safe when the business operation is idempotent. HTTPS protects the connection to the TLS termination point, so I define the trust boundary behind the proxy as well. For read-heavy paths I use validators and explicit cache policy rather than assuming GET alone is cache-safe.”

## References

- [RFC 9110 — HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html)
- [RFC 9111 — HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111.html)
- [RFC 9112 — HTTP/1.1](https://www.rfc-editor.org/rfc/rfc9112.html)
