# HTTP and HTTPS

HTTP, or Hypertext Transfer Protocol, is an application-layer protocol used for communication between clients and servers.

A typical HTTP interaction follows a request-response model:

1. A client sends an HTTP request.
2. The server processes the request.
3. The server returns an HTTP response.

HTTPS is HTTP secured using TLS, or Transport Layer Security. It protects communication through encryption, authentication, and data-integrity checks.

```text
Client
  |
  | HTTP Request
  v
Server
  |
  | HTTP Response
  v
Client
```

With HTTPS:

```text
Client
  |
  | Encrypted HTTPS Request
  v
TLS Termination / Server
  |
  | Encrypted HTTPS Response
  v
Client
```

HTTP and HTTPS are fundamental to:

* REST APIs
* Web applications
* Mobile applications
* Microservices
* API gateways
* Load balancers
* Service-to-service communication
* Content delivery networks

## Why Does It Matter?

HTTP is the standard communication protocol for most web-based systems.

A strong understanding of HTTP and HTTPS helps backend engineers:

* Design reliable APIs
* Improve application security
* Reduce latency
* Implement effective caching
* Debug production issues
* Configure reverse proxies and load balancers
* Protect sensitive user data
* Build scalable distributed systems

Poor HTTP design can lead to:

* Security vulnerabilities
* Slow response times
* Duplicate operations
* Inefficient network usage
* Cache inconsistencies
* Difficult-to-maintain APIs
* Unexpected client behavior

HTTPS is especially important because HTTP traffic is transmitted as plain text. Without TLS, attackers may be able to read or modify data sent between a client and server.

---

## Core Concepts

### 1. HTTP Request

An HTTP request contains:

* HTTP method
* URL
* Headers
* Optional request body
* HTTP protocol version

Example:

```http
POST /api/v1/orders HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer <token>
Idempotency-Key: order-12345

{
  "productId": "prod-101",
  "quantity": 2
}
```

---

### 2. HTTP Response

An HTTP response contains:

* Status code
* Headers
* Optional response body
* HTTP protocol version

Example:

```http
HTTP/1.1 201 Created
Content-Type: application/json
Location: /api/v1/orders/order-789

{
  "id": "order-789",
  "status": "created"
}
```

---

### 3. HTTP Methods

| Method    | Typical Purpose                          | Idempotent | Safe |
| --------- | ---------------------------------------- | ---------: | ---: |
| `GET`     | Retrieve a resource                      |        Yes |  Yes |
| `POST`    | Create a resource or execute an action   |         No |   No |
| `PUT`     | Replace a resource                       |        Yes |   No |
| `PATCH`   | Partially update a resource              | Not always |   No |
| `DELETE`  | Delete a resource                        |    Usually |   No |
| `HEAD`    | Retrieve headers without a response body |        Yes |  Yes |
| `OPTIONS` | Discover supported operations            |        Yes |  Yes |

### Safe Methods

A safe method should not change server state.

Examples:

```http
GET /users/123
HEAD /documents/456
```

### Idempotent Methods

An idempotent operation produces the same intended server state when executed multiple times.

For example:

```http
PUT /users/123
```

Sending the same `PUT` request repeatedly should leave the resource in the same state.

---

### 4. Status Codes

HTTP status codes are divided into categories.

#### Informational: `1xx`

The request has been received and processing continues.

#### Success: `2xx`

| Code             | Meaning                                      |
| ---------------- | -------------------------------------------- |
| `200 OK`         | Request completed successfully               |
| `201 Created`    | A new resource was created                   |
| `202 Accepted`   | Request accepted for asynchronous processing |
| `204 No Content` | Request succeeded without a response body    |

#### Redirection: `3xx`

| Code                     | Meaning                                  |
| ------------------------ | ---------------------------------------- |
| `301 Moved Permanently`  | Resource permanently moved               |
| `302 Found`              | Temporary redirect                       |
| `304 Not Modified`       | Cached response is still valid           |
| `307 Temporary Redirect` | Temporary redirect preserving the method |
| `308 Permanent Redirect` | Permanent redirect preserving the method |

#### Client Errors: `4xx`

| Code                        | Meaning                                        |
| --------------------------- | ---------------------------------------------- |
| `400 Bad Request`           | Invalid request                                |
| `401 Unauthorized`          | Authentication is required or invalid          |
| `403 Forbidden`             | Client is authenticated but lacks permission   |
| `404 Not Found`             | Resource does not exist                        |
| `409 Conflict`              | Request conflicts with current resource state  |
| `422 Unprocessable Content` | Request syntax is valid, but validation failed |
| `429 Too Many Requests`     | Rate limit exceeded                            |

#### Server Errors: `5xx`

| Code                        | Meaning                                   |
| --------------------------- | ----------------------------------------- |
| `500 Internal Server Error` | Unexpected server failure                 |
| `502 Bad Gateway`           | Invalid response from an upstream service |
| `503 Service Unavailable`   | Service is temporarily unavailable        |
| `504 Gateway Timeout`       | Upstream service did not respond in time  |

---

### 5. HTTP Headers

Headers provide metadata about requests and responses.

Common request headers:

```http
Authorization: Bearer <token>
Accept: application/json
Content-Type: application/json
User-Agent: example-client/1.0
If-None-Match: "resource-version-1"
X-Request-ID: req-123
```

Common response headers:

```http
Content-Type: application/json
Cache-Control: public, max-age=300
ETag: "resource-version-1"
Location: /users/123
Retry-After: 60
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

---

### 6. Statelessness

HTTP is stateless by default.

Each request should contain enough information for the server to process it independently.

The server does not automatically remember previous requests from the same client.

State can still be implemented using:

* Cookies
* Sessions
* Access tokens
* Refresh tokens
* Database records
* Distributed caches

Example:

```text
Client Request
   |
   | Authorization Token
   v
Load Balancer
   |
   v
Any Available Backend Instance
```

Stateless services are easier to scale horizontally because any healthy server can process the request.

---

### 7. Cookies and Sessions

A server may create a session and send a session identifier to the client through a cookie.

```http
Set-Cookie: sessionId=abc123; Secure; HttpOnly; SameSite=Lax
```

The client includes the cookie in later requests:

```http
Cookie: sessionId=abc123
```

Important cookie attributes:

* `Secure`: Sends the cookie only over HTTPS
* `HttpOnly`: Prevents JavaScript access
* `SameSite`: Helps reduce CSRF attacks
* `Max-Age`: Defines cookie lifetime
* `Domain`: Defines valid domains
* `Path`: Defines valid URL paths

---

### 8. Authentication and Authorization

Authentication verifies identity.

Authorization determines what an authenticated identity is allowed to do.

Common authentication approaches:

* Session-based authentication
* API keys
* Basic authentication
* Bearer tokens
* OAuth 2.0
* OpenID Connect
* Mutual TLS

Example:

```http
Authorization: Bearer eyJhbGciOi...
```

A secure backend should validate:

* Token signature
* Token issuer
* Token audience
* Token expiration
* User permissions
* Resource ownership

---

### 9. Caching

HTTP caching reduces latency and backend load.

Common cache headers:

```http
Cache-Control: public, max-age=3600
ETag: "user-123-v5"
Last-Modified: Wed, 22 Jul 2026 10:00:00 GMT
```

A client can perform a conditional request:

```http
GET /users/123 HTTP/1.1
If-None-Match: "user-123-v5"
```

If the resource has not changed:

```http
HTTP/1.1 304 Not Modified
```

Caching may happen at multiple levels:

```text
Client Cache
    |
Browser Cache
    |
CDN Cache
    |
Reverse Proxy Cache
    |
Application Cache
    |
Database
```

---

### 10. Content Negotiation

Clients and servers can negotiate response formats.

Request:

```http
Accept: application/json
Accept-Encoding: gzip, br
Accept-Language: en-US
```

Response:

```http
Content-Type: application/json
Content-Encoding: br
Content-Language: en-US
```

---

### 11. Persistent Connections

Opening a new network connection for every request is expensive.

Persistent connections allow multiple requests to reuse an existing connection.

Benefits include:

* Reduced connection setup overhead
* Lower latency
* Better resource utilization
* Improved throughput

HTTP/1.1 commonly uses persistent connections by default.

---

### 12. TLS

TLS secures HTTPS communication.

TLS provides:

* Encryption
* Server authentication
* Message integrity
* Optional client authentication

A simplified TLS flow:

```text
Client                         Server
  |                              |
  |------ Client Hello --------->|
  |<----- Server Hello ----------|
  |<----- Certificate -----------|
  |------ Key Agreement -------->|
  |                              |
  |==== Encrypted Connection ====|
```

The server certificate proves that the server controls the requested domain, assuming the certificate is valid and trusted.

---

## Architecture

A common production HTTP architecture looks like this:

```text
                        ┌─────────────────┐
                        │     Client      │
                        │ Web / Mobile /  │
                        │ API Consumer    │
                        └────────┬────────┘
                                 │
                               HTTPS
                                 │
                        ┌────────▼────────┐
                        │      DNS        │
                        └────────┬────────┘
                                 │
                        ┌────────▼────────┐
                        │      CDN        │
                        │ Static Content  │
                        │ Edge Caching    │
                        └────────┬────────┘
                                 │
                        ┌────────▼────────┐
                        │ WAF / DDoS      │
                        │ Protection      │
                        └────────┬────────┘
                                 │
                        ┌────────▼────────┐
                        │ Load Balancer   │
                        │ TLS Termination │
                        └────────┬────────┘
                                 │
                ┌────────────────┼────────────────┐
                │                │                │
       ┌────────▼──────┐ ┌───────▼──────┐ ┌───────▼───────┐
       │ Backend API 1 │ │ Backend API 2│ │ Backend API 3 │
       └────────┬──────┘ └────────┬─────┘ └─────┬─────────┘
                │                 │             │
                └─────────────────┼─────────────┘
                                  │
                   ┌──────────────┼──────────────┐
                   │              │              │
           ┌───────▼──────┐ ┌─────▼────┐ ┌───────▼─────┐
           │ Distributed  │ │ Database │ │ Message     │
           │ Cache        │ │ Cluster  │ │ Queue       │
           └──────────────┘ └──────────┘ └─────────────┘
```

### Request Flow

1. The client resolves the domain using DNS.
2. The client establishes a connection to the edge or load balancer.
3. A TLS handshake establishes an encrypted session.
4. The CDN serves cached content when possible.
5. The WAF filters malicious traffic.
6. The load balancer routes the request to a healthy backend instance.
7. The backend authenticates and validates the request.
8. The backend accesses the cache, database, or another service.
9. The response travels back through the infrastructure.
10. Relevant metrics, traces, and logs are recorded.

---

## HTTP Version Comparison

| Feature               | HTTP/1.1                  | HTTP/2                     | HTTP/3                              |
| --------------------- | ------------------------- | -------------------------- | ----------------------------------- |
| Transport             | TCP                       | TCP                        | QUIC over UDP                       |
| Request multiplexing  | Limited                   | Yes                        | Yes                                 |
| Header compression    | No                        | HPACK                      | QPACK                               |
| Binary framing        | No                        | Yes                        | Yes                                 |
| Connection setup      | TCP                       | TCP + TLS                  | QUIC + TLS                          |
| Head-of-line blocking | Application and TCP level | TCP level                  | Reduced through independent streams |
| Server push           | Limited support           | Supported, but rarely used | Limited adoption                    |
| Best use case         | Simple or legacy systems  | Modern web applications    | High-latency or unreliable networks |

### HTTP/1.1

HTTP/1.1 is text-based and widely supported.

Its limitations include:

* Multiple connections are often needed
* Requests may block one another
* Headers are repeatedly transmitted
* Connection utilization is less efficient

### HTTP/2

HTTP/2 introduces binary framing and multiplexing.

Multiple streams can share one TCP connection:

```text
Single TCP Connection
   |
   ├── Stream 1: GET /users
   ├── Stream 2: GET /orders
   └── Stream 3: POST /payments
```

However, packet loss at the TCP level may still delay all streams.

### HTTP/3

HTTP/3 uses QUIC over UDP.

Advantages include:

* Faster connection establishment
* Better behavior during packet loss
* Independent streams
* Improved connection migration between networks

---

## HTTP vs HTTPS Comparison

| Category                   | HTTP           | HTTPS                          |
| -------------------------- | -------------- | ------------------------------ |
| Encryption                 | No             | Yes                            |
| Default port               | `80`           | `443`                          |
| Server authentication      | No             | Yes, through certificates      |
| Data integrity             | Not guaranteed | Protected by TLS               |
| Safe for credentials       | No             | Yes, when correctly configured |
| Vulnerable to interception | Yes            | Significantly reduced          |
| Production recommendation  | Avoid          | Required                       |

HTTPS protects data in transit, but it does not automatically secure the entire application.

A backend can still be vulnerable to:

* SQL injection
* Broken authorization
* Cross-site scripting
* Server-side request forgery
* Insecure token storage
* Application logic flaws

---

## REST vs RPC vs GraphQL

| Category        | REST                | RPC                       | GraphQL                            |
| --------------- | ------------------- | ------------------------- | ---------------------------------- |
| Primary model   | Resources           | Actions and procedures    | Query graph                        |
| Example         | `GET /users/123`    | `POST /getUser`           | `query { user(id: 123) }`          |
| Caching         | Strong HTTP support | Depends on implementation | Often requires custom handling     |
| Flexibility     | Moderate            | High for actions          | High for client-selected fields    |
| Complexity      | Low to moderate     | Low to moderate           | Moderate to high                   |
| Common use case | Public APIs         | Internal services         | Complex frontend data requirements |

---

## Real-World Example: E-Commerce Order Creation

Consider an API that creates an order.

### Request

```http
POST /api/v1/orders HTTP/1.1
Host: api.shop.example
Authorization: Bearer <access-token>
Content-Type: application/json
Idempotency-Key: checkout-session-987
X-Request-ID: req-456

{
  "customerId": "cust-123",
  "items": [
    {
      "productId": "prod-101",
      "quantity": 2
    }
  ],
  "paymentMethodId": "pm-555"
}
```

### Backend Request Flow

```text
Client
  |
  | POST /orders
  v
API Gateway
  |
  | Authentication
  | Rate Limiting
  | Request Validation
  v
Order Service
  |
  ├── Check Idempotency Key
  ├── Validate Inventory
  ├── Calculate Price
  ├── Reserve Inventory
  ├── Create Order
  └── Publish OrderCreated Event
```

### Successful Response

```http
HTTP/1.1 201 Created
Content-Type: application/json
Location: /api/v1/orders/order-789

{
  "id": "order-789",
  "status": "pending",
  "total": {
    "amount": 4998,
    "currency": "USD"
  }
}
```

### Asynchronous Processing

Some operations may continue after the API response:

```text
OrderCreated Event
       |
       ├── Payment Service
       ├── Inventory Service
       ├── Notification Service
       └── Analytics Service
```

The API may return `202 Accepted` when processing cannot be completed immediately.

```http
HTTP/1.1 202 Accepted
Location: /api/v1/jobs/job-456
Retry-After: 5
```

### Failure Examples

Invalid input:

```http
HTTP/1.1 422 Unprocessable Content
Content-Type: application/problem+json

{
  "type": "https://api.shop.example/problems/validation-error",
  "title": "Validation failed",
  "status": 422,
  "errors": [
    {
      "field": "items[0].quantity",
      "message": "Quantity must be greater than zero"
    }
  ]
}
```

Insufficient inventory:

```http
HTTP/1.1 409 Conflict
Content-Type: application/problem+json

{
  "type": "https://api.shop.example/problems/insufficient-inventory",
  "title": "Insufficient inventory",
  "status": 409,
  "productId": "prod-101"
}
```

Rate limit exceeded:

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 30
```

---

## Best Practices

### 1. Use HTTPS Everywhere

Redirect HTTP requests to HTTPS.

```text
http://api.example.com
          |
          | 308 Permanent Redirect
          v
https://api.example.com
```

Use modern TLS configurations and disable obsolete protocols and cipher suites.

---

### 2. Use Correct HTTP Methods

Prefer resource-oriented APIs.

```http
GET    /users/123
POST   /users
PUT    /users/123
PATCH  /users/123
DELETE /users/123
```

Avoid action-heavy endpoint names when a resource model is sufficient.

Less preferable:

```http
POST /getUser
POST /deleteUser
```

---

### 3. Return Meaningful Status Codes

Do not return `200 OK` for every outcome.

Poor response:

```http
HTTP/1.1 200 OK

{
  "success": false,
  "error": "User not found"
}
```

Better response:

```http
HTTP/1.1 404 Not Found
```

---

### 4. Design Idempotent Operations

Retries are common in distributed systems.

Use idempotency keys for non-idempotent operations such as:

* Payment creation
* Order creation
* Subscription creation
* Fund transfer
* Message publishing

```http
Idempotency-Key: payment-request-123
```

The server should store the key and return the original result for duplicate requests.

---

### 5. Implement Timeouts

Every network request should have a timeout.

Without timeouts, requests can consume resources indefinitely.

```text
Client Timeout
    >
Load Balancer Timeout
    >
Application Timeout
    >
Downstream Service Timeout
```

The timeout budget should generally decrease as the request travels deeper into the system.

---

### 6. Retry Carefully

Retry only transient failures.

Good retry candidates:

* Connection timeout
* `502 Bad Gateway`
* `503 Service Unavailable`
* `504 Gateway Timeout`
* `429 Too Many Requests`, when allowed by `Retry-After`

Avoid blindly retrying:

* Validation errors
* Authentication failures
* Authorization failures
* Non-idempotent requests without an idempotency mechanism

Use exponential backoff with jitter.

```text
Attempt 1: 100 ms
Attempt 2: 220 ms
Attempt 3: 470 ms
Attempt 4: 950 ms
```

---

### 7. Use Rate Limiting

Rate limiting protects services from:

* Abuse
* Accidental traffic spikes
* Brute-force attacks
* Resource exhaustion
* Noisy clients

Common algorithms:

* Fixed window
* Sliding window
* Token bucket
* Leaky bucket

Useful response headers:

```http
RateLimit-Limit: 100
RateLimit-Remaining: 12
RateLimit-Reset: 30
Retry-After: 30
```

---

### 8. Validate Request Size

Set limits for:

* Request body size
* Header size
* URL length
* File upload size
* Batch size
* Pagination size

Large unbounded requests can exhaust memory, CPU, bandwidth, or database resources.

---

### 9. Use Pagination

Avoid returning unbounded collections.

Cursor-based pagination:

```http
GET /api/v1/orders?limit=50&cursor=eyJpZCI6MTIzfQ
```

Response:

```json
{
  "items": [],
  "nextCursor": "eyJpZCI6MTczfQ"
}
```

Cursor pagination is often more stable than offset pagination for frequently changing datasets.

---

### 10. Support Observability

Use correlation identifiers:

```http
X-Request-ID: req-123
Traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
```

Collect:

* Request rate
* Error rate
* Latency percentiles
* Response sizes
* Status-code distribution
* Active connections
* Timeout counts
* Retry counts
* Cache hit rate

A useful service-level view includes:

```text
RED Method

Rate
Errors
Duration
```

---

### 11. Protect Sensitive Data

Do not expose sensitive data in:

* URLs
* Query parameters
* Logs
* Error messages
* Response headers
* Monitoring labels

Avoid:

```http
GET /reset-password?token=secret-token
```

URLs may be stored in browser history, proxy logs, analytics systems, and monitoring tools.

Prefer sending sensitive information in a protected request body.

---

### 12. Configure Security Headers

Useful response headers include:

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
Content-Security-Policy: default-src 'self'
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=()
```

Browser-facing applications should also implement a suitable Content Security Policy.

---

### 13. Avoid Exposing Internal Errors

Poor response:

```json
{
  "error": "NullPointerException at OrderService.java:87",
  "database": "orders-primary.internal"
}
```

Better response:

```json
{
  "type": "https://api.example.com/problems/internal-error",
  "title": "An unexpected error occurred",
  "status": 500,
  "requestId": "req-123"
}
```

Store detailed diagnostics in internal logs.

---

### 14. Version APIs Carefully

Common versioning approaches include:

Path versioning:

```http
GET /api/v1/users/123
```

Header versioning:

```http
Accept: application/vnd.example.v1+json
```

Version only when introducing a breaking contract change.

---

### 15. Terminate TLS at the Correct Layer

TLS may terminate at:

* CDN
* Edge proxy
* Load balancer
* API gateway
* Application server

For sensitive systems, re-encrypt traffic between internal components.

```text
Client
  |
  | HTTPS
  v
Load Balancer
  |
  | HTTPS or mTLS
  v
Backend Service
```

---

## Common Mistakes

### 1. Using HTTP in Production

Plain HTTP exposes traffic to interception and modification.

---

### 2. Treating HTTPS as Complete Application Security

HTTPS protects data in transit. It does not fix insecure authentication, broken authorization, injection vulnerabilities, or poor secret management.

---

### 3. Returning `200 OK` for Errors

This makes client behavior, monitoring, retries, and caching less reliable.

---

### 4. Confusing `401` and `403`

Use:

* `401 Unauthorized` when authentication is missing or invalid
* `403 Forbidden` when authentication succeeded but access is denied

---

### 5. Retrying Every Failed Request

Blind retries can:

* Duplicate transactions
* Overload unhealthy services
* Increase latency
* Create retry storms

---

### 6. Ignoring Idempotency

A client may retry an order or payment request after a timeout even though the original operation succeeded.

Without idempotency handling, duplicate records or charges may be created.

---

### 7. Sending Sensitive Data in URLs

URLs frequently appear in logs, browser history, analytics tools, and proxy systems.

---

### 8. Creating Unbounded Endpoints

An endpoint such as the following may return millions of records:

```http
GET /orders
```

Always define pagination and maximum limits.

---

### 9. Caching Private Data Incorrectly

Do not allow shared caches to store user-specific data unless the caching policy is explicitly safe.

Use:

```http
Cache-Control: private, no-store
```

for highly sensitive responses.

---

### 10. Forgetting Proxy Behavior

Backends behind proxies must carefully handle forwarded information.

Relevant headers may include:

```http
Forwarded: for=192.0.2.1;proto=https;host=api.example.com
X-Forwarded-For: 192.0.2.1
X-Forwarded-Proto: https
```

Only trust forwarded headers from known infrastructure. Attackers can provide fake values directly.

---

### 11. Missing Timeouts and Connection Limits

Unbounded connections and long-running requests can exhaust:

* Threads
* File descriptors
* Memory
* Connection pools
* Database connections

---

### 12. Logging Authorization Tokens

Access tokens, session identifiers, API keys, and cookies should be redacted before logging.

---

## Interview Questions

### 1. What is the difference between HTTP and HTTPS?

HTTP sends application data without transport encryption. HTTPS uses TLS to encrypt traffic, verify the server identity, and protect message integrity.

---

### 2. What does idempotency mean in HTTP?

An idempotent request can be repeated without changing the intended final server state beyond the effect of the first successful request. `GET`, `PUT`, and usually `DELETE` are considered idempotent.

---

### 3. What is the difference between `401 Unauthorized` and `403 Forbidden`?

`401` means valid authentication is missing. `403` means the client is authenticated but does not have permission to perform the requested operation.

---

### 4. How does HTTP caching reduce backend load?

Clients, CDNs, and proxies can reuse previously stored responses. Cache headers such as `Cache-Control`, `ETag`, and `Last-Modified` determine whether a response can be reused or revalidated.

---

### 5. Why is HTTP/2 usually faster than HTTP/1.1?

HTTP/2 supports multiplexing, binary framing, and header compression. Multiple requests can share one connection without requiring a separate connection for each request.

---

## Key Takeaways

1. **HTTP defines the communication contract between clients and backend systems.** Correct methods, status codes, headers, caching, and API semantics make systems easier to scale and maintain.

2. **HTTPS is mandatory for modern production systems.** TLS protects data in transit, authenticates servers, and prevents many interception and tampering attacks.

3. **Reliable HTTP systems require more than basic request handling.** Timeouts, retries, idempotency, rate limiting, observability, caching, pagination, and security controls are essential for production-grade backend architecture.

