# REST

REST, or Representational State Transfer, is an architectural style for building distributed systems around resources, standard HTTP semantics, and stateless communication.

In a RESTful system, clients interact with resources through URLs and use HTTP methods to perform operations.

Example:

```http
GET /api/v1/users/123
```

The server returns a representation of the resource:

```json
{
  "id": "123",
  "name": "Alex Morgan",
  "email": "alex@example.com"
}
```

REST is not a protocol. It is a set of architectural constraints and design principles commonly implemented over HTTP.

A typical REST API uses:

* Resource-oriented URLs
* HTTP methods
* HTTP status codes
* Request and response headers
* JSON representations
* Stateless requests
* Cacheable responses
* Layered infrastructure

REST is widely used for:

* Public APIs
* Mobile backends
* Web applications
* Microservices
* Partner integrations
* Internal business services
* Backend-for-frontend systems

---

## Why REST?

REST is popular because it aligns naturally with HTTP and provides a simple, standardized way for clients and servers to communicate.

### 1. Simplicity

REST uses familiar HTTP concepts.

```http
GET    /products
POST   /products
GET    /products/123
PATCH  /products/123
DELETE /products/123
```

Developers do not need a custom transport protocol to understand the API.

---

### 2. Broad Compatibility

REST APIs work with:

* Browsers
* Mobile applications
* Command-line tools
* Backend services
* API gateways
* Load balancers
* CDNs
* Proxies

Nearly every programming language has mature HTTP client and server libraries.

---

### 3. Scalability

REST encourages stateless communication.

Each request contains the information required to process it.

```text
Client
  |
  | Request + Authentication
  v
Load Balancer
  |
  ├── Backend Instance 1
  ├── Backend Instance 2
  └── Backend Instance 3
```

Because requests are independent, any healthy backend instance can handle them.

---

### 4. Cacheability

REST can use standard HTTP caching.

```http
Cache-Control: public, max-age=300
ETag: "product-123-v7"
```

Caching can reduce:

* Latency
* Database queries
* Backend CPU usage
* Network traffic
* Origin-server load

---

### 5. Loose Coupling

Clients depend on resource contracts rather than internal server implementation.

A client does not need to know:

* Which database is used
* How many services exist
* Where business logic runs
* Whether data comes from a cache
* Whether processing is synchronous or asynchronous

---

### 6. Observability

REST works well with existing infrastructure for:

* Access logs
* Metrics
* Distributed tracing
* Rate limiting
* Authentication
* Request inspection
* Traffic routing
* Error monitoring

---

### 7. Evolvability

REST APIs can evolve through:

* Backward-compatible field additions
* New endpoints
* Optional query parameters
* API versioning
* Content negotiation

A well-designed REST contract allows the backend to change without breaking clients.

---

## Core Concepts

## 1. Resources

A resource is the central concept in REST.

A resource represents a business entity or concept.

Examples:

* User
* Product
* Order
* Payment
* Subscription
* Invoice
* Notification

Resources are identified using URLs.

```http
/users
/users/123
/orders/456
/products/789/reviews
```

Use nouns rather than verbs.

Prefer:

```http
POST /orders
DELETE /orders/123
```

Avoid:

```http
POST /createOrder
POST /deleteOrder
```

---

## 2. Resource Representations

Clients do not usually receive the actual internal resource object.

They receive a representation of the resource.

Common formats include:

* JSON
* XML
* HTML
* Protocol-specific media types

JSON example:

```json
{
  "id": "order-789",
  "status": "pending",
  "total": {
    "amount": 12999,
    "currency": "USD"
  }
}
```

The internal database model does not need to match the external API representation.

---

## 3. HTTP Methods

REST APIs use HTTP methods to express intent.

| Method    | Purpose                                | Safe | Idempotent |
| --------- | -------------------------------------- | ---: | ---------: |
| `GET`     | Retrieve a resource                    |  Yes |        Yes |
| `POST`    | Create a resource or trigger an action |   No |         No |
| `PUT`     | Replace a resource                     |   No |        Yes |
| `PATCH`   | Partially update a resource            |   No | Not always |
| `DELETE`  | Delete a resource                      |   No |    Usually |
| `HEAD`    | Retrieve headers only                  |  Yes |        Yes |
| `OPTIONS` | Discover supported operations          |  Yes |        Yes |

### GET

```http
GET /api/v1/products/123
```

Use `GET` only for retrieval.

A `GET` request should not modify server state.

---

### POST

```http
POST /api/v1/orders
Content-Type: application/json

{
  "customerId": "cust-123",
  "items": [
    {
      "productId": "prod-456",
      "quantity": 2
    }
  ]
}
```

Use `POST` to create a resource or perform a non-idempotent operation.

---

### PUT

```http
PUT /api/v1/users/123
Content-Type: application/json

{
  "name": "Alex Morgan",
  "email": "alex@example.com"
}
```

`PUT` commonly replaces the full representation of a resource.

Sending the same request repeatedly should produce the same intended final state.

---

### PATCH

```http
PATCH /api/v1/users/123
Content-Type: application/json

{
  "email": "new-email@example.com"
}
```

Use `PATCH` for partial updates.

The API should clearly define patch semantics.

---

### DELETE

```http
DELETE /api/v1/users/123
```

A successful delete may return:

```http
HTTP/1.1 204 No Content
```

---

## 4. HTTP Status Codes

Status codes communicate the result of a request.

### Successful Responses

| Code             | Meaning                                      |
| ---------------- | -------------------------------------------- |
| `200 OK`         | Request completed successfully               |
| `201 Created`    | Resource created successfully                |
| `202 Accepted`   | Request accepted for asynchronous processing |
| `204 No Content` | Request succeeded with no response body      |

### Client Errors

| Code                        | Meaning                                   |
| --------------------------- | ----------------------------------------- |
| `400 Bad Request`           | Invalid request syntax or malformed input |
| `401 Unauthorized`          | Authentication is missing or invalid      |
| `403 Forbidden`             | Authenticated client lacks permission     |
| `404 Not Found`             | Resource does not exist                   |
| `409 Conflict`              | Request conflicts with current state      |
| `412 Precondition Failed`   | Conditional request failed                |
| `422 Unprocessable Content` | Validation failed                         |
| `429 Too Many Requests`     | Rate limit exceeded                       |

### Server Errors

| Code                        | Meaning                         |
| --------------------------- | ------------------------------- |
| `500 Internal Server Error` | Unexpected server failure       |
| `502 Bad Gateway`           | Invalid upstream response       |
| `503 Service Unavailable`   | Service temporarily unavailable |
| `504 Gateway Timeout`       | Upstream service timed out      |

---

## 5. Statelessness

REST communication should be stateless.

The server should not depend on conversational state stored in a specific application instance.

Each request should include:

* Authentication information
* Request parameters
* Resource identifier
* Required headers
* Request body, when needed

```text
Request 1
Authorization: Bearer token

Request 2
Authorization: Bearer token
```

Statelessness makes horizontal scaling easier.

State may still exist in:

* Databases
* Distributed caches
* Object storage
* Session stores
* Message queues

The important rule is that application instances should not require local memory from previous requests.

---

## 6. Uniform Interface

REST uses a consistent interface for interacting with resources.

The main elements are:

* Resource identification through URLs
* Manipulation through representations
* Self-descriptive messages
* Hypermedia links, where applicable

Example:

```http
GET /api/v1/orders/789
```

```json
{
  "id": "order-789",
  "status": "shipped",
  "links": {
    "self": "/api/v1/orders/789",
    "tracking": "/api/v1/orders/789/tracking",
    "customer": "/api/v1/customers/cust-123"
  }
}
```

---

## 7. Idempotency

An operation is idempotent when repeating it produces the same intended final server state.

Example:

```http
PUT /api/v1/users/123
```

Sending the same `PUT` request multiple times should leave the user in the same state.

`POST` is not inherently idempotent.

For sensitive create operations, use an idempotency key.

```http
POST /api/v1/payments
Idempotency-Key: payment-req-abc123
```

The server stores the result associated with the key and returns the same result for retries.

This helps prevent:

* Duplicate orders
* Duplicate payments
* Duplicate subscriptions
* Duplicate messages

---

## 8. Query Parameters

Query parameters commonly support:

* Filtering
* Sorting
* Searching
* Pagination
* Field selection

Example:

```http
GET /api/v1/products?category=laptop&sort=-price&limit=20
```

Possible conventions:

```text
Filtering:
?status=active

Sorting:
?sort=createdAt
?sort=-createdAt

Search:
?q=wireless

Pagination:
?limit=20&cursor=abc123

Field selection:
?fields=id,name,price
```

Keep query behavior predictable and documented.

---

## 9. Pagination

Large collections should always be paginated.

### Offset Pagination

```http
GET /api/v1/orders?limit=20&offset=40
```

Advantages:

* Simple
* Easy to understand
* Supports page numbers

Limitations:

* Can become slow for large offsets
* Results may shift when data changes

### Cursor Pagination

```http
GET /api/v1/orders?limit=20&cursor=eyJpZCI6Nzg5fQ
```

Response:

```json
{
  "items": [],
  "nextCursor": "eyJpZCI6ODA5fQ"
}
```

Advantages:

* More efficient for large datasets
* More stable when records are inserted or deleted
* Better for feeds and timelines

---

## 10. Filtering and Sorting

Filtering should use consistent field names.

```http
GET /api/v1/orders?status=shipped&customerId=cust-123
```

Sorting may support ascending and descending order.

```http
GET /api/v1/orders?sort=createdAt
GET /api/v1/orders?sort=-createdAt
```

Always define:

* Supported fields
* Default sort order
* Maximum filter complexity
* Indexing implications

---

## 11. Content Negotiation

Clients can specify acceptable response formats.

```http
Accept: application/json
```

The server responds with:

```http
Content-Type: application/json
```

Custom media types may be used for versioning or specialized representations.

```http
Accept: application/vnd.example.order.v2+json
```

---

## 12. Caching

REST APIs can use HTTP caching headers.

```http
Cache-Control: public, max-age=300
ETag: "product-123-v4"
```

Conditional request:

```http
GET /api/v1/products/123
If-None-Match: "product-123-v4"
```

Unchanged response:

```http
HTTP/1.1 304 Not Modified
```

Common caching headers:

* `Cache-Control`
* `ETag`
* `Last-Modified`
* `If-None-Match`
* `If-Modified-Since`
* `Vary`

Sensitive responses may require:

```http
Cache-Control: private, no-store
```

---

## 13. API Versioning

Versioning protects clients from breaking changes.

### URL Versioning

```http
GET /api/v1/users/123
```

Advantages:

* Easy to understand
* Easy to route
* Visible in logs

### Header Versioning

```http
Accept: application/vnd.example.v2+json
```

Advantages:

* Keeps URLs resource-focused
* Supports representation versioning

Version only for breaking changes.

Backward-compatible changes may include:

* Adding optional fields
* Adding endpoints
* Adding optional query parameters
* Adding new enum values when clients handle them safely

---

## 14. Error Responses

Use a consistent error format.

Example:

```json
{
  "type": "https://api.example.com/problems/validation-error",
  "title": "Validation failed",
  "status": 422,
  "requestId": "req-123",
  "errors": [
    {
      "field": "email",
      "message": "Email address is invalid"
    }
  ]
}
```

A good error response should include:

* Machine-readable error type
* Human-readable message
* HTTP status
* Request identifier
* Validation details
* Optional remediation guidance

Do not expose:

* Stack traces
* Database names
* Internal hostnames
* Source-code locations
* Secrets
* Access tokens

---

## 15. Authentication and Authorization

REST APIs commonly use:

* Session cookies
* API keys
* OAuth 2.0
* Bearer tokens
* OpenID Connect
* Mutual TLS

Example:

```http
Authorization: Bearer <access-token>
```

Authentication answers:

```text
Who is the caller?
```

Authorization answers:

```text
What is the caller allowed to do?
```

Every protected resource should enforce authorization on the server.

---

## Architecture

A common production REST architecture looks like this:

```text
                         ┌────────────────────┐
                         │       Client       │
                         │ Web / Mobile / API │
                         └─────────┬──────────┘
                                   │
                                  HTTPS
                                   │
                         ┌─────────▼──────────┐
                         │    CDN / Edge      │
                         │ Caching / TLS      │
                         └─────────┬──────────┘
                                   │
                         ┌─────────▼──────────┐
                         │    WAF / DDoS      │
                         │    Protection      │
                         └─────────┬──────────┘
                                   │
                         ┌─────────▼──────────┐
                         │    API Gateway     │
                         │ Authentication     │
                         │ Rate Limiting      │
                         │ Routing            │
                         └─────────┬──────────┘
                                   │
                         ┌─────────▼──────────┐
                         │   Load Balancer    │
                         └─────────┬──────────┘
                                   │
                  ┌────────────────┼────────────────┐
                  │                │                │
         ┌────────▼───────┐ ┌──────▼────────┐ ┌─────▼──────────┐
         │ User Service   │ │ Order Service │ │ Product Service│
         │ REST API       │ │ REST API      │ │ REST API       │
         └────────┬───────┘ └──────┬────────┘ └────┬───────────┘
                  │                │                │
                  └────────────────┼────────────────┘
                                   │
                 ┌─────────────────┼─────────────────┐
                 │                 │                 │
         ┌───────▼──────┐  ┌───────▼─────┐  ┌────────▼────────┐
         │ Distributed  │  │ Database    │  │ Message Queue   │
         │ Cache        │  │ Cluster     │  │ Event Stream    │
         └──────────────┘  └─────────────┘  └─────────────────┘
```

### Request Flow

1. The client sends an HTTPS request.
2. The CDN returns a cached response when possible.
3. The WAF filters malicious traffic.
4. The API gateway authenticates and rate-limits the request.
5. The load balancer selects a healthy backend instance.
6. The service validates the request.
7. The service checks authorization.
8. The service reads or updates data.
9. Events may be published for asynchronous processing.
10. The response returns with an appropriate status code.
11. Logs, metrics, and traces are recorded.

---

## Layered Architecture

REST supports layered systems.

```text
Client
  |
  v
 CDN
  |
  v
API Gateway
  |
  v
Backend-for-Frontend
  |
  v
Domain Service
  |
  v
Database
```

The client does not need to know which layer processes the request.

Layers may provide:

* Caching
* Authentication
* Authorization
* Traffic routing
* Request transformation
* Protocol translation
* Logging
* Rate limiting
* Resilience

---

## Comparison

## REST vs SOAP

| Category           | REST                | SOAP                     |
| ------------------ | ------------------- | ------------------------ |
| Type               | Architectural style | Protocol                 |
| Common format      | JSON                | XML                      |
| Transport          | Usually HTTP        | HTTP and others          |
| Contract           | Optional OpenAPI    | WSDL                     |
| Message size       | Usually smaller     | Usually larger           |
| Caching            | Strong HTTP support | Less natural             |
| Complexity         | Lower               | Higher                   |
| Built-in standards | Limited             | Extensive WS-* standards |
| Best fit           | Web and mobile APIs | Enterprise integrations  |

SOAP may be useful where strict contracts, formal enterprise standards, or advanced message security are required.

---

## REST vs GraphQL

| Category       | REST                | GraphQL                     |
| -------------- | ------------------- | --------------------------- |
| API model      | Resources           | Query graph                 |
| Endpoints      | Multiple            | Usually one                 |
| Response shape | Server-defined      | Client-defined              |
| Caching        | Native HTTP caching | Often more complex          |
| Over-fetching  | Possible            | Reduced                     |
| Under-fetching | Possible            | Reduced                     |
| Learning curve | Lower               | Higher                      |
| File uploads   | Straightforward     | Requires conventions        |
| Error handling | HTTP status codes   | Often response-level errors |
| Best fit       | General APIs        | Complex frontend data needs |

REST is usually easier to cache and operate.

GraphQL is useful when clients need flexible combinations of related data.

---

## REST vs gRPC

| Category          | REST               | gRPC                          |
| ----------------- | ------------------ | ----------------------------- |
| Transport         | HTTP/1.1 or HTTP/2 | HTTP/2                        |
| Payload           | Usually JSON       | Protocol Buffers              |
| Human readability | High               | Low                           |
| Browser support   | Native             | Limited without adapters      |
| Performance       | Good               | Usually higher                |
| Streaming         | Limited or custom  | Built in                      |
| Contract          | Optional OpenAPI   | Required `.proto`             |
| Public APIs       | Common             | Less common                   |
| Internal services | Common             | Very common                   |
| Best fit          | External APIs      | High-performance internal RPC |

REST is often preferred for public APIs.

gRPC is often preferred for internal service-to-service communication where performance and strict contracts matter.

---

## REST vs WebSocket

| Category      | REST                   | WebSocket                |
| ------------- | ---------------------- | ------------------------ |
| Communication | Request-response       | Persistent bidirectional |
| Connection    | Short or reused HTTP   | Long-lived               |
| Server push   | Limited                | Native                   |
| Caching       | Supported              | Custom                   |
| Statelessness | Common                 | Stateful connection      |
| Best fit      | CRUD and standard APIs | Real-time communication  |

Use REST for:

* Resource retrieval
* CRUD operations
* Standard business workflows

Use WebSocket for:

* Live chat
* Multiplayer games
* Collaborative editing
* Real-time dashboards
* Continuous bidirectional updates

---

## REST vs RPC-Style APIs

| Category            | REST                   | RPC                   |
| ------------------- | ---------------------- | --------------------- |
| Primary abstraction | Resource               | Action                |
| Example             | `POST /orders`         | `POST /createOrder`   |
| HTTP semantics      | Strongly used          | Often limited         |
| Caching             | Natural for reads      | Less natural          |
| Discoverability     | Resource-oriented      | Operation-oriented    |
| Complex actions     | Sometimes awkward      | Natural               |
| Best fit            | Resource-based domains | Command-heavy systems |

Not every action maps cleanly to CRUD.

For domain actions, a REST API may model an operation as a subresource:

```http
POST /orders/123/cancellations
POST /users/123/password-resets
POST /accounts/123/transfers
```

---

## Real-World Example: E-Commerce Order API

Consider an e-commerce platform that allows customers to create and manage orders.

### Create an Order

```http
POST /api/v1/orders
Authorization: Bearer <token>
Content-Type: application/json
Idempotency-Key: checkout-session-789
X-Request-ID: req-456

{
  "customerId": "cust-123",
  "items": [
    {
      "productId": "prod-101",
      "quantity": 2
    }
  ],
  "shippingAddressId": "addr-555",
  "paymentMethodId": "pm-987"
}
```

### Request Processing

```text
Client
  |
  | POST /orders
  v
API Gateway
  |
  ├── Authenticate
  ├── Rate Limit
  └── Validate Headers
  |
  v
Order Service
  |
  ├── Check Idempotency Key
  ├── Validate Customer
  ├── Check Inventory
  ├── Calculate Total
  ├── Create Order
  ├── Reserve Inventory
  └── Publish OrderCreated Event
```

### Successful Response

```http
HTTP/1.1 201 Created
Location: /api/v1/orders/order-789
Content-Type: application/json
```

```json
{
  "id": "order-789",
  "status": "pending",
  "customerId": "cust-123",
  "items": [
    {
      "productId": "prod-101",
      "quantity": 2,
      "unitPrice": 49.99
    }
  ],
  "total": {
    "amount": 99.98,
    "currency": "USD"
  },
  "createdAt": "2026-07-25T10:30:00Z"
}
```

### Retrieve the Order

```http
GET /api/v1/orders/order-789
Authorization: Bearer <token>
```

```http
HTTP/1.1 200 OK
ETag: "order-789-v3"
Cache-Control: private, max-age=30
```

### Update the Shipping Address

```http
PATCH /api/v1/orders/order-789
Authorization: Bearer <token>
Content-Type: application/json
If-Match: "order-789-v3"

{
  "shippingAddressId": "addr-777"
}
```

`If-Match` helps prevent overwriting a newer version of the order.

Conflict response:

```http
HTTP/1.1 412 Precondition Failed
```

### Cancel the Order

A cancellation is a domain action.

It can be modeled as a subresource:

```http
POST /api/v1/orders/order-789/cancellations
Authorization: Bearer <token>
Content-Type: application/json

{
  "reason": "customer_request"
}
```

Response:

```http
HTTP/1.1 202 Accepted
Location: /api/v1/operations/op-456
```

The cancellation may continue asynchronously.

```text
OrderCancellationRequested
          |
          ├── Payment Service
          ├── Inventory Service
          ├── Notification Service
          └── Audit Service
```

### Validation Error

```http
HTTP/1.1 422 Unprocessable Content
Content-Type: application/problem+json
```

```json
{
  "type": "https://api.shop.example/problems/validation-error",
  "title": "Validation failed",
  "status": 422,
  "requestId": "req-456",
  "errors": [
    {
      "field": "items[0].quantity",
      "message": "Quantity must be greater than zero"
    }
  ]
}
```

### Inventory Conflict

```http
HTTP/1.1 409 Conflict
Content-Type: application/problem+json
```

```json
{
  "type": "https://api.shop.example/problems/insufficient-inventory",
  "title": "Insufficient inventory",
  "status": 409,
  "productId": "prod-101"
}
```

---

## Best Practices

## 1. Use Resource-Oriented URLs

Use nouns for resources.

Good:

```http
GET /users
GET /users/123
POST /orders
DELETE /orders/456
```

Avoid verbs when standard HTTP methods already express the action.

Poor:

```http
GET /getUsers
POST /createOrder
POST /deleteProduct
```

---

## 2. Use Correct HTTP Methods

Do not use `POST` for every operation.

Prefer:

```http
GET    /users/123
POST   /users
PUT    /users/123
PATCH  /users/123
DELETE /users/123
```

Correct methods improve:

* Caching
* Retry behavior
* Observability
* Client expectations
* Proxy behavior

---

## 3. Return Accurate Status Codes

Do not return `200 OK` for errors.

Poor:

```http
HTTP/1.1 200 OK

{
  "success": false,
  "error": "User not found"
}
```

Better:

```http
HTTP/1.1 404 Not Found
```

---

## 4. Keep Requests Stateless

Do not store request-specific conversational state in a single backend instance.

Use:

* Access tokens
* Distributed session stores
* Databases
* Caches
* Signed cookies

Stateless services are easier to scale and recover.

---

## 5. Design for Idempotency

Retries are normal in distributed systems.

Use idempotency keys for operations such as:

* Creating payments
* Creating orders
* Creating subscriptions
* Booking inventory
* Sending messages

```http
Idempotency-Key: order-create-123
```

---

## 6. Use Pagination for Collections

Never return unbounded collections.

```http
GET /api/v1/orders?limit=50&cursor=abc123
```

Set:

* Default page size
* Maximum page size
* Stable sort order
* Cursor expiration rules

---

## 7. Use Consistent Naming

Choose one naming convention and apply it everywhere.

Example:

```json
{
  "createdAt": "2026-07-25T10:30:00Z",
  "customerId": "cust-123"
}
```

Do not mix:

```json
{
  "created_at": "...",
  "customerId": "...",
  "OrderStatus": "..."
}
```

---

## 8. Validate All Input

Validate:

* Request body
* Path parameters
* Query parameters
* Headers
* File size
* Data types
* Enum values
* String lengths
* Date ranges

Reject unexpected fields when strict contracts are required.

---

## 9. Use a Standard Error Format

Return errors consistently.

```json
{
  "type": "https://api.example.com/problems/not-found",
  "title": "Resource not found",
  "status": 404,
  "requestId": "req-123"
}
```

Clients should not need custom parsing for every endpoint.

---

## 10. Use Timeouts

Every network call should have a timeout.

```text
Client Timeout:          10 seconds
Gateway Timeout:          9 seconds
Service Timeout:          7 seconds
Database Timeout:         3 seconds
```

Timeouts should become smaller deeper in the request path.

---

## 11. Retry Only Transient Failures

Possible retry candidates:

* Connection failures
* `429 Too Many Requests`
* `502 Bad Gateway`
* `503 Service Unavailable`
* `504 Gateway Timeout`

Do not retry:

* `400 Bad Request`
* `401 Unauthorized`
* `403 Forbidden`
* `404 Not Found`
* Validation failures

Use exponential backoff with jitter.

---

## 12. Protect Sensitive Data

Do not put secrets in URLs.

Avoid:

```http
GET /password-reset?token=secret-value
```

URLs may appear in:

* Browser history
* Proxy logs
* Analytics
* Monitoring systems
* Referrer headers

Use secure request bodies and redact sensitive fields from logs.

---

## 13. Implement Rate Limiting

Rate limiting protects against:

* Abuse
* Bots
* Brute-force attacks
* Accidental traffic spikes
* Resource exhaustion

Useful headers:

```http
RateLimit-Limit: 100
RateLimit-Remaining: 12
RateLimit-Reset: 30
Retry-After: 30
```

---

## 14. Use Conditional Requests

Conditional requests help prevent lost updates.

Example:

```http
GET /api/v1/products/123
```

Response:

```http
ETag: "product-123-v8"
```

Update:

```http
PATCH /api/v1/products/123
If-Match: "product-123-v8"
```

If the resource changed:

```http
HTTP/1.1 412 Precondition Failed
```

---

## 15. Cache Read-Heavy Resources

Cache suitable responses at:

* Client
* Browser
* CDN
* Reverse proxy
* Application cache

Do not cache sensitive or user-specific responses in shared caches without careful configuration.

---

## 16. Version Only for Breaking Changes

Do not create a new API version for every field addition.

Use a new version when changing:

* Field meaning
* Required fields
* Response structure
* Resource behavior
* Authentication model
* Error contract

---

## 17. Document the API Contract

Use an API specification such as OpenAPI.

Document:

* Endpoints
* Methods
* Headers
* Request schemas
* Response schemas
* Status codes
* Authentication
* Rate limits
* Pagination
* Error formats

Documentation should match production behavior.

---

## 18. Design for Observability

Use request and trace identifiers.

```http
X-Request-ID: req-123
Traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
```

Track:

* Request rate
* Error rate
* Latency percentiles
* Status-code distribution
* Response size
* Cache hit rate
* Timeout count
* Retry count
* Rate-limit rejections

---

## 19. Keep Responses Focused

Avoid returning every related object by default.

Use:

* Dedicated subresources
* Optional expansion
* Field selection
* Pagination

Example:

```http
GET /orders/123?expand=customer
```

Use expansion carefully to prevent expensive queries.

---

## 20. Secure Every Endpoint

Apply:

* HTTPS
* Authentication
* Authorization
* Input validation
* Output encoding
* Rate limiting
* Audit logging
* Secret redaction

Never trust the client to enforce authorization.

---

## Common Mistakes

## 1. Using Verbs in URLs

Poor:

```http
POST /createUser
POST /cancelOrder
```

Better:

```http
POST /users
POST /orders/123/cancellations
```

---

## 2. Returning `200 OK` for Every Result

This makes monitoring, caching, and client logic harder.

Use accurate status codes.

---

## 3. Confusing `PUT` and `PATCH`

`PUT` usually replaces a complete resource representation.

`PATCH` performs a partial update.

The API contract should clearly define behavior.

---

## 4. Using `GET` to Modify Data

Poor:

```http
GET /users/123/delete
GET /orders/456/cancel
```

`GET` may be cached, prefetched, repeated, or crawled.

Never use it for state-changing operations.

---

## 5. Ignoring Idempotency

A client may retry after a timeout even though the server completed the original request.

Without idempotency, the system may create:

* Duplicate payments
* Duplicate orders
* Duplicate subscriptions

---

## 6. Returning Unbounded Collections

Poor:

```http
GET /orders
```

A large dataset can exhaust:

* Memory
* Database connections
* Bandwidth
* Client resources

Always paginate.

---

## 7. Exposing Database Models Directly

Database entities often contain:

* Internal identifiers
* Sensitive fields
* Implementation-specific structure
* Relationships that should remain private

Use dedicated API models or data-transfer objects.

---

## 8. Breaking Clients with Response Changes

Renaming or removing fields can break existing clients.

Prefer backward-compatible evolution.

---

## 9. Inconsistent Error Formats

Clients should not receive one error shape from one endpoint and a different shape from another.

Standardize error responses across services.

---

## 10. Ignoring Authorization at Resource Level

Authentication alone is not enough.

A valid user must not automatically access every resource.

Always verify:

* Ownership
* Role
* Tenant
* Organization
* Scope
* Permission

---

## 11. Putting Sensitive Data in Query Parameters

Query parameters may be stored in logs and analytics systems.

Do not include:

* Passwords
* Access tokens
* API keys
* Session identifiers
* Personal secrets

---

## 12. Over-Nesting URLs

Poor:

```http
/companies/1/departments/2/teams/3/users/4/tasks/5
```

Deep nesting creates long, tightly coupled URLs.

Prefer direct resource access when the identifier is globally unique.

```http
/tasks/5
```

---

## 13. Ignoring Partial Failures

A REST request may depend on several downstream services.

One failure can cause:

* Timeout
* Partial update
* Duplicate retry
* Inconsistent state

Use:

* Transactions where appropriate
* Idempotency
* Compensating actions
* Asynchronous workflows
* Clear operation status

---

## 14. Overusing Synchronous Processing

Long operations should not keep HTTP requests open unnecessarily.

For long-running tasks:

```http
POST /reports
```

Return:

```http
HTTP/1.1 202 Accepted
Location: /operations/op-123
```

The client can check:

```http
GET /operations/op-123
```

---

## 15. Treating REST as CRUD Only

REST can model workflows and actions using resources.

Examples:

```http
POST /orders/123/cancellations
POST /accounts/123/transfers
POST /users/123/password-resets
```

The goal is a clear resource model, not forcing every business operation into a database CRUD shape.

---

## Interview Questions

### 1. What is REST?

REST is an architectural style for distributed systems that models data as resources and uses a uniform, stateless interface, commonly over HTTP.

---

### 2. What does statelessness mean in REST?

Each request contains all information required to process it. A backend instance should not depend on local state from earlier requests by the same client.

---

### 3. What is the difference between `PUT` and `PATCH`?

`PUT` usually replaces the full resource representation, while `PATCH` updates only selected fields. `PUT` is idempotent; `PATCH` may or may not be.

---

### 4. Why is idempotency important in REST APIs?

Clients and infrastructure may retry requests after timeouts or connection failures. Idempotency prevents retries from creating unintended duplicate effects.

---

### 5. When should REST not be used?

REST may not be ideal for high-performance binary RPC, complex client-selected graph queries, or persistent real-time bidirectional communication. gRPC, GraphQL, or WebSocket may fit those cases better.

---

## Key Takeaways

1. **REST organizes APIs around resources, standard HTTP methods, meaningful status codes, and stateless request handling.**

2. **Production-ready REST APIs require pagination, idempotency, caching, validation, authorization, rate limiting, observability, and consistent error contracts.**

3. **REST is a strong default for public and general-purpose APIs, but protocol choice should depend on the communication pattern, performance requirements, and client needs.**
