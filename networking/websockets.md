# WebSockets

WebSocket is a message-framing protocol for long-lived two-way communication between a client and server.

The key distinction from ordinary HTTP is **bidirectional application messaging and server-initiated delivery**, not merely connection reuse—modern HTTP already supports persistent connections, and HTTP/2 multiplexes requests.

## Interview TL;DR

1. WebSockets provide long-lived bidirectional message exchange.
2. Traditional WebSocket uses an HTTP opening handshake and then WebSocket framing over TCP; extensions also define bootstrapping over newer HTTP versions.
3. Persistent connections create a **connection-capacity** problem, not only an RPS problem.
4. A connection server should not own irreplaceable session/message state only in memory.
5. Define heartbeats, idle timeouts, reconnect, resume, and duplicate handling.
6. Load balancers/proxies must support upgrades/long-lived connections and graceful draining.
7. Backpressure matters: slow clients can consume memory if outbound buffers are unbounded.
8. Message delivery semantics and ordering are application-level concerns.
9. Horizontal scaling requires a routing/fan-out layer when events for one connection originate elsewhere.
10. Consider SSE for one-way server push and ordinary HTTP for infrequent updates.

## Connection Model

```text
client <=================> connection gateway
                              |
                              +--> auth/session
                              +--> event/fan-out layer
                              +--> backend services
```

The gateway manages connections; durable business state should usually live elsewhere.

## Opening Handshake

Traditional WebSocket starts with an HTTP upgrade handshake and then switches to WebSocket framing.

Do not use the handshake as proof that application authentication remains valid forever. Define token/session refresh or reconnect policy.

## Capacity Estimation

Estimate:

```text
concurrent connections
messages/sec
bytes/message
fan-out recipients
heartbeat interval
reconnect burst
outbound buffer memory
```

One million mostly idle connections and one million active chatters are different systems.

## Heartbeats

Use protocol/app heartbeat strategy to detect dead peers and keep infrastructure state fresh.

Too-frequent heartbeats create large baseline traffic.

Example:

```text
2M connections / 30s heartbeat
≈ 66k heartbeat operations/sec
```

before user messages.

## Backpressure and Slow Consumers

If a client reads slowly:

```text
producer fast
   ↓
server outbound buffer grows
   ↓
memory exhaustion
```

Define:

- max buffered messages/bytes;
- drop policy for ephemeral updates;
- disconnect policy;
- durable replay source where required.

## Reconnect and Resume

Connections will drop.

Client reconnect strategy:

```text
disconnect
  ↓
backoff + jitter
  ↓
re-authenticate
  ↓
resume from cursor/sequence if supported
```

If messages matter, use durable sequence/cursor semantics. Do not depend on the socket itself for durability.

## Horizontal Scaling

Suppose user A connects to gateway 7, but a message is produced on another node.

You need a distribution mechanism:

```text
event
  ↓
broker / pub-sub / routing layer
  ↓
gateway owning user A's connection
```

Avoid broadcasting every event to every gateway at large scale unless the workload is tiny.

## Ordering

TCP orders bytes on one connection, but end-to-end message ordering can still be affected by:

- reconnects;
- multiple producers;
- multiple regions;
- async processing.

Use application sequence numbers if ordering is a product requirement.

## Graceful Deployment

When draining a gateway:

- stop new connections;
- allow existing connections to migrate/close;
- send reconnect signal if protocol supports it;
- avoid terminating millions of sockets simultaneously.

Prevent a reconnect storm.

## Security

Validate:

- Origin where appropriate for browser clients;
- authentication;
- authorization per message/action;
- message size;
- rate limits;
- compression abuse;
- input schema.

## Alternatives

### Server-Sent Events

Good for server → browser streams with HTTP semantics and automatic reconnect support.

### Long polling

Simple fallback for lower-frequency realtime needs.

### gRPC streaming

Strong for controlled service/native clients.

## Common Mistakes

- “HTTP opens a new TCP connection for every request”;
- storing durable chat state only in gateway memory;
- no slow-client buffer limit;
- no reconnect/resume design;
- sticky session treated as the whole routing strategy;
- forgetting connection drain/reconnect storms;
- assuming socket order solves global event ordering.

## 2-Minute Interview Answer

> “I use WebSockets when the product needs long-lived bidirectional messaging. I estimate concurrent connections and fan-out separately from HTTP RPS, keep gateways replaceable, and put durable state in backend stores/brokers. I define heartbeat, backpressure, reconnect/resume, ordering, and graceful draining. Events are routed to the gateway owning the connection rather than broadcast blindly to every node.”

## References

- [RFC 6455 — WebSocket Protocol](https://www.rfc-editor.org/rfc/rfc6455.html)
- [RFC 8441 — Bootstrapping WebSockets with HTTP/2](https://www.rfc-editor.org/rfc/rfc8441.html)
