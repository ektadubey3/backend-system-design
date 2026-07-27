# gRPC

gRPC is a high-performance remote procedure call framework designed for communication between distributed services.

It allows one application to call a method on another application as though it were a local function.

```text
Client Service
      |
      | Remote Procedure Call
      v
Server Service
```

A service contract is defined using Protocol Buffers.

```proto
syntax = "proto3";

service UserService {
  rpc GetUser(GetUserRequest) returns (GetUserResponse);
}

message GetUserRequest {
  string user_id = 1;
}

message GetUserResponse {
  string user_id = 1;
  string name = 2;
  string email = 3;
}
```

The Protocol Buffer compiler generates strongly typed client and server code.

```text
user.proto
    |
    v
Protocol Buffer Compiler
    |
    ├── Client Stub
    ├── Server Interface
    ├── Request Types
    └── Response Types
```

gRPC commonly uses:

* HTTP/2
* Protocol Buffers
* Strongly typed service contracts
* Generated client and server code
* Binary serialization
* Streaming
* Deadlines
* Cancellation
* Metadata
* Standardized status codes

gRPC is frequently used for:

* Microservice communication
* Internal backend APIs
* Low-latency systems
* Polyglot service architectures
* Real-time streaming
* Mobile-to-backend communication
* Infrastructure control planes
* Machine learning serving
* Financial systems
* Distributed data platforms

gRPC is most commonly used for internal service-to-service communication, although it can also support external clients with the correct infrastructure.

---

## Why gRPC?

gRPC is useful when backend systems require strict contracts, efficient serialization, low latency, and streaming communication.

### 1. High Performance

gRPC uses Protocol Buffers, a compact binary serialization format.

Example JSON payload:

```json
{
  "userId": "user-123",
  "name": "Alex Morgan",
  "active": true
}
```

A Protocol Buffer representation is encoded in a smaller binary format.

Benefits include:

* Smaller payloads
* Faster serialization
* Faster deserialization
* Reduced network bandwidth
* Lower CPU overhead for many workloads

Actual performance depends on:

* Message size
* Network latency
* Serialization implementation
* Compression settings
* Language runtime
* Connection reuse
* Server workload

---

### 2. Strong API Contracts

A `.proto` file defines the API contract.

```proto
service OrderService {
  rpc CreateOrder(CreateOrderRequest) returns (CreateOrderResponse);
  rpc GetOrder(GetOrderRequest) returns (GetOrderResponse);
}
```

The contract specifies:

* Service names
* Method names
* Request types
* Response types
* Field types
* Field identifiers
* Streaming behavior

This reduces ambiguity between clients and servers.

---

### 3. Automatic Code Generation

gRPC generates client and server code from the service definition.

Supported languages commonly include:

* Java
* Go
* C#
* Python
* C++
* JavaScript
* TypeScript
* Kotlin
* Swift
* Ruby
* PHP
* Rust through community libraries

Generated code provides:

* Client stubs
* Server interfaces
* Serialization logic
* Type definitions
* Method signatures

This reduces repetitive networking code.

---

### 4. Efficient Multiplexing

gRPC commonly runs over HTTP/2.

HTTP/2 allows multiple requests to share a single connection.

```text
Single HTTP/2 Connection
         |
         ├── Stream 1: GetUser
         ├── Stream 3: CreateOrder
         ├── Stream 5: GetInventory
         └── Stream 7: UpdatePayment
```

This reduces:

* Connection establishment
* TLS handshakes
* Socket usage
* Network overhead
* Connection-pool pressure

---

### 5. Native Streaming

gRPC supports four communication patterns:

* Unary RPC
* Server streaming
* Client streaming
* Bidirectional streaming

This makes gRPC suitable for systems that need more than standard request-response communication.

---

### 6. Deadline and Cancellation Support

Clients can specify how long they are willing to wait.

```text
Client Deadline: 500 ms
```

When the deadline expires:

* The client stops waiting.
* The RPC is cancelled.
* The server can stop unnecessary work.
* Downstream calls can inherit the remaining deadline.

This is important for controlling latency in distributed systems.

---

### 7. Polyglot Communication

A service can be implemented in one language and consumed by clients written in another.

```text
Go Client
    |
    | gRPC
    v
Java Server
```

Both sides use code generated from the same `.proto` contract.

---

### 8. Suitable for Internal Microservices

Internal services often require:

* Low latency
* High throughput
* Strong typing
* Reliable contracts
* Streaming
* Controlled client generation

gRPC addresses these requirements well.

---

## Core Concepts

## 1. Protocol Buffers

Protocol Buffers, commonly called Protobuf, are gRPC's default interface definition and serialization format.

Example:

```proto
syntax = "proto3";

package users.v1;

message User {
  string id = 1;
  string name = 2;
  string email = 3;
  bool active = 4;
}
```

Each field has:

* A name
* A type
* A numeric field identifier

```proto
string email = 3;
```

The field number is part of the serialized format and must remain stable.

---

## 2. Service Definitions

A gRPC service contains one or more RPC methods.

```proto
service ProductService {
  rpc GetProduct(GetProductRequest) returns (GetProductResponse);
  rpc ListProducts(ListProductsRequest) returns (ListProductsResponse);
}
```

A method defines:

```text
Method Name
Request Type
Response Type
Streaming Mode
```

---

## 3. Unary RPC

Unary RPC is the standard request-response pattern.

```proto
rpc GetUser(GetUserRequest) returns (GetUserResponse);
```

Flow:

```text
Client
  |
  | One Request
  v
Server
  |
  | One Response
  v
Client
```

Example use cases:

* Retrieve a user
* Create an order
* Update a payment
* Validate a token
* Fetch inventory

---

## 4. Server Streaming

The client sends one request, and the server returns a stream of responses.

```proto
rpc StreamOrderEvents(OrderEventsRequest)
    returns (stream OrderEvent);
```

Flow:

```text
Client
  |
  | One Request
  v
Server
  |
  ├── Response 1
  ├── Response 2
  ├── Response 3
  └── Response N
```

Use cases:

* Activity feeds
* Log streaming
* Large result sets
* Notifications
* Progress updates
* Market data

---

## 5. Client Streaming

The client sends multiple messages, and the server returns one response.

```proto
rpc UploadMetrics(stream Metric)
    returns (UploadMetricsResponse);
```

Flow:

```text
Client
  |
  ├── Message 1
  ├── Message 2
  ├── Message 3
  └── Message N
  |
  v
Server
  |
  | One Response
  v
Client
```

Use cases:

* Telemetry upload
* Sensor data
* Batch ingestion
* File chunk upload
* Event aggregation

---

## 6. Bidirectional Streaming

Both client and server send independent message streams.

```proto
rpc Chat(stream ChatMessage)
    returns (stream ChatMessage);
```

Flow:

```text
Client                     Server
  |------ Message 1 -------->|
  |<----- Message A ---------|
  |------ Message 2 -------->|
  |------ Message 3 -------->|
  |<----- Message B ---------|
```

Use cases:

* Live chat
* Multiplayer games
* Collaborative editing
* Device control
* Real-time trading
* Interactive workflows

Message ordering is preserved within each individual stream.

---

## 7. Client Stubs

A client stub is generated code that represents the remote service.

Example pseudocode:

```go
client := NewUserServiceClient(connection)

response, err := client.GetUser(
    context,
    &GetUserRequest{
        UserId: "user-123",
    },
)
```

The stub handles:

* Serialization
* HTTP/2 communication
* Method routing
* Response deserialization
* Status propagation
* Metadata transmission

---

## 8. Server Implementations

The server implements the generated service interface.

Example pseudocode:

```go
type UserServer struct {
    userRepository UserRepository
}

func (s *UserServer) GetUser(
    ctx context.Context,
    request *GetUserRequest,
) (*GetUserResponse, error) {
    user, err := s.userRepository.FindByID(
        ctx,
        request.UserId,
    )

    if err != nil {
        return nil, err
    }

    return &GetUserResponse{
        UserId: user.ID,
        Name:   user.Name,
        Email:  user.Email,
    }, nil
}
```

The generated interface guarantees that the implementation matches the contract.

---

## 9. HTTP/2 Transport

gRPC commonly uses HTTP/2 as its transport layer.

HTTP/2 provides:

* Multiplexed streams
* Binary framing
* Header compression
* Persistent connections
* Bidirectional streams
* Flow control

Typical protocol stack:

```text
Application
    |
    v
  gRPC
    |
    v
  HTTP/2
    |
    v
   TLS
    |
    v
   TCP
```

In trusted internal networks, plaintext HTTP/2 may sometimes be used, but encrypted communication is preferred for production environments.

---

## 10. Metadata

Metadata carries information associated with an RPC.

It is similar to HTTP headers.

Example metadata:

```text
authorization: Bearer <token>
x-request-id: req-123
x-tenant-id: tenant-456
traceparent: 00-...
```

Metadata may be sent:

* From client to server
* From server to client as headers
* From server to client as trailers

Use metadata for cross-cutting information, not normal business data.

---

## 11. Deadlines

A deadline defines the maximum time available for an RPC.

Example:

```text
Current Time: 10:00:00.000
Deadline:     10:00:00.500
Budget:       500 ms
```

Client pseudocode:

```go
ctx, cancel := context.WithTimeout(
    context.Background(),
    500*time.Millisecond,
)
defer cancel()

response, err := client.GetUser(ctx, request)
```

When the deadline is exceeded, gRPC commonly returns:

```text
DEADLINE_EXCEEDED
```

Every production RPC should have a deadline.

---

## 12. Deadline Propagation

In a distributed call chain, the remaining deadline should propagate to downstream services.

```text
Client Deadline: 1000 ms
        |
        v
API Service: 850 ms remaining
        |
        v
Order Service: 600 ms remaining
        |
        v
Payment Service: 300 ms remaining
```

Without propagation, downstream services may continue working after the original client has stopped waiting.

---

## 13. Cancellation

Clients may cancel RPCs.

Cancellation should stop:

* Database queries
* External API calls
* Expensive computation
* Stream processing
* Downstream RPCs

Server code should regularly check whether the request context has been cancelled.

---

## 14. gRPC Status Codes

gRPC uses standardized status codes.

| Code                  | Meaning                               |
| --------------------- | ------------------------------------- |
| `OK`                  | Operation completed successfully      |
| `CANCELLED`           | Operation was cancelled               |
| `UNKNOWN`             | Unknown error                         |
| `INVALID_ARGUMENT`    | Invalid client input                  |
| `DEADLINE_EXCEEDED`   | Deadline expired                      |
| `NOT_FOUND`           | Resource was not found                |
| `ALREADY_EXISTS`      | Resource already exists               |
| `PERMISSION_DENIED`   | Caller lacks permission               |
| `RESOURCE_EXHAUSTED`  | Quota or capacity exceeded            |
| `FAILED_PRECONDITION` | System state prevents operation       |
| `ABORTED`             | Operation was aborted due to conflict |
| `OUT_OF_RANGE`        | Value is outside a valid range        |
| `UNIMPLEMENTED`       | Method is not implemented             |
| `INTERNAL`            | Internal server failure               |
| `UNAVAILABLE`         | Service is temporarily unavailable    |
| `DATA_LOSS`           | Unrecoverable data corruption         |
| `UNAUTHENTICATED`     | Authentication is missing or invalid  |

Use the most specific status code available.

---

## 15. Rich Error Details

A status code alone may not provide enough information.

Structured error details can include:

* Field violations
* Retry information
* Resource identifiers
* Quota failures
* Precondition failures
* Localized messages

Conceptual example:

```text
Status:
INVALID_ARGUMENT

Details:
- Field: email
- Description: Invalid email format
```

Clients should rely on stable machine-readable error details rather than parsing human-readable messages.

---

## 16. Interceptors

Interceptors add shared behavior around RPC execution.

They are similar to middleware.

Common interceptor use cases:

* Authentication
* Authorization
* Logging
* Metrics
* Distributed tracing
* Rate limiting
* Retry policies
* Error mapping
* Request validation

Flow:

```text
Request
  |
  v
Authentication Interceptor
  |
  v
Tracing Interceptor
  |
  v
Metrics Interceptor
  |
  v
Service Handler
```

Interceptors may exist on both clients and servers.

---

## 17. Channel

A channel represents a connection-oriented abstraction to a gRPC server.

```text
Client
  |
  v
gRPC Channel
  |
  ├── RPC Stream 1
  ├── RPC Stream 2
  ├── RPC Stream 3
  └── RPC Stream N
```

Channels should usually be:

* Long-lived
* Reused
* Shared safely where supported
* Configured with connection management
* Monitored

Creating a new channel for every request removes many gRPC performance benefits.

---

## 18. Flow Control

Flow control prevents a fast sender from overwhelming a slower receiver.

It operates at:

* HTTP/2 connection level
* HTTP/2 stream level
* Application processing level

Streaming handlers should process messages at a sustainable rate.

Do not read an entire unbounded stream into memory.

---

## 19. Backpressure

Backpressure allows receivers to limit how quickly senders produce data.

Without backpressure:

```text
Producer Rate: 10,000 messages/second
Consumer Rate: 1,000 messages/second
```

Possible consequences:

* Memory growth
* Queue accumulation
* Increased latency
* Process crashes
* Dropped messages

Applications should combine transport flow control with bounded application buffers.

---

## 20. Load Balancing

gRPC connections are long-lived and multiplex many requests.

Traditional request-level load balancing may not distribute traffic evenly if clients maintain only one connection.

Common approaches include:

### Proxy-Side Load Balancing

```text
Client
  |
  v
Load Balancer
  |
  ├── Service Instance 1
  ├── Service Instance 2
  └── Service Instance 3
```

The proxy understands HTTP/2 and distributes streams or connections.

### Client-Side Load Balancing

```text
Client
  |
  ├── Instance 1
  ├── Instance 2
  └── Instance 3
```

The client receives service endpoints and selects an instance for each call.

---

## 21. Service Discovery

Client-side balancing requires service discovery.

Possible sources include:

* DNS
* Kubernetes services
* Service registries
* Control planes
* Service mesh APIs

```text
Client
  |
  | Discover Instances
  v
Service Registry
  |
  ├── 10.0.1.10
  ├── 10.0.1.11
  └── 10.0.1.12
```

---

## 22. Health Checking

gRPC services should expose health information.

Health status may include:

* `SERVING`
* `NOT_SERVING`
* `UNKNOWN`
* `SERVICE_UNKNOWN`

Load balancers and orchestrators can use health checks to remove unhealthy instances.

Health checks should distinguish between:

* Process alive
* Service ready
* Dependencies available
* Traffic acceptance state

---

## 23. Reflection

Server reflection allows tools to inspect available gRPC services at runtime.

It can support:

* Interactive debugging
* Command-line testing
* Service discovery
* API exploration

Reflection should be controlled carefully in production environments.

It is not a substitute for authentication or authorization.

---

## 24. Compression

gRPC can compress messages.

Compression may reduce network usage for large payloads but increases CPU cost.

Consider compression for:

* Large text payloads
* Repetitive data
* High-latency networks
* Bandwidth-constrained clients

Avoid unnecessary compression for:

* Tiny messages
* Already compressed data
* CPU-sensitive workloads

Measure before enabling it broadly.

---

## 25. Keepalive

Keepalive can detect broken connections and preserve long-lived channels.

However, overly aggressive keepalive settings may:

* Increase network traffic
* Overload proxies
* Trigger server enforcement limits
* Drain mobile batteries
* Cause connection churn

Coordinate keepalive settings across clients, servers, and load balancers.

---

## Architecture

A production gRPC architecture may look like this:

```text
                         ┌─────────────────────┐
                         │       Client        │
                         │ Service / Mobile /  │
                         │ Backend Application │
                         └──────────┬──────────┘
                                    │
                              gRPC over TLS
                                    │
                         ┌──────────▼──────────┐
                         │   Load Balancer /   │
                         │   Service Mesh      │
                         │ HTTP/2 Routing      │
                         └──────────┬──────────┘
                                    │
                         ┌──────────▼──────────┐
                         │   gRPC API Layer    │
                         │ Authentication      │
                         │ Validation          │
                         │ Rate Limiting       │
                         └──────────┬──────────┘
                                    │
                  ┌─────────────────┼─────────────────┐
                  │                 │                 │
         ┌────────▼────────┐ ┌──────▼────────┐ ┌──────▼──────────┐
         │ User Service    │ │ Order Service │ │ Product Service │
         │ gRPC            │ │ gRPC          │ │ gRPC            │
         └────────┬────────┘ └──────┬────────┘ └──────┬──────────┘
                  │                 │                 │
                  └─────────────────┼─────────────────┘
                                    │
                 ┌──────────────────┼──────────────────┐
                 │                  │                  │
         ┌───────▼───────┐  ┌───────▼──────┐  ┌────────▼────────┐
         │ Distributed   │  │ Database     │  │ Event Stream    │
         │ Cache         │  │ Cluster      │  │ Message Queue   │
         └───────────────┘  └──────────────┘  └─────────────────┘
```

### Request Flow

1. The client obtains an endpoint through service discovery.
2. The client reuses an existing gRPC channel.
3. The client serializes the request using Protocol Buffers.
4. Metadata is attached to the RPC.
5. The request is sent over an HTTP/2 stream.
6. The load balancer selects a healthy service instance.
7. Server interceptors process authentication, tracing, and validation.
8. The service handler executes domain logic.
9. The service calls databases, caches, or downstream services.
10. The response is serialized and returned.
11. Metrics, traces, and logs record the result.
12. The client receives either a typed response or a gRPC status.

---

## Direct Service-to-Service Architecture

A simple internal architecture may allow services to call each other directly.

```text
Order Service
     |
     ├── gRPC -> User Service
     ├── gRPC -> Inventory Service
     └── gRPC -> Payment Service
```

Advantages:

* Low latency
* Simple data path
* Fewer proxy hops
* Direct service ownership

Risks:

* Complex service discovery
* Client configuration duplication
* Tight runtime dependencies
* Difficult network policy management
* Retry amplification
* Distributed failure propagation

---

## Service Mesh Architecture

A service mesh places a proxy beside each application service.

```text
┌──────────────────────────────┐
│ Order Service Pod            │
│                              │
│  Order Application           │
│          |                   │
│     Sidecar Proxy            │
└──────────┬───────────────────┘
           |
           | gRPC with mTLS
           |
┌──────────▼───────────────────┐
│ Inventory Service Pod        │
│                              │
│     Sidecar Proxy            │
│          |                   │
│  Inventory Application       │
└──────────────────────────────┘
```

A service mesh may provide:

* Mutual TLS
* Traffic routing
* Load balancing
* Retries
* Circuit breaking
* Metrics
* Distributed tracing
* Access control

A mesh adds operational complexity and does not replace application-level resilience.

---

## External Client Architecture

Browsers cannot always call standard gRPC services directly.

A common architecture uses gRPC-Web or protocol translation.

```text
Browser
   |
   | gRPC-Web
   v
Edge Proxy
   |
   | Standard gRPC
   v
Backend Service
```

Another approach exposes REST externally and gRPC internally.

```text
Web / Mobile Client
        |
        | REST or GraphQL
        v
API Gateway
        |
        | gRPC
        v
Internal Services
```

This separates public API requirements from internal service communication.

---

## Comparison

## gRPC vs REST

| Category            | gRPC                       | REST                           |
| ------------------- | -------------------------- | ------------------------------ |
| Communication model | Remote procedures          | Resources                      |
| Common transport    | HTTP/2                     | HTTP/1.1, HTTP/2, or HTTP/3    |
| Payload format      | Protocol Buffers           | Usually JSON                   |
| Serialization       | Binary                     | Usually text                   |
| Contract            | Required `.proto` file     | Optional OpenAPI specification |
| Code generation     | Built in                   | Optional                       |
| Human readability   | Low                        | High                           |
| Browser support     | Limited without gRPC-Web   | Native                         |
| Streaming           | Native                     | Requires additional patterns   |
| Caching             | Less natural               | Strong HTTP caching            |
| Performance         | Usually higher             | Usually sufficient             |
| Public APIs         | Less common                | Very common                    |
| Internal services   | Excellent fit              | Common                         |
| Debugging           | Requires specialized tools | Easy with standard HTTP tools  |

Use gRPC when performance, streaming, and strong contracts matter.

Use REST when browser compatibility, human-readable payloads, and standard HTTP caching are priorities.

---

## gRPC vs GraphQL

| Category               | gRPC                      | GraphQL                         |
| ---------------------- | ------------------------- | ------------------------------- |
| Primary use            | Service-to-service RPC    | Flexible client-facing data API |
| API model              | Procedures                | Typed data graph                |
| Payload                | Protocol Buffers          | Usually JSON                    |
| Client field selection | No                        | Yes                             |
| Streaming              | Native                    | Subscriptions                   |
| Browser support        | Limited                   | Strong                          |
| Performance            | High                      | Moderate                        |
| Schema                 | `.proto`                  | GraphQL SDL                     |
| Best fit               | Internal backend services | Web and mobile applications     |

A common system uses both:

```text
Web Client
    |
    | GraphQL
    v
GraphQL Gateway
    |
    | gRPC
    v
Backend Services
```

---

## gRPC vs Message Queue

| Category                 | gRPC                    | Message Queue             |
| ------------------------ | ----------------------- | ------------------------- |
| Communication            | Usually synchronous     | Usually asynchronous      |
| Response expected        | Yes                     | Not always                |
| Coupling                 | Temporal coupling       | Reduced temporal coupling |
| Latency                  | Low                     | Usually higher            |
| Delivery                 | Request-response        | Broker-based delivery     |
| Offline consumer support | No                      | Yes                       |
| Replay                   | Not native              | Often supported           |
| Best fit                 | Immediate service calls | Event-driven workflows    |

Use gRPC when the caller needs an immediate response.

Use a message queue when work can be asynchronous or consumers may be temporarily unavailable.

---

## gRPC vs WebSocket

| Category        | gRPC                        | WebSocket                      |
| --------------- | --------------------------- | ------------------------------ |
| API contract    | Strongly typed              | Application-defined            |
| Communication   | RPC and streaming           | Bidirectional messages         |
| Serialization   | Protocol Buffers by default | Flexible                       |
| Multiplexing    | Built into HTTP/2           | Application-managed            |
| Browser support | Limited                     | Strong                         |
| Best fit        | Typed backend communication | Browser real-time applications |

Bidirectional gRPC streaming is well suited to service-to-service communication.

WebSocket is often easier for browser-based real-time features.

---

## gRPC vs SOAP

| Category             | gRPC                       | SOAP                          |
| -------------------- | -------------------------- | ----------------------------- |
| Message format       | Protocol Buffers           | XML                           |
| Transport            | Primarily HTTP/2           | HTTP and others               |
| Contract             | `.proto`                   | WSDL                          |
| Payload size         | Small                      | Larger                        |
| Performance          | High                       | Lower in many workloads       |
| Streaming            | Native                     | Limited                       |
| Enterprise standards | Smaller ecosystem          | Extensive WS-* standards      |
| Best fit             | Modern distributed systems | Legacy enterprise integration |

---

## gRPC vs Custom TCP Protocol

| Category         | gRPC              | Custom TCP          |
| ---------------- | ----------------- | ------------------- |
| Framing          | Provided          | Must be designed    |
| Serialization    | Protocol Buffers  | Must be selected    |
| Streaming        | Built in          | Must be implemented |
| Deadlines        | Built in          | Must be implemented |
| Status codes     | Standardized      | Custom              |
| Load balancing   | Ecosystem support | Custom              |
| Tooling          | Mature            | Custom              |
| Flexibility      | Moderate          | Maximum             |
| Development cost | Lower             | Higher              |

A custom protocol may offer specialized performance, but it significantly increases implementation and operational complexity.

---

## Real-World Example: E-Commerce Checkout

Consider an e-commerce checkout service that needs to:

* Validate the customer
* Verify inventory
* calculate prices
* authorize payment
* create an order
* publish an event

### Service Contract

```proto
syntax = "proto3";

package checkout.v1;

service CheckoutService {
  rpc PlaceOrder(PlaceOrderRequest)
      returns (PlaceOrderResponse);
}

message PlaceOrderRequest {
  string customer_id = 1;
  repeated OrderItem items = 2;
  string payment_method_id = 3;
  string idempotency_key = 4;
}

message OrderItem {
  string product_id = 1;
  int32 quantity = 2;
}

message PlaceOrderResponse {
  Order order = 1;
}

message Order {
  string order_id = 1;
  OrderStatus status = 2;
  Money total = 3;
}

message Money {
  int64 amount_cents = 1;
  string currency = 2;
}

enum OrderStatus {
  ORDER_STATUS_UNSPECIFIED = 0;
  ORDER_STATUS_PENDING = 1;
  ORDER_STATUS_CONFIRMED = 2;
  ORDER_STATUS_FAILED = 3;
}
```

### Request Flow

```text
Checkout Client
      |
      | PlaceOrder
      v
Checkout Service
      |
      ├── GetCustomer -> Customer Service
      ├── ReserveItems -> Inventory Service
      ├── CalculatePrice -> Pricing Service
      ├── AuthorizePayment -> Payment Service
      ├── CreateOrder -> Order Database
      └── Publish OrderCreated Event
```

### Detailed Architecture

```text
                         ┌─────────────────────┐
                         │   Checkout Client   │
                         └──────────┬──────────┘
                                    │
                         PlaceOrder RPC
                         Deadline: 2 seconds
                                    │
                         ┌──────────▼──────────┐
                         │  Checkout Service   │
                         │ Idempotency Check   │
                         └──────────┬──────────┘
                                    │
          ┌─────────────────────────┼─────────────────────────┐
          │                         │                         │
 ┌────────▼─────────┐     ┌─────────▼────────┐      ┌─────────▼───────┐
 │ Customer Service │     │ Inventory Service│      │ Pricing Service │
 │ GetCustomer      │     │ ReserveInventory │      │ CalculateTotal  │
 └──────────────────┘     └──────────────────┘      └─────────────────┘
                                    │
                         ┌──────────▼──────────┐
                         │  Payment Service    │
                         │ AuthorizePayment    │
                         └──────────┬──────────┘
                                    │
                         ┌──────────▼──────────┐
                         │   Order Database    │
                         └──────────┬──────────┘
                                    │
                         ┌──────────▼──────────┐
                         │    Event Stream     │
                         │ OrderCreated Event  │
                         └─────────────────────┘
```

### Client Call

Pseudocode:

```go
ctx, cancel := context.WithTimeout(
    context.Background(),
    2*time.Second,
)
defer cancel()

response, err := checkoutClient.PlaceOrder(
    ctx,
    &PlaceOrderRequest{
        CustomerId:     "cust-123",
        PaymentMethodId: "pm-456",
        IdempotencyKey:  "checkout-789",
        Items: []*OrderItem{
            {
                ProductId: "prod-101",
                Quantity:  2,
            },
        },
    },
)
```

### Successful Response

Conceptual response:

```text
order_id: "order-789"
status: ORDER_STATUS_CONFIRMED
total:
  amount_cents: 19998
  currency: "USD"
```

This represents a total of `$199.98`.

### Validation Error

Invalid quantity:

```text
Status:
INVALID_ARGUMENT

Details:
- Field: items[0].quantity
- Message: Quantity must be greater than zero
```

### Inventory Conflict

If inventory is unavailable:

```text
Status:
FAILED_PRECONDITION

Details:
- Product: prod-101
- Reason: INSUFFICIENT_INVENTORY
```

### Temporary Payment Failure

If the payment service is unavailable:

```text
Status:
UNAVAILABLE
```

The client may retry only when:

* The request uses an idempotency key.
* The retry budget allows another attempt.
* The deadline has not expired.
* The error is considered transient.

### Deadline Propagation

```text
Checkout Client Deadline:          2000 ms
Checkout Service Processing:       150 ms
Remaining for Inventory:           1850 ms
Remaining for Pricing:             1500 ms
Remaining for Payment:              900 ms
Remaining for Order Creation:       350 ms
```

Every downstream request should receive only the remaining budget.

### Asynchronous Work

Non-critical operations should not delay checkout completion.

```text
Order Created
    |
    v
Event Stream
    |
    ├── Email Service
    ├── Analytics Service
    ├── Recommendation Service
    └── Audit Service
```

The checkout service returns after the critical transaction succeeds.

---

## Best Practices

## 1. Design APIs Around Business Capabilities

Use meaningful service and method names.

Good:

```proto
service PaymentService {
  rpc AuthorizePayment(AuthorizePaymentRequest)
      returns (AuthorizePaymentResponse);

  rpc CapturePayment(CapturePaymentRequest)
      returns (CapturePaymentResponse);
}
```

Avoid generic services:

```proto
service DataService {
  rpc Execute(Command) returns (Result);
}
```

The service contract should clearly express domain behavior.

---

## 2. Always Set Deadlines

Never allow RPCs to wait indefinitely.

```text
User Lookup:       200 ms
Inventory Check:   400 ms
Payment Request:  1000 ms
Report Generation: 10 s
```

Deadline values should be based on:

* Service-level objectives
* Observed latency
* Retry policy
* Downstream dependencies
* User experience

---

## 3. Propagate Deadlines and Cancellation

A downstream service should not receive a fresh timeout that exceeds the caller's remaining budget.

```text
Original Deadline: 1000 ms
Elapsed Time:       400 ms
Remaining Budget:   600 ms
```

Propagate the remaining deadline and cancellation context.

---

## 4. Reuse Channels

Create long-lived channels and reuse them.

Poor:

```text
Request
  |
Create Channel
  |
Call RPC
  |
Close Channel
```

Better:

```text
Application
  |
Long-Lived Channel
  |
  ├── RPC 1
  ├── RPC 2
  ├── RPC 3
  └── RPC N
```

Frequent channel creation causes:

* Extra TLS handshakes
* Higher latency
* More CPU usage
* Socket churn
* Load balancer pressure

---

## 5. Design Protobuf Fields for Compatibility

Never change an existing field number.

Poor:

```proto
string name = 1;
```

Later changed to:

```proto
string email = 1;
```

Old clients may interpret email data as a name.

When removing a field, reserve its number and name.

```proto
message User {
  reserved 3;
  reserved "legacy_username";

  string id = 1;
  string name = 2;
}
```

---

## 6. Never Reuse Removed Field Numbers

Field numbers are part of the binary wire format.

Reusing them can cause silent data corruption or incorrect decoding.

Always mark removed numbers as reserved.

---

## 7. Add Fields Instead of Renaming Them

Changing field names does not affect the binary format, but it can break generated code or JSON representations.

For major semantic changes:

1. Add a new field.
2. Deprecate the old field.
3. Migrate clients.
4. Reserve the old field after removal.

---

## 8. Use Enums Safely

Always include an unspecified zero value.

Good:

```proto
enum PaymentStatus {
  PAYMENT_STATUS_UNSPECIFIED = 0;
  PAYMENT_STATUS_PENDING = 1;
  PAYMENT_STATUS_AUTHORIZED = 2;
  PAYMENT_STATUS_FAILED = 3;
}
```

This handles:

* Missing values
* Unknown defaults
* Invalid mappings
* Future compatibility

Clients should tolerate enum values introduced by newer servers.

---

## 9. Keep Messages Focused

Avoid creating one massive request or response type for many unrelated operations.

Poor:

```proto
message UniversalRequest {
  string user_id = 1;
  string order_id = 2;
  string product_id = 3;
  string payment_id = 4;
  string action = 5;
}
```

Prefer operation-specific messages.

```proto
GetUserRequest
CreateOrderRequest
AuthorizePaymentRequest
```

---

## 10. Avoid Excessively Large Messages

Large messages can cause:

* Memory spikes
* Longer garbage-collection pauses
* Increased latency
* Proxy limits
* Slow retries
* Flow-control pressure

For large datasets, use:

* Pagination
* Streaming
* Object storage
* Chunking
* Batch limits

Do not send multi-gigabyte files as a single unary message.

---

## 11. Use Streaming Only When Needed

Streaming adds complexity related to:

* Connection lifetime
* Backpressure
* Error handling
* Reconnection
* Authentication refresh
* Load balancing
* Observability

Use unary RPC for simple request-response operations.

---

## 12. Implement Backpressure

For streaming services:

* Use bounded buffers.
* Process messages incrementally.
* Limit concurrent streams.
* Reject overloaded clients.
* Monitor queue depth.
* Respect flow control.

Do not accumulate an unlimited stream in memory.

---

## 13. Use Idempotency for Retried Writes

gRPC clients and proxies may retry transient failures.

For write operations, include an idempotency key.

```proto
message CreateOrderRequest {
  string idempotency_key = 1;
  string customer_id = 2;
}
```

The server should store or recognize completed requests associated with the key.

---

## 14. Retry Only Safe Operations

Suitable retry candidates may include:

* Idempotent reads
* Idempotent writes
* Requests protected by idempotency keys
* Temporary `UNAVAILABLE` failures
* Some `RESOURCE_EXHAUSTED` responses

Do not retry automatically:

* `INVALID_ARGUMENT`
* `UNAUTHENTICATED`
* `PERMISSION_DENIED`
* `NOT_FOUND`
* Non-idempotent writes without protection

Use exponential backoff, jitter, maximum attempts, and retry budgets.

---

## 15. Prevent Retry Storms

When a dependency fails, retries can multiply traffic.

Example:

```text
Original Traffic: 10,000 requests/second
Each Client Retries 3 Times
Potential Traffic: 40,000 requests/second
```

Use:

* Retry limits
* Retry budgets
* Exponential backoff
* Jitter
* Circuit breakers
* Load shedding
* Deadline checks

---

## 16. Use Specific Status Codes

Do not return `INTERNAL` for every failure.

Prefer:

```text
Invalid input          -> INVALID_ARGUMENT
Missing resource       -> NOT_FOUND
Duplicate resource     -> ALREADY_EXISTS
Temporary outage       -> UNAVAILABLE
Quota exceeded         -> RESOURCE_EXHAUSTED
Authentication failure -> UNAUTHENTICATED
Authorization failure  -> PERMISSION_DENIED
```

Consistent status codes improve retry and monitoring behavior.

---

## 17. Separate Authentication and Authorization

Authentication determines who the caller is.

Authorization determines what the caller may do.

Flow:

```text
Request Metadata
      |
      v
Authentication Interceptor
      |
      v
Caller Identity
      |
      v
Authorization Policy
      |
      v
Service Method
```

Always authorize access to the requested resource.

---

## 18. Use TLS or Mutual TLS

Use TLS for encrypted communication.

Use mutual TLS when both sides must prove their identities.

```text
Client Certificate
        |
        v
Server verifies client

Server Certificate
        |
        v
Client verifies server
```

A service mesh can automate certificate issuance and rotation.

---

## 19. Validate Every Request

Generated types provide structural validation, not complete business validation.

Validate:

* Required semantic fields
* String lengths
* Identifier formats
* Numeric ranges
* Timestamps
* Enum values
* Collection sizes
* Business constraints

Example:

```text
quantity must be greater than zero
currency must be supported
customer_id must not be empty
```

---

## 20. Use Interceptors for Cross-Cutting Concerns

Use interceptors for:

* Authentication
* Request IDs
* Tracing
* Metrics
* Logging
* Recovery
* Rate limiting

Do not duplicate this logic in every service method.

---

## 21. Monitor RPC-Level Metrics

Track metrics by:

* Service
* Method
* Status code
* Client
* Server
* Region
* Version

Important measurements include:

* Request rate
* Error rate
* p50 latency
* p95 latency
* p99 latency
* Active streams
* Message size
* Deadline exceeded count
* Cancellation count
* Retry count
* Connection count

Avoid uncontrolled high-cardinality labels such as raw user IDs.

---

## 22. Enable Distributed Tracing

A trace should follow the request across service boundaries.

```text
Checkout Service
      |
      ├── Inventory Service
      ├── Pricing Service
      └── Payment Service
```

Propagate trace context through gRPC metadata.

Tracing helps identify:

* Slow dependencies
* Retry loops
* Failed services
* Large fan-out
* Deadline exhaustion

---

## 23. Design for Graceful Shutdown

During deployment:

1. Stop accepting new traffic.
2. Mark the instance as not ready.
3. Allow active unary calls to finish.
4. Drain streams where possible.
5. Close connections after a timeout.
6. Terminate the process.

Abrupt shutdowns create unnecessary failures and retries.

---

## 24. Test Contract Compatibility

Automate checks for:

* Removed fields
* Reused field numbers
* Changed field types
* Removed methods
* Enum incompatibility
* Package changes
* Breaking API changes

Run compatibility checks in continuous integration.

---

## 25. Version Packages Deliberately

Package names can include major API versions.

```proto
package orders.v1;
```

A breaking redesign may introduce:

```proto
package orders.v2;
```

Avoid creating new versions for every additive change.

---

## Common Mistakes

## 1. Creating a New Channel for Every Request

This creates unnecessary handshakes, latency, sockets, and CPU usage.

Channels should usually be long-lived and reused.

---

## 2. Omitting Deadlines

Without deadlines, calls may wait indefinitely and consume resources long after callers stop caring about the result.

Every production RPC should have a deadline.

---

## 3. Retrying Non-Idempotent Operations

Retrying an unprotected write can create:

* Duplicate orders
* Duplicate payments
* Duplicate notifications
* Duplicate inventory reservations

Use idempotency keys or disable automatic retries.

---

## 4. Reusing Protobuf Field Numbers

Field numbers are permanent wire identifiers.

Reusing a removed number can cause old clients to misinterpret new data.

Reserve removed fields.

---

## 5. Changing Field Types Incompatibly

Changing:

```proto
string customer_id = 1;
```

to:

```proto
int64 customer_id = 1;
```

can break compatibility.

Add a new field instead.

---

## 6. Using `INTERNAL` for Every Error

This hides useful error meaning and prevents clients from making correct decisions.

Map errors to specific gRPC status codes.

---

## 7. Returning Sensitive Internal Errors

Do not expose:

* Stack traces
* SQL queries
* Internal hostnames
* Secret values
* File paths
* Infrastructure topology

Log internal details securely and return safe error information.

---

## 8. Assuming Load Balancing Works Like REST

One long-lived HTTP/2 connection may send many calls to the same backend.

Use a gRPC-aware proxy or client-side load balancing.

---

## 9. Sending Huge Unary Messages

Large unary messages increase memory pressure and retry cost.

Use pagination, streaming, or object storage for large data transfers.

---

## 10. Ignoring Backpressure

A fast producer can overwhelm a slower streaming consumer.

Use bounded queues, flow control, and application-level limits.

---

## 11. Using Streaming for Simple CRUD

Streaming creates operational complexity.

Use unary RPC when one request and one response are sufficient.

---

## 12. Treating Generated Code as Domain Logic

Generated Protobuf classes are transport models.

Do not tightly couple the entire domain layer to generated types.

Consider mapping between:

* Transport models
* Domain models
* Persistence models

This reduces coupling to the API contract.

---

## 13. Putting Business Data in Metadata

Metadata is intended for cross-cutting request information.

Do not place normal domain payloads in metadata.

Poor:

```text
x-order-items: ...
x-payment-amount: ...
```

Use typed request messages instead.

---

## 14. Ignoring Browser Limitations

Standard browser APIs do not provide full native gRPC support.

For browser clients, use:

* gRPC-Web
* A compatible edge proxy
* REST gateway
* GraphQL gateway

---

## 15. Assuming gRPC Automatically Makes a System Fast

gRPC reduces transport overhead, but it does not fix:

* Slow database queries
* Excessive service fan-out
* Poor caching
* Lock contention
* Large payloads
* Inefficient algorithms
* Network distance

Measure end-to-end performance.

---

## Interview Questions

### 1. What is gRPC?

gRPC is a high-performance RPC framework that commonly uses HTTP/2 and Protocol Buffers to provide strongly typed service-to-service communication.

---

### 2. What are the four gRPC communication patterns?

They are unary RPC, server streaming, client streaming, and bidirectional streaming.

---

### 3. Why are deadlines important in gRPC?

Deadlines prevent requests from waiting indefinitely, limit wasted work, and provide a latency budget that can propagate across downstream services.

---

### 4. How does gRPC differ from REST?

gRPC exposes typed remote procedures using binary Protocol Buffer messages, while REST usually exposes resource-oriented HTTP endpoints with JSON representations.

---

### 5. How do you maintain backward compatibility in Protocol Buffers?

Do not reuse field numbers, reserve removed fields, prefer additive changes, use safe enum defaults, and introduce a new major package version for breaking changes.

---

## Key Takeaways

1. **gRPC provides efficient, strongly typed communication using Protocol Buffers, generated code, HTTP/2 multiplexing, and native streaming.**

2. **Production gRPC systems depend on disciplined deadline propagation, cancellation, channel reuse, safe retries, load balancing, observability, and Protobuf compatibility.**

3. **gRPC is an excellent choice for high-performance internal service communication, while REST or GraphQL may remain better suited to browser-facing and public APIs.**
