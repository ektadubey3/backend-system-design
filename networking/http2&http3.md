# HTTP/2 and HTTP/3

HTTP/2 and HTTP/3 are newer versions of the HTTP protocol designed to overcome performance limitations found in HTTP/1.1.

Both protocols preserve familiar HTTP concepts such as:

* Methods like `GET`, `POST`, `PUT`, and `DELETE`
* Status codes
* Headers
* URLs
* Request-response communication
* Caching semantics
* Authentication mechanisms

The major differences are in how requests and responses are transported over the network.

```text
HTTP/1.1
Application Layer: HTTP
Transport Layer: TCP
Security Layer: Optional TLS
```

```text
HTTP/2
Application Layer: HTTP/2
Transport Layer: TCP
Security Layer: Usually TLS
```

```text
HTTP/3
Application Layer: HTTP/3
Transport Layer: QUIC over UDP
Security Layer: TLS 1.3 integrated into QUIC
```

HTTP/2 improves connection efficiency by introducing binary framing, multiplexing, and header compression.

HTTP/3 builds on these improvements by replacing TCP with QUIC, reducing connection-establishment time and minimizing transport-level head-of-line blocking.

---

## Why HTTP/2 and HTTP/3?

Modern applications frequently load many resources and call multiple backend services.

A single page or mobile screen may require:

* HTML
* CSS
* JavaScript
* Images
* Fonts
* API responses
* Authentication requests
* Analytics requests
* Feature configuration
* Recommendation data

With HTTP/1.1, clients often open multiple TCP connections because each connection handles requests inefficiently.

HTTP/2 and HTTP/3 improve this model by allowing many requests to share a smaller number of connections.

Benefits include:

* Lower latency
* Better bandwidth utilization
* Fewer network connections
* Reduced header overhead
* Improved performance on high-latency networks
* Better handling of unstable mobile connections
* Faster page and API response delivery
* Reduced connection-establishment cost

For backend system design, HTTP version choice affects:

* Load balancer configuration
* API gateway behavior
* Connection pooling
* TLS termination
* Service-to-service communication
* CDN performance
* Observability
* Capacity planning
* Retry behavior
* Infrastructure compatibility

---

## Core Concepts

## 1. Binary Framing

HTTP/1.1 is primarily a text-based protocol.

HTTP/2 and HTTP/3 divide requests and responses into binary frames.

```text
HTTP Message
     |
     v
+------------+
| Headers    |
+------------+
| Body       |
+------------+

Converted into:

+--------------+
| HEADERS Frame|
+--------------+
| DATA Frame   |
+--------------+
| DATA Frame   |
+--------------+
```

Binary framing makes protocol parsing more efficient and allows multiple streams to share a single connection.

Common frame types in HTTP/2 include:

* `HEADERS`
* `DATA`
* `SETTINGS`
* `WINDOW_UPDATE`
* `RST_STREAM`
* `PING`
* `GOAWAY`

Applications do not usually manage frames directly. HTTP libraries, servers, proxies, and gateways handle them.

---

## 2. Streams

A stream is an independent sequence of frames within a connection.

Each request-response exchange usually uses its own stream.

```text
Single Connection
   |
   ├── Stream 1: GET /users
   ├── Stream 3: GET /products
   ├── Stream 5: POST /orders
   └── Stream 7: GET /recommendations
```

Multiple streams allow concurrent communication without opening a separate connection for every request.

HTTP/2 streams operate over one TCP connection.

HTTP/3 streams operate over QUIC.

---

## 3. Multiplexing

Multiplexing allows multiple requests and responses to be active at the same time over one connection.

Without multiplexing:

```text
Request A
   |
Response A
   |
Request B
   |
Response B
```

With multiplexing:

```text
Connection
   |
   ├── Request A ───── Response A
   ├── Request B ─ Response B
   ├── Request C ───────── Response C
   └── Request D ─── Response D
```

Frames from different streams can be interleaved.

This reduces the need for many parallel TCP connections.

---

## 4. Head-of-Line Blocking

Head-of-line blocking occurs when one delayed unit of data prevents other data from being processed.

### HTTP/1.1 Application-Level Blocking

In HTTP/1.1 pipelining, responses must generally arrive in request order.

A slow response can delay later responses.

```text
Request A: Slow
Request B: Fast
Request C: Fast

Response order:
A -> B -> C
```

### HTTP/2 Transport-Level Blocking

HTTP/2 solves application-level head-of-line blocking through multiplexing.

However, all streams still share one TCP connection.

If a TCP packet is lost, TCP must recover the missing packet before delivering later data to the application.

```text
TCP Connection
   |
   ├── HTTP/2 Stream A
   ├── HTTP/2 Stream B
   └── HTTP/2 Stream C

Packet loss affects delivery across the connection.
```

### HTTP/3 Stream-Level Independence

HTTP/3 uses QUIC streams.

Packet loss affecting one stream does not necessarily block unrelated streams.

```text
QUIC Connection
   |
   ├── Stream A: Packet loss
   ├── Stream B: Continues
   └── Stream C: Continues
```

This is one of the most important advantages of HTTP/3.

---

## 5. Header Compression

HTTP requests frequently repeat headers.

Example:

```http
Authorization: Bearer <token>
Accept: application/json
User-Agent: mobile-app/4.2
Content-Type: application/json
```

Sending these headers repeatedly wastes bandwidth.

### HTTP/2: HPACK

HTTP/2 uses HPACK header compression.

HPACK uses:

* Static header tables
* Dynamic header tables
* Huffman encoding
* Indexed header values

### HTTP/3: QPACK

HTTP/3 uses QPACK.

QPACK is designed for QUIC's independent stream model.

It reduces compression-related blocking while retaining efficient header compression.

---

## 6. Flow Control

Flow control prevents a sender from overwhelming a receiver.

HTTP/2 supports flow control at:

* Connection level
* Stream level

```text
Connection Window
   |
   ├── Stream A Window
   ├── Stream B Window
   └── Stream C Window
```

Receivers use window updates to indicate how much additional data they can accept.

Improper flow-control configuration can reduce throughput or consume excessive memory.

---

## 7. Stream Prioritization

Clients may communicate which resources are more important.

For example:

```text
High Priority
   ├── Main HTML
   ├── Critical CSS
   └── Authentication API

Lower Priority
   ├── Analytics
   ├── Recommendations
   └── Background Images
```

Prioritization can improve perceived performance.

However, implementation behavior differs across clients, servers, proxies, and protocol versions.

Backend systems should not depend exclusively on client prioritization for critical operations.

---

## 8. Connection Reuse

HTTP/2 and HTTP/3 are designed to reuse connections efficiently.

Connection reuse reduces:

* TCP handshakes
* TLS handshakes
* CPU overhead
* Socket usage
* Network round trips
* Load balancer connection count

```text
Without Reuse

Request 1 -> New Connection
Request 2 -> New Connection
Request 3 -> New Connection
```

```text
With Reuse

Single Connection
   ├── Request 1
   ├── Request 2
   └── Request 3
```

Connection reuse is especially important for:

* Mobile applications
* Microservices
* High-traffic APIs
* Multi-region systems
* High-latency networks

---

## 9. QUIC

QUIC is the transport protocol used by HTTP/3.

QUIC runs over UDP but implements transport features commonly associated with TCP and TLS.

QUIC provides:

* Reliable delivery
* Congestion control
* Encryption
* Stream multiplexing
* Connection migration
* Faster connection establishment
* Integrated TLS 1.3

```text
 HTTP/3
   |
   v
  QUIC
   |
   v
  UDP
   |
   v
  IP
```

QUIC is implemented mostly in user space, which makes protocol evolution easier than modifying operating-system TCP stacks.

---

## 10. TLS Integration

HTTP/2 usually operates over TLS in public web environments.

The normal setup requires:

1. TCP connection establishment
2. TLS handshake
3. HTTP/2 communication

```text
Client                       Server
  |----- TCP Handshake ------->|
  |----- TLS Handshake ------->|
  |===== HTTP/2 Traffic =======|
```

HTTP/3 integrates TLS 1.3 into QUIC.

```text
Client                       Server
  |----- QUIC + TLS ---------->|
  |===== HTTP/3 Traffic =======|
```

This can reduce connection-establishment latency.

---

## 11. Zero-Round-Trip Resumption

QUIC can support zero-round-trip connection resumption for previously contacted servers.

```text
First Connection
Client -> Handshake -> Server
Client <-> Encrypted Traffic
```

```text
Resumed Connection
Client -> Early Application Data
```

This can improve latency for returning clients.

However, zero-round-trip data may be replayed by an attacker.

It should only be used for operations that are safe to repeat.

Good candidates:

* `GET` requests
* Cacheable requests
* Idempotent reads

Risky candidates:

* Payment creation
* Order submission
* Password changes
* Fund transfers
* Non-idempotent operations

---

## 12. Connection Migration

A TCP connection is normally associated with:

* Source IP
* Source port
* Destination IP
* Destination port

When a mobile device switches from Wi-Fi to cellular, the connection may break.

QUIC uses connection identifiers, allowing an existing connection to survive some network changes.

```text
Mobile Device
   |
   | Wi-Fi Connection
   v
HTTP/3 Server

Network changes

Mobile Device
   |
   | Cellular Connection
   v
Same HTTP/3 Session
```

This is particularly valuable for mobile applications.

---

## Architecture

A production architecture supporting HTTP/2 and HTTP/3 may look like this:

```text
                         ┌────────────────────┐
                         │       Client       │
                         │ Browser / Mobile / │
                         │ API Consumer       │
                         └─────────┬──────────┘
                                   │
                    HTTP/2 over TCP or HTTP/3 over QUIC
                                   │
                         ┌─────────▼──────────┐
                         │        DNS         │
                         │ HTTPS / SVCB Data  │
                         └─────────┬──────────┘
                                   │
                         ┌─────────▼──────────┐
                         │    CDN / Edge      │
                         │ HTTP/2 + HTTP/3    │
                         │ TLS Termination    │
                         │ Caching            │
                         └─────────┬──────────┘
                                   │
                         ┌─────────▼──────────┐
                         │   WAF / Gateway    │
                         │ Authentication     │
                         │ Rate Limiting      │
                         └─────────┬──────────┘
                                   │
                         HTTP/2, HTTP/1.1,
                           gRPC, or mTLS
                                   │
                         ┌─────────▼──────────┐
                         │   Load Balancer    │
                         └─────────┬──────────┘
                                   │
                    ┌──────────────┼──────────────┐
                    │              │              │
           ┌────────▼───────┐ ┌────▼──────────┐ ┌─▼──────────────┐
           │ Backend API 1  │ │ Backend API 2 │ │ Backend API 3  │
           │ HTTP/2 / gRPC  │ │ HTTP/2 / gRPC │ │ HTTP/2 / gRPC  │
           └────────┬───────┘ └────┬──────────┘ └─┬──────────────┘
                    │              │              │
                    └──────────────┼──────────────┘
                                   │
                     ┌─────────────┼─────────────┐
                     │             │             │
             ┌───────▼──────┐ ┌────▼──────┐ ┌────▼───────────┐
             │ Distributed  │ │ Database  │ │ Message Queue  │
             │ Cache        │ │ Cluster   │ │ Event Stream   │
             └──────────────┘ └───────────┘ └────────────────┘
```

### Request Flow

1. The client resolves the domain through DNS.
2. The client connects to a CDN or edge server.
3. The client negotiates HTTP/2 or HTTP/3.
4. TLS establishes encrypted communication.
5. The edge checks whether the response is cached.
6. The WAF inspects the request.
7. The API gateway performs authentication and rate limiting.
8. The request is forwarded to the backend.
9. Backend services communicate using HTTP/2, gRPC, or another internal protocol.
10. The response returns through the gateway and edge.
11. Metrics, logs, and traces are collected across every layer.

---

## Protocol Negotiation

Clients and servers need to agree on the HTTP version.

### HTTP/2 Negotiation

HTTP/2 over TLS commonly uses Application-Layer Protocol Negotiation.

The client may advertise:

```text
h2
http/1.1
```

The server selects a supported protocol.

```text
Client supports:
- h2
- http/1.1

Server selects:
- h2
```

### HTTP/3 Discovery

A server may indicate HTTP/3 availability using the `Alt-Svc` response header.

```http
Alt-Svc: h3=":443"; ma=86400
```

This tells the client that HTTP/3 is available on port `443`.

The client may use HTTP/2 for the first request and HTTP/3 for later requests.

Clients may also discover HTTP/3 support through DNS service-binding records.

---

## Comparison

## HTTP/1.1 vs HTTP/2 vs HTTP/3

| Feature                         | HTTP/1.1              | HTTP/2              | HTTP/3                            |
| ------------------------------- | --------------------- | ------------------- | --------------------------------- |
| Transport                       | TCP                   | TCP                 | QUIC over UDP                     |
| Message format                  | Text-based            | Binary framing      | Binary framing                    |
| Multiplexing                    | Limited               | Yes                 | Yes                               |
| Header compression              | No native compression | HPACK               | QPACK                             |
| Connection reuse                | Supported             | Strongly optimized  | Strongly optimized                |
| Stream independence             | No                    | Partial             | Yes                               |
| Transport head-of-line blocking | Yes                   | Yes                 | Reduced                           |
| TLS                             | Optional              | Commonly required   | Integrated TLS 1.3                |
| Connection migration            | No                    | No                  | Yes                               |
| Handshake latency               | Higher                | Lower with reuse    | Lower, especially with resumption |
| Packet-loss behavior            | Can affect connection | Affects all streams | Mostly isolated per stream        |
| UDP dependency                  | No                    | No                  | Yes                               |
| Infrastructure compatibility    | Very high             | High                | Growing                           |
| Best fit                        | Legacy systems        | Modern web and APIs | Mobile, global, lossy networks    |

---

## HTTP/2 vs HTTP/3

| Category                   | HTTP/2                      | HTTP/3                                    |
| -------------------------- | --------------------------- | ----------------------------------------- |
| Underlying transport       | TCP                         | QUIC over UDP                             |
| Encryption                 | TLS layered over TCP        | TLS integrated into QUIC                  |
| Multiplexing               | Multiple streams over TCP   | Multiple QUIC streams                     |
| Packet loss                | Can delay all streams       | Usually impacts only affected streams     |
| Connection migration       | Not supported               | Supported                                 |
| Header compression         | HPACK                       | QPACK                                     |
| Network support            | Broad                       | May be blocked by some networks           |
| Operational maturity       | Very mature                 | Increasingly mature                       |
| Debugging complexity       | Moderate                    | Higher                                    |
| Load balancer requirements | TCP and TLS support         | UDP and QUIC support                      |
| CPU usage                  | Usually predictable         | Can be higher depending on implementation |
| Ideal use case             | Stable network environments | Mobile and high-latency environments      |

---

## HTTP/2 vs WebSocket

HTTP/2 and WebSocket solve different problems.

| Category                  | HTTP/2                        | WebSocket                          |
| ------------------------- | ----------------------------- | ---------------------------------- |
| Communication model       | Request-response with streams | Persistent bidirectional messaging |
| Server push               | Limited protocol support      | Native server-to-client messaging  |
| API style                 | HTTP semantics                | Message-based                      |
| Caching                   | Standard HTTP caching         | Usually custom                     |
| Good for REST APIs        | Yes                           | Not usually                        |
| Good for live chat        | Possible but not ideal        | Yes                                |
| Good for streaming events | Possible                      | Yes                                |
| Multiplexing              | Built in                      | Requires application design        |

Use HTTP/2 or HTTP/3 for standard API communication.

Use WebSocket when the application requires long-lived, real-time, bidirectional messaging.

---

## HTTP/2 and gRPC

gRPC commonly uses HTTP/2 because HTTP/2 supports:

* Multiplexed streams
* Binary messages
* Bidirectional streaming
* Long-lived connections
* Header compression
* Efficient service-to-service communication

```text
Client Service
     |
     | gRPC over HTTP/2
     v
Server Service
```

gRPC communication patterns include:

* Unary request-response
* Server streaming
* Client streaming
* Bidirectional streaming

HTTP/3 support for RPC systems is possible, but HTTP/2 remains common in backend service meshes and internal networks.

---

## Real-World Example: Mobile E-Commerce Application

Consider a mobile application loading its home screen.

The application needs:

* User profile
* Product categories
* Recommended products
* Cart information
* Promotions
* Recently viewed items

### HTTP/1.1 Approach

```text
Mobile App
   |
   ├── Connection 1 -> User Profile
   ├── Connection 2 -> Categories
   ├── Connection 3 -> Recommendations
   ├── Connection 4 -> Cart
   ├── Connection 5 -> Promotions
   └── Connection 6 -> Recent Items
```

This creates multiple connections and increases handshake overhead.

### HTTP/2 Approach

```text
Single TCP Connection
   |
   ├── Stream 1 -> User Profile
   ├── Stream 3 -> Categories
   ├── Stream 5 -> Recommendations
   ├── Stream 7 -> Cart
   ├── Stream 9 -> Promotions
   └── Stream 11 -> Recent Items
```

All requests share one connection.

This reduces connection overhead and improves resource utilization.

### Packet Loss with HTTP/2

Suppose the user is on an unstable mobile network.

```text
TCP Packet Lost
      |
      v
TCP Recovery
      |
      v
Delivery of multiple HTTP/2 streams may pause
```

Even unrelated responses may be delayed because TCP preserves ordered delivery.

### HTTP/3 Approach

```text
Single QUIC Connection
   |
   ├── Stream 1 -> User Profile
   ├── Stream 3 -> Categories
   ├── Stream 5 -> Recommendations
   ├── Stream 7 -> Cart
   ├── Stream 9 -> Promotions
   └── Stream 11 -> Recent Items
```

If a packet for the recommendations stream is lost:

```text
Recommendations Stream -> Delayed

User Profile Stream     -> Continues
Categories Stream       -> Continues
Cart Stream             -> Continues
Promotions Stream       -> Continues
```

This improves responsiveness on unreliable networks.

### Network Change

The user leaves home and switches from Wi-Fi to cellular data.

With TCP, the connection may need to be re-established.

With QUIC, connection migration may preserve the session.

```text
Wi-Fi
  |
  v
QUIC Connection
  |
Network Change
  |
  v
Cellular
  |
  v
Same Logical QUIC Connection
```

### Backend Architecture

```text
Mobile Application
       |
       | HTTP/3
       v
CDN / Edge
       |
       | HTTP/2
       v
API Gateway
       |
       | gRPC over HTTP/2
       v
Backend-for-Frontend
       |
       ├── User Service
       ├── Catalog Service
       ├── Cart Service
       ├── Promotion Service
       └── Recommendation Service
```

The edge may accept HTTP/3 from clients while using HTTP/2 or HTTP/1.1 to communicate with internal services.

This is common because external and internal protocol requirements may differ.

---

## Best Practices

## 1. Support Protocol Fallback

Do not assume all clients and networks support HTTP/3.

A production system should support graceful fallback.

```text
Preferred:
HTTP/3

Fallback:
HTTP/2

Final fallback:
HTTP/1.1
```

Some corporate networks, firewalls, and proxies may block UDP traffic.

Clients should still be able to connect using HTTP/2 or HTTP/1.1.

---

## 2. Enable HTTP/2 and HTTP/3 at the Edge

CDNs and reverse proxies are usually the best place to terminate client HTTP/2 and HTTP/3 connections.

Benefits include:

* Centralized TLS configuration
* Better protocol compatibility
* Reduced backend complexity
* Improved caching
* DDoS protection
* Easier certificate management
* Global edge presence

```text
Client
   |
   | HTTP/3
   v
CDN
   |
   | HTTP/2
   v
Backend
```

Backend services do not always need native HTTP/3 support.

---

## 3. Keep Connections Long-Lived

Avoid frequently closing HTTP/2 or HTTP/3 connections.

Long-lived connections improve:

* Multiplexing efficiency
* Congestion-window growth
* TLS reuse
* Header compression
* Connection-pool efficiency

However, define reasonable limits for:

* Maximum connection age
* Idle timeout
* Maximum streams
* Maximum requests per connection
* Memory consumption

---

## 4. Configure Stream Limits

A single connection can carry many concurrent streams.

Without limits, one client may consume excessive resources.

Configure:

* Maximum concurrent streams
* Maximum request body size
* Maximum header size
* Maximum field count
* Stream idle timeout
* Connection-level memory limits

A typical server advertises stream limits using protocol settings.

---

## 5. Avoid Creating Too Many Connections

HTTP/2 and HTTP/3 are designed to multiplex requests.

Opening many connections can reduce their benefits.

Too many connections increase:

* TLS handshakes
* CPU usage
* Memory usage
* Socket count
* Load balancer state
* Congestion-control competition

Use connection pools and reuse established connections.

---

## 6. Configure Timeouts at Every Layer

Define timeouts for:

* Connection establishment
* TLS or QUIC handshake
* Request headers
* Request body
* Backend response
* Stream inactivity
* Connection inactivity

Example:

```text
Client Timeout:            10 seconds
Edge Timeout:               9 seconds
API Gateway Timeout:        8 seconds
Backend Service Timeout:    6 seconds
Database Timeout:           3 seconds
```

Downstream timeouts should fit within the overall request budget.

---

## 7. Use Idempotency for Retries and Zero-RTT

Operations that may be retried should be idempotent.

For non-idempotent operations, use an idempotency key.

```http
POST /payments
Idempotency-Key: payment-7f928
```

This is especially important when using:

* Automatic retries
* Mobile networks
* HTTP/3 connection recovery
* Zero-round-trip resumption
* Load balancer failover

---

## 8. Monitor Protocol-Specific Metrics

Track HTTP version usage separately.

Useful metrics include:

* HTTP/1.1 request count
* HTTP/2 request count
* HTTP/3 request count
* Connection-establishment latency
* TLS handshake duration
* QUIC handshake duration
* Active connections
* Active streams
* Stream resets
* Connection resets
* Packet-loss rate
* Round-trip time
* Retry count
* Protocol fallback rate
* UDP failure rate
* Request latency by protocol

Example dashboard:

```text
Protocol Adoption
- HTTP/1.1: 12%
- HTTP/2:   61%
- HTTP/3:   27%

Performance
- HTTP/2 p95 latency: 420 ms
- HTTP/3 p95 latency: 350 ms
```

---

## 9. Test on Real Network Conditions

Local development networks rarely represent production conditions.

Test with:

* High latency
* Packet loss
* Limited bandwidth
* Wi-Fi-to-cellular transitions
* Corporate proxies
* UDP-blocked networks
* Geographic distance
* Mobile devices
* Older operating systems

HTTP/3 benefits are often more visible under imperfect network conditions.

---

## 10. Keep UDP Infrastructure Ready

HTTP/3 requires UDP support.

Check that the following components support QUIC:

* Firewall
* Load balancer
* CDN
* Reverse proxy
* Network security group
* Container platform
* Observability platform
* DDoS protection layer

TCP port `443` support alone is not sufficient.

HTTP/3 typically requires UDP on port `443`.

---

## 11. Avoid Unnecessary Domain Sharding

Domain sharding was historically used to bypass HTTP/1.1 browser connection limits.

Example:

```text
static1.example.com
static2.example.com
static3.example.com
```

With HTTP/2 and HTTP/3, excessive sharding can be harmful because it creates additional:

* DNS lookups
* Connections
* TLS handshakes
* Congestion windows
* Certificate management complexity

Prefer fewer well-optimized origins when possible.

---

## 12. Validate Proxy and Gateway Compatibility

Some proxies may:

* Downgrade HTTP versions
* Buffer streams
* Remove headers
* Alter timeout behavior
* Break long-lived streams
* Limit concurrent streams
* Disable trailers

Test the entire request path, not only the client-facing server.

```text
Client
  |
HTTP/3
  |
CDN
  |
HTTP/2
  |
Gateway
  |
HTTP/1.1
  |
Backend
```

The effective behavior may be limited by the weakest layer.

---

## 13. Tune Flow Control Carefully

Small flow-control windows may limit throughput.

Very large windows may increase memory consumption.

Tune based on:

* Average response size
* Bandwidth-delay product
* Number of concurrent streams
* Client behavior
* Server memory
* Network latency

Do not copy flow-control settings without measuring their effect.

---

## 14. Use Compression Carefully

Header compression is built into HTTP/2 and HTTP/3.

Response-body compression still requires formats such as:

* Brotli
* Gzip
* Zstandard, where supported

Example:

```http
Accept-Encoding: br, gzip
```

Avoid compressing:

* Already compressed images
* Video files
* Very small responses
* Secret-dependent data in unsafe contexts

Compression uses CPU, so measure the trade-off.

---

## 15. Protect Against Protocol-Level Abuse

Attackers may abuse multiplexed connections by creating many streams or sending incomplete requests.

Protect systems using:

* Stream limits
* Header limits
* Request-size limits
* Idle timeouts
* Rate limiting
* Connection limits
* WAF policies
* Resource quotas
* Updated server software

Protocol support should always include resource-control policies.

---

## Common Mistakes

## 1. Assuming HTTP/2 Removes All Head-of-Line Blocking

HTTP/2 removes HTTP/1.1 application-level blocking.

It does not remove TCP-level head-of-line blocking.

Packet loss can still delay all streams sharing the same TCP connection.

HTTP/3 reduces this problem using independent QUIC streams.

---

## 2. Treating HTTP/3 as HTTP/2 Over UDP

HTTP/3 is not simply HTTP/2 transported over UDP.

It uses:

* QUIC streams
* QPACK
* Integrated TLS 1.3
* QUIC connection management
* QUIC congestion control

HTTP/2 frames cannot simply be sent unchanged over UDP.

---

## 3. Enabling HTTP/3 Without Fallback

UDP may be blocked or degraded.

A system that supports only HTTP/3 may become unavailable to some users.

Always maintain HTTP/2 or HTTP/1.1 fallback.

---

## 4. Opening Many Connections

Using many connections defeats multiplexing benefits.

One well-managed connection can often handle many parallel requests.

Too many connections increase infrastructure overhead.

---

## 5. Ignoring Long-Lived Connection Costs

Multiplexing reduces connection count, but each connection still consumes resources.

Long-lived connections use:

* Memory
* File descriptors
* Connection state
* TLS state
* Flow-control buffers
* Load balancer state

Set appropriate idle and lifetime limits.

---

## 6. Assuming Every Backend Must Use HTTP/3

HTTP/3 is especially useful between external clients and edge infrastructure.

Internal services on stable private networks may perform well with:

* HTTP/2
* gRPC
* HTTP/1.1 with connection pooling
* Other RPC protocols

Choose protocols based on requirements, not trends.

---

## 7. Ignoring UDP Load Balancing

A TCP load balancer cannot automatically handle QUIC traffic.

HTTP/3 requires infrastructure that supports UDP-aware routing and QUIC connection identifiers.

Verify support before enabling the protocol.

---

## 8. Using Zero-RTT for Unsafe Requests

Zero-round-trip data may be replayed.

Do not use it for operations such as:

* Charging a payment method
* Placing an order
* Changing credentials
* Sending money
* Deleting critical data

Restrict it to safe or idempotent operations.

---

## 9. Expecting HTTP/3 to Improve Every Request

HTTP/3 is not always faster.

Performance depends on:

* Network quality
* Packet loss
* Client support
* Server implementation
* Connection reuse
* Geographic distance
* CPU cost
* Existing CDN behavior

On a stable low-latency network, HTTP/2 may perform similarly.

Measure before and after adoption.

---

## 10. Ignoring Protocol Downgrades

A client may connect with HTTP/3 at the edge while the edge uses HTTP/1.1 to the backend.

This is not necessarily wrong, but it must be understood.

```text
Client -> HTTP/3 -> CDN -> HTTP/1.1 -> Backend
```

Backend bottlenecks may still exist despite HTTP/3 at the client-facing layer.

---

## 11. Relying on Server Push

HTTP/2 server push was designed to send resources before the client requested them.

In practice, it has limited support and can waste bandwidth by sending resources already cached by the client.

Prefer:

* Preload hints
* Good caching
* Resource prioritization
* CDN optimization
* Efficient API design

---

## 12. Ignoring Observability Gaps

Traditional TCP monitoring tools may not provide enough visibility into QUIC because transport behavior is encrypted and implemented in user space.

Ensure observability tools can report:

* QUIC handshakes
* Stream resets
* Connection migration
* Packet loss
* Protocol negotiation
* HTTP/3 fallback

---

## Interview Questions

#### 1. What is the main difference between HTTP/2 and HTTP/3?

HTTP/2 runs over TCP, while HTTP/3 runs over QUIC using UDP. QUIC allows independent streams, reducing transport-level head-of-line blocking.

#### 2. How does multiplexing improve HTTP performance?

Multiplexing allows multiple requests and responses to share one connection concurrently, reducing the need for multiple connections and repeated handshakes.

#### 3. Why can packet loss affect all HTTP/2 streams?

All HTTP/2 streams share one TCP connection. TCP must recover lost packets in order before delivering later data, which can delay every stream on that connection.

#### 4. What is connection migration in HTTP/3?

Connection migration allows a QUIC connection to continue when the client's network changes, such as switching from Wi-Fi to cellular, using a connection identifier rather than relying only on IP addresses and ports.

#### 5. Should every backend service use HTTP/3?

No. HTTP/3 is most valuable for client-to-edge communication over unreliable or high-latency networks. HTTP/2 or gRPC may remain simpler and effective for internal service communication.

---

## Key Takeaways

1. **HTTP/2 improves efficiency through binary framing, multiplexing, connection reuse, and HPACK compression, but it still inherits TCP-level head-of-line blocking.**

2. **HTTP/3 uses QUIC to provide faster connection establishment, stream independence, integrated TLS, and connection migration, making it valuable for mobile and unreliable networks.**

3. **Successful adoption requires more than enabling a protocol flag. Infrastructure must support fallback, UDP routing, connection limits, observability, security controls, and realistic performance testing.**
