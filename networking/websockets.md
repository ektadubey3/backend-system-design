# WebSockets

WebSockets is a full-duplex communication protocol that enables persistent, bidirectional communication between clients and servers over a single TCP connection.

Unlike traditional HTTP request-response communication, WebSockets allow both the client and the server to send messages independently without repeatedly opening new connections.

```text
        HTTP
Client ----------> Server
       Request

Client <---------- Server
      Response
```

WebSockets:

```text
Client <=======================> Server
        Persistent Connection
      Bidirectional Messages
```

WebSockets begin as an HTTP request.

```http
GET /chat HTTP/1.1
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: xxxxx
Sec-WebSocket-Version: 13
```

The server upgrades the connection.

```http
HTTP/1.1 101 Switching Protocols

Upgrade: websocket
Connection: Upgrade
```

After the handshake, communication switches from HTTP to the WebSocket protocol.

WebSockets are commonly used for:

* Chat applications
* Live notifications
* Multiplayer games
* Financial trading
* IoT platforms
* Collaborative editing
* Live dashboards
* Sports score updates
* Video signaling
* Presence systems
* Real-time analytics

---

# Why WebSockets?

Traditional HTTP is optimized for request-response interactions.

Many modern applications require continuous communication.

WebSockets solve this efficiently.

---

## 1. Full Duplex Communication

Both client and server can send data independently.

```text
Client
   |
   |------ Message ------>
   |
   |<----- Message -------
   |
   |------ Message ------>
```

No polling required.

---

## 2. Persistent Connection

Instead of opening a TCP connection for every request:

```text
Request 1

Open TCP
Send
Receive
Close

Request 2

Open TCP
Send
Receive
Close
```

WebSockets reuse one connection.

```text
Open TCP

Message
Message
Message
Message

Close TCP
```

Benefits include:

* Lower latency
* Fewer TCP handshakes
* Reduced TLS overhead
* Lower CPU usage
* Lower bandwidth consumption

---

## 3. Real-Time Communication

Servers immediately push events.

```text
Database Updated

↓

Server

↓

Client immediately receives update
```

Examples:

* New message
* Stock price update
* Live sports score
* Auction bid
* Device alert

---

## 4. Lower Network Overhead

HTTP requests repeatedly send:

* Headers
* Cookies
* Authentication metadata

WebSockets perform the HTTP handshake once.

Subsequent messages contain significantly less protocol overhead.

---

## 5. Event-Driven Systems

Servers publish events immediately.

```text
Order Created

↓

WebSocket Gateway

↓

Connected Clients
```

Ideal for event-driven architectures.

---

## 6. Efficient for High-Frequency Updates

Instead of:

```text
GET /price

GET /price

GET /price

GET /price
```

The server streams updates.

```text
$101

$101.25

$101.18

$101.41
```

---

## 7. Better User Experience

Users receive:

* Instant notifications
* Smooth dashboards
* Live collaboration
* Real-time chats
* Immediate game updates

without manually refreshing the page.

---

# Core Concepts

# 1. Persistent Connection

A WebSocket connection remains open until one side closes it.

```text
Client

Connect

↓

Open Connection

↓

Exchange Messages

↓

Disconnect
```

Persistent connections reduce connection setup costs.

---

# 2. Handshake

Every WebSocket connection starts as an HTTP request.

```http
GET /socket

Upgrade: websocket
```

Server:

```http
101 Switching Protocols
```

After this point:

```text
HTTP

↓

WebSocket Frames
```

---

# 3. Full Duplex Communication

Both sides communicate simultaneously.

```text
Client
    |
    |------ Ping ------->
    |
    |<----- Event -------
    |
    |------ Ack -------->
```

Neither side waits for the other.

---

# 4. Frames

WebSockets transmit frames rather than HTTP messages.

Common frame types:

* Text
* Binary
* Ping
* Pong
* Close
* Continuation

Example:

```text
Frame

Opcode
Payload Length
Payload
```

Applications typically work with complete messages while the protocol handles frame boundaries.

---

# 5. Messages

Applications exchange logical messages.

Example JSON:

```json
{
  "type": "chat_message",
  "roomId": "room-1",
  "text": "Hello!"
}
```

Binary formats such as Protocol Buffers or MessagePack may be used when performance matters.

---

# 6. Ping and Pong

Ping frames verify that the connection is still alive.

```text
Server

Ping

↓

Client

Pong
```

This helps detect dead connections.

---

# 7. Close Frames

Connections should close gracefully.

```text
Client

Close

↓

Server

Close

↓

TCP Closed
```

Graceful shutdown reduces resource leaks.

---

# 8. Connection Lifecycle

```text
HTTP Request

↓

Upgrade

↓

Connected

↓

Authenticated

↓

Message Exchange

↓

Heartbeat

↓

Disconnected
```

Every stage should be monitored.

---

# 9. Authentication

Authentication usually occurs:

* During the HTTP upgrade (e.g., cookies or headers)
* Using a token supplied during connection setup
* Via an application-level authentication message immediately after connecting

Example application message:

```json
{
  "type": "authenticate",
  "token": "<jwt-token>"
}
```

Always authenticate before allowing access to protected channels.

---

# 10. Rooms / Channels

Applications often group connections.

```text
Room A

Client 1
Client 2
Client 3
```

```text
Room B

Client 4
Client 5
```

Broadcasting becomes efficient.

---

# 11. Broadcasting

One message reaches many clients.

```text
Server

↓

Room

↓

Client A

↓

Client B

↓

Client C
```

Examples:

* Sports updates
* Notifications
* Auctions
* Chat rooms

---

# 12. One-to-One Messaging

```text
Alice

↓

Server

↓

Bob
```

Only the intended recipient receives the message.

---

# 13. Presence

Applications often track:

* Online
* Offline
* Away
* Busy
* Typing

Example:

```json
{
  "type": "presence",
  "status": "online"
}
```

Presence usually expires automatically if heartbeats stop.

---

# 14. Reconnection

Connections may break because of:

* Wi-Fi changes
* Mobile network switches
* Browser refresh
* Server restart
* Idle timeout

Clients should reconnect automatically.

Example strategy:

```text
Reconnect

1 sec

2 sec

4 sec

8 sec
```

Use exponential backoff with jitter.

---

# 15. Ordering

Messages sent over a single WebSocket connection preserve order.

```text
1

↓

2

↓

3

↓

4
```

Across multiple connections or distributed servers, ordering requires additional coordination.

---

# 16. Reliability

WebSockets provide reliable transport over TCP, but applications must still handle:

* Duplicate business operations
* Retries after reconnect
* Idempotency where appropriate
* Message acknowledgment (if required)
* Missed events during disconnection

---

# 17. Binary Messages

Applications may exchange binary payloads.

Advantages:

* Smaller messages
* Faster parsing
* Lower bandwidth
* Better performance

Useful for:

* Games
* Video signaling
* IoT
* Financial systems

---

# 18. Backpressure

A slow consumer can cause buffers to grow.

```text
Producer

10,000 msg/sec

↓

Consumer

1,000 msg/sec
```

Applications should:

* Limit queues
* Drop non-critical updates
* Slow producers
* Disconnect unhealthy clients if necessary

---

# 19. Heartbeats

Heartbeats detect stale connections.

```text
Server

Ping

↓

Client

Pong
```

If no response is received within the timeout, the server closes the connection.

---

# 20. Pub/Sub Integration

Large systems commonly integrate WebSocket gateways with a message broker.

```text
Publisher

↓

Kafka / Redis / NATS

↓

WebSocket Gateway

↓

Connected Clients
```

This enables scalable fan-out.

---

# Architecture

A production WebSocket architecture may look like this:

```text
                    ┌─────────────────────┐
                    │ Browser / Mobile App│
                    └──────────┬──────────┘
                               │
                        WebSocket (WSS)
                               │
                    ┌──────────▼──────────┐
                    │ CDN / Edge (HTTP)   │
                    │ TLS Termination     │
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │ Load Balancer       │
                    │ Sticky or Aware     │
                    └──────────┬──────────┘
                               │
          ┌────────────────────┼────────────────────┐
          │                    │                    │
 ┌────────▼────────┐  ┌────────▼────────┐  ┌────────▼────────┐
 │ WebSocket Node  │  │ WebSocket Node  │  │ WebSocket Node  │
 └────────┬────────┘  └────────┬────────┘  └────────┬────────┘
          │                    │                    │
          └────────────────────┼────────────────────┘
                               │
                 ┌─────────────▼─────────────┐
                 │ Pub/Sub (Redis/Kafka/NATS)│
                 └─────────────┬─────────────┘
                               │
      ┌────────────────────────┼────────────────────────┐
      │                        │                        │
┌─────▼─────┐          ┌───────▼───────┐       ┌────────▼────────┐
│ Chat Svc  │          │ Notification  │       │ Presence Service│
└───────────┘          └───────────────┘       └─────────────────┘
                               │
                        Databases / Cache
```

## Request Flow

1. Client establishes a WebSocket connection.
2. TLS handshake completes.
3. HTTP upgrade succeeds.
4. Client authenticates.
5. Connection joins one or more rooms.
6. Backend services publish events.
7. Pub/Sub distributes events.
8. WebSocket gateways push events to subscribed clients.
9. Heartbeats maintain connection health.
10. On disconnect, resources are cleaned up.

---

# Comparison

## WebSockets vs HTTP

| Feature       | WebSockets     | HTTP             |
| ------------- | -------------- | ---------------- |
| Connection    | Persistent     | Request-response |
| Communication | Bidirectional  | Client initiated |
| Latency       | Very low       | Higher           |
| Server Push   | Native         | Limited          |
| Headers       | One handshake  | Every request    |
| Best For      | Real-time apps | CRUD APIs        |

---

## WebSockets vs Server-Sent Events (SSE)

| Feature         | WebSockets    | SSE             |
| --------------- | ------------- | --------------- |
| Direction       | Bidirectional | Server → Client |
| Browser Support | Excellent     | Excellent       |
| Binary Data     | Yes           | No              |
| Client Messages | Yes           | No              |
| Use Case        | Chat, Games   | Notifications   |

---

## WebSockets vs Long Polling

| Feature     | WebSockets | Long Polling  |
| ----------- | ---------- | ------------- |
| Connection  | Persistent | Repeated HTTP |
| Latency     | Low        | Moderate      |
| Bandwidth   | Efficient  | Higher        |
| Scalability | Better     | Lower         |
| Complexity  | Higher     | Lower         |

---

## WebSockets vs gRPC Streaming

| Feature         | WebSockets             | gRPC Streaming                           |
| --------------- | ---------------------- | ---------------------------------------- |
| Browser Support | Native                 | Limited (requires gRPC-Web for browsers) |
| API Contract    | Application-defined    | Strong `.proto` contract                 |
| Serialization   | JSON/Binary            | Protocol Buffers                         |
| Best Use        | Browser real-time apps | Internal microservices                   |

---

## WebSockets vs Message Queues

| Feature          | WebSockets      | Message Queue     |
| ---------------- | --------------- | ----------------- |
| Communication    | Live            | Asynchronous      |
| Persistence      | No              | Often Yes         |
| Offline Delivery | No              | Usually           |
| Best Use         | Connected users | Backend workflows |

---

# Real-World Example

## Real-Time Chat Application

Requirements:

* Millions of users
* Instant messaging
* Typing indicators
* Read receipts
* Online presence
* Push notifications for offline users

### Architecture

```text
User A

↓

WebSocket Gateway

↓

Redis Pub/Sub

↓

WebSocket Gateway

↓

User B
```

### Message Flow

```text
User A

↓

Send Message

↓

Chat Service

↓

Store Message

↓

Publish Event

↓

Redis

↓

WebSocket Gateway

↓

User B
```

### Typing Indicator

```json
{
  "type": "typing",
  "roomId": "room-1",
  "userId": "user-123"
}
```

### Read Receipt

```json
{
  "type": "read_receipt",
  "messageId": "msg-789"
}
```

### Presence Update

```json
{
  "type": "presence",
  "status": "online"
}
```

### Offline User

If the recipient is disconnected:

1. Persist the message.
2. Push a mobile notification.
3. Deliver unread messages after reconnection.
4. Synchronize missed events.

---

# Best Practices

## 1. Always Use WSS

Encrypt all production traffic.

```text
wss://example.com/socket
```

---

## 2. Authenticate Every Connection

Never trust an unauthenticated socket.

Validate:

* JWT
* OAuth token
* Session cookie
* API token

---

## 3. Validate Every Message

Check:

* Schema
* Authorization
* Size
* Required fields
* Business rules

Never trust client input.

---

## 4. Implement Heartbeats

Detect stale connections using Ping/Pong.

---

## 5. Handle Reconnection Gracefully

Use exponential backoff with jitter.

Restore:

* Authentication (if needed)
* Subscriptions
* Room memberships
* Missed state

---

## 6. Limit Message Size

Reject oversized payloads.

Large messages can:

* Consume memory
* Increase latency
* Enable denial-of-service attacks

---

## 7. Apply Rate Limiting

Protect against:

* Spam
* Flood attacks
* Abuse
* Resource exhaustion

Rate-limit both:

* Connections
* Messages

---

## 8. Use Pub/Sub for Horizontal Scaling

Do not broadcast directly between WebSocket servers.

Instead:

```text
Backend

↓

Broker

↓

WebSocket Nodes
```

---

## 9. Track Connection State

Monitor:

* Active connections
* Room membership
* Authentication status
* Last heartbeat
* Connection age

---

## 10. Separate Business Logic

Keep WebSocket handlers lightweight.

```text
Socket Handler

↓

Application Service

↓

Database
```

---

## 11. Handle Backpressure

Slow clients should not affect the entire system.

Strategies include:

* Buffer limits
* Message dropping for non-critical events
* Connection termination when buffers overflow

---

## 12. Make Important Operations Idempotent

Reconnects and retries can produce duplicate actions.

Use message IDs or idempotency keys where business operations must not repeat.

---

## 13. Monitor Everything

Track:

* Connected clients
* Messages/sec
* Latency
* Errors
* Reconnect rate
* Disconnect reasons
* Broadcast fan-out
* Memory usage

---

## 14. Gracefully Drain Connections

During deployments:

1. Stop accepting new connections.
2. Notify clients if appropriate.
3. Allow active work to finish.
4. Close sockets cleanly.
5. Reconnect clients to healthy nodes.

---

## 15. Use WebSockets Only When Necessary

If updates occur infrequently, HTTP or Server-Sent Events may be simpler.

Choose WebSockets for genuine bidirectional or high-frequency communication.

---

# Common Mistakes

## 1. Using WebSockets for Simple CRUD

Not every application needs persistent connections.

---

## 2. Forgetting Authentication

Opening a socket does not imply authorization.

Authenticate and authorize every client.

---

## 3. Never Sending Heartbeats

Dead TCP connections can remain undetected.

Implement Ping/Pong.

---

## 4. Keeping Unlimited Buffers

Slow clients can exhaust server memory.

Apply limits and backpressure.

---

## 5. Ignoring Reconnection

Network interruptions are normal.

Clients should reconnect automatically.

---

## 6. Broadcasting Everything to Everyone

Target only the relevant:

* Room
* User
* Channel
* Tenant

---

## 7. Storing Connection State in One Server

Without shared state or Pub/Sub, horizontal scaling becomes difficult.

---

## 8. Ignoring Sticky Sessions or Shared Routing

Connection-aware load balancing is important for persistent connections.

---

## 9. Trusting Client Messages

Validate every incoming message.

Never assume:

* Identity
* Permissions
* Message format

---

## 10. Sending Huge Messages

Large payloads increase latency and memory usage.

Use pagination, chunking, or streaming strategies.

---

## 11. Ignoring Resource Cleanup

Always clean up:

* Rooms
* Presence
* Timers
* Buffers
* Subscriptions

after disconnect.

---

## 12. Assuming WebSockets Guarantee Business Delivery

TCP provides reliable transport for connected peers, but applications should still handle reconnection, missed events, acknowledgments (when required), and offline synchronization.

---

## Interview Questions

### 1. What is WebSockets?

WebSockets is a protocol that enables persistent, full-duplex communication between clients and servers over a single TCP connection.

---

### 2. How is WebSockets different from HTTP?

HTTP follows a request-response model, while WebSockets keep a persistent connection that allows both client and server to send messages at any time.

---

### 3. What is the WebSocket handshake?

A WebSocket connection starts as an HTTP request with an `Upgrade: websocket` header. If accepted, the server responds with `101 Switching Protocols`, after which communication uses the WebSocket protocol.

---

### 4. Why are Ping and Pong frames important?

They detect broken or idle connections, enabling servers and clients to remove stale sockets and maintain connection health.

---

### 5. How do you scale a WebSocket application?

Use multiple WebSocket servers behind a load balancer, share events through a Pub/Sub system such as Redis, Kafka, or NATS, maintain shared connection metadata where necessary, and monitor connection health and backpressure.

---

# Key Takeaways

1. **WebSockets provide persistent, bidirectional communication that enables low-latency, real-time applications such as chat, gaming, dashboards, and live notifications.**

2. **Scalable WebSocket systems require careful connection management, authentication, heartbeats, reconnection strategies, Pub/Sub integration, observability, and backpressure handling.**

3. **WebSockets are ideal for continuous real-time communication, but traditional HTTP, Server-Sent Events, or message queues may be better choices for simpler or asynchronous workloads.**
