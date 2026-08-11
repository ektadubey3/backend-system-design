# TCP vs UDP

TCP and UDP are transport-layer protocols with different guarantees. They are not simply “slow vs fast.”

## Interview TL;DR

1. TCP provides a reliable, ordered **byte stream** with flow control and congestion control.
2. UDP provides independent datagrams with no built-in delivery, ordering, retransmission, flow-control, or congestion-control guarantee.
3. Applications using UDP can implement the reliability/ordering semantics they need.
4. QUIC uses UDP as a substrate but adds secure connections, reliable streams, loss recovery, flow control, and congestion control.
5. TCP reliability does not make a business operation reliable or idempotent.
6. TCP has connection-level head-of-line effects; HTTP/3/QUIC gives independent streams at the transport layer.
7. Choose from application semantics, not packet-header size alone.

## Mental Model

### TCP

```text
application writes bytes
        ↓
TCP reliable ordered byte stream
        ↓
IP network
```

The receiver gets an ordered byte stream, not application messages. Message framing belongs to the application protocol.

### UDP

```text
application sends datagram
        ↓
UDP datagram
        ↓
IP network
```

Datagrams may be lost, duplicated, or reordered.

## TCP Properties

TCP includes:

- connection establishment;
- sequence numbers;
- acknowledgements;
- retransmission;
- flow control;
- congestion control;
- ordered delivery.

Use TCP when the application protocol benefits from a reliable stream and does not need to reimplement transport behavior.

Common examples:

- HTTP/1.1;
- HTTP/2;
- WebSocket over its traditional TCP transport;
- database connections;
- SSH.

## UDP Properties

UDP is useful when application/transport design needs datagrams or specialized loss/ordering behavior.

Examples:

- DNS commonly uses UDP for many queries, with TCP also used when required;
- real-time media stacks;
- games;
- QUIC.

Do not say “UDP is for anything where packet loss is acceptable.” QUIC demonstrates that reliable application transports can be built over UDP.

## Head-of-Line Blocking

TCP exposes one ordered byte stream.

If a packet needed for earlier bytes is lost, later bytes cannot be delivered to the application until the missing data is recovered.

HTTP/2 multiplexes many HTTP streams over one TCP connection, but TCP loss can stall progress across those streams.

QUIC provides independent reliable streams, reducing this connection-wide transport head-of-line effect.

## Flow Control vs Congestion Control

### Flow control

Protects the receiver from a sender that transmits faster than the receiver can consume.

### Congestion control

Protects the network path from persistent overload.

UDP itself does not provide these mechanisms; responsible higher-level protocols may.

## Connection Cost

TCP connection establishment costs network round trips. TLS may add handshake work on top.

Connection reuse matters for latency and resource efficiency.

See:

- [Keep-Alive](keep-alive.md)
- [Connection Pooling](connection-pooling.md)

## Failure Semantics

A TCP write succeeding locally does not prove the remote application committed the operation.

Example:

```text
client sends payment
server commits payment
connection breaks before response
```

The client has an uncertain outcome.

Solve this at the application level with idempotency/status lookup—not by relying on TCP reliability.

## Choosing TCP or UDP

Ask:

- reliable ordered stream or datagrams?
- is stale data still useful?
- does the higher-level protocol already implement recovery?
- do we need connection migration?
- what does congestion behavior look like?
- what does the infrastructure support?

Most application developers choose an application protocol (HTTP, QUIC, gRPC, media protocol), not raw TCP/UDP directly.

## Common Mistakes

- “TCP is slow, UDP is fast”;
- assuming one TCP write equals one application message;
- assuming TCP prevents duplicate business operations;
- saying UDP always loses data;
- forgetting DNS can use TCP;
- describing QUIC as “unreliable because it uses UDP.”

## 2-Minute Interview Answer

> “TCP gives me a reliable ordered byte stream with flow and congestion control; UDP gives me datagrams and leaves those semantics to the higher-level protocol. I would normally choose the application protocol first. For example, HTTP/3 uses QUIC over UDP but still provides reliable streams. I also separate transport reliability from business idempotency: a connection failure after the server commits can still leave the client uncertain.”

## References

- [RFC 9293 — Transmission Control Protocol](https://www.rfc-editor.org/rfc/rfc9293.html)
- [RFC 768 — User Datagram Protocol](https://www.rfc-editor.org/rfc/rfc768.html)
- [RFC 9000 — QUIC](https://www.rfc-editor.org/rfc/rfc9000.html)
