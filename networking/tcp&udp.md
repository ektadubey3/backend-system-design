# TCP vs UDP

TCP and UDP are transport-layer protocols used to move data between applications over IP networks.

```text
Application Layer
HTTP, DNS, gRPC, WebSocket, Games
        |
        v
Transport Layer
TCP or UDP
        |
        v
Internet Layer
IP
        |
        v
Network Access Layer
Ethernet, Wi-Fi, Cellular
```

Both protocols use port numbers to deliver data to the correct application process, but they provide very different communication guarantees.

### TCP

TCP stands for **Transmission Control Protocol**.

It provides:

* Connection-oriented communication
* Reliable delivery
* Ordered delivery
* Duplicate detection
* Retransmission
* Flow control
* Congestion control
* Byte-stream communication

Common TCP-based protocols include:

* HTTP/1.1
* HTTP/2
* HTTPS
* WebSocket
* SSH
* SMTP
* FTP
* PostgreSQL connections
* MySQL connections
* Redis connections

### UDP

UDP stands for **User Datagram Protocol**.

It provides:

* Connectionless communication
* Message-oriented datagrams
* Low protocol overhead
* No built-in delivery guarantee
* No built-in ordering guarantee
* No built-in retransmission
* No built-in flow control
* No built-in congestion control

Common UDP-based protocols and systems include:

* DNS
* DHCP
* NTP
* Voice over IP
* Live media streaming
* Online games
* Service discovery
* Telemetry
* QUIC and HTTP/3

At a high level:

```text
TCP:
Correctness and reliability over raw speed

UDP:
Low overhead and application-controlled delivery behavior
```

TCP and UDP are not simply “slow versus fast.”

The correct choice depends on:

* Whether every message must arrive
* Whether ordering matters
* How much latency is acceptable
* Whether stale data remains useful
* Whether the application can tolerate loss
* Whether the application needs custom reliability
* Whether the traffic is unicast, multicast, or broadcast
* How the system behaves under congestion

---

## Why TCP and UDP?

TCP and UDP solve different transport problems.

Understanding both is essential when designing APIs, real-time platforms, streaming systems, service communication, gaming infrastructure, and distributed applications.

---

### 1. Different Applications Need Different Guarantees

A banking transaction must not disappear.

```text
Transfer $100
      |
      v
Every byte must arrive correctly
```

A live game-position update may become useless after a few milliseconds.

```text
Position at 10:00:00.100
Position at 10:00:00.120
Position at 10:00:00.140
```

If the first update is lost, retransmitting it later may provide no value because a newer position already exists.

TCP is usually appropriate when missing or reordered data would break correctness.

UDP is useful when timeliness is more important than perfect delivery.

---

### 2. TCP Simplifies Reliable Communication

Without TCP, an application that requires reliable delivery may need to implement:

* Sequence numbers
* Acknowledgments
* Retransmission
* Duplicate detection
* Reordering
* Timeout handling
* Receive windows
* Congestion behavior

TCP provides these mechanisms at the transport layer.

```text
Application
    |
    | Send byte stream
    v
TCP handles reliability
    |
    v
Network
```

This makes TCP a strong default for general backend communication.

---

### 3. UDP Minimizes Transport Overhead

UDP has a small header and does not establish a transport-layer connection before sending data.

```text
 Application
  |
  | Create datagram
  v
 UDP
  |
  | Send immediately
  v
 Network
```

This is useful for:

* Small request-response exchanges
* Real-time media
* Games
* Network discovery
* Telemetry
* Custom transport protocols

---

### 4. UDP Gives Applications More Control

TCP imposes specific reliability and ordering behavior.

UDP allows the application to decide:

* Which messages require acknowledgment
* Which messages may be dropped
* Which messages should be retransmitted
* Whether ordering matters
* How congestion should be handled
* Whether forward error correction is useful

Modern protocols such as QUIC use UDP as a foundation while implementing reliability, encryption, stream management, and congestion control in user space.

---

### 5. TCP Works Well for Stateful Sessions

TCP maintains a logical connection between two endpoints.

```text
Client IP:Port
        |
        | TCP Connection
        |
Server IP:Port
```

This supports:

* Persistent API connections
* Database sessions
* WebSocket sessions
* File transfers
* Secure shells
* Long-running application streams

---

### 6. UDP Supports Broadcast and Multicast Patterns

TCP is fundamentally point-to-point.

UDP can support:

* Unicast
* Broadcast
* Multicast

```text
                   ┌── Receiver A
Sender ──Multicast─┼── Receiver B
                   └── Receiver C
```

This can be useful for:

* Local network discovery
* Market-data distribution
* Media distribution
* Device coordination
* Routing protocols

Broadcast and multicast support depends on network infrastructure and is often restricted across public networks.

---

## Core Concepts

## 1. Transport Layer

TCP and UDP operate at the transport layer.

Their job is to deliver application data between processes running on networked hosts.

```text
Host A                                      Host B

Application A                              Application B
     |                                          ^
     v                                          |
TCP or UDP                                  TCP or UDP
     |                                          ^
     v                                          |
IP  ─────────────── Network ─────────────────> IP
```

IP delivers packets between hosts.

TCP and UDP use ports to deliver data to the correct application.

---

## 2. Ports

A port identifies an application endpoint on a host.

Examples:

| Protocol or Service | Common Port | Transport  |
| ------------------- | ----------: | ---------- |
| HTTP                |          80 | TCP        |
| HTTPS               |         443 | TCP        |
| HTTP/3              |         443 | UDP        |
| DNS                 |          53 | UDP or TCP |
| SSH                 |          22 | TCP        |
| PostgreSQL          |        5432 | TCP        |
| MySQL               |        3306 | TCP        |
| NTP                 |         123 | UDP        |

A connection or flow can be identified using a five-tuple:

```text
Source IP
Source Port
Destination IP
Destination Port
Transport Protocol
```

Example:

```text
192.0.2.10:53000
        |
        | TCP
        v
198.51.100.20:443
```

---

## 3. TCP Connection Establishment

TCP establishes a connection using the three-way handshake.

```text
Client                                  Server
   |                                       |
   |------------- SYN -------------------->|
   |                                       |
   |<--------- SYN + ACK ------------------|
   |                                       |
   |------------- ACK -------------------->|
   |                                       |
   |        Connection Established         |
```

### SYN

The client requests a connection and sends an initial sequence number.

### SYN-ACK

The server acknowledges the client and provides its own initial sequence number.

### ACK

The client acknowledges the server.

After the handshake, application data can be exchanged.

The handshake introduces additional latency before the first application message.

---

## 4. TCP Connection Termination

TCP commonly closes a connection through a four-step exchange.

```text
Client                                  Server
   |                                       |
   |------------- FIN -------------------->|
   |<------------ ACK ---------------------|
   |                                       |
   |<------------ FIN ---------------------|
   |------------- ACK -------------------->|
```

Each side closes its sending direction independently.

TCP also supports abrupt termination through a reset:

```text
RST
```

A reset indicates that the connection cannot continue normally.

---

## 5. TCP Byte Stream

TCP provides a continuous byte stream.

Suppose the sender writes:

```text
Write 1: "HELLO"
Write 2: "WORLD"
```

The receiver might read:

```text
Read 1: "HEL"
Read 2: "LOWOR"
Read 3: "LD"
```

TCP does not preserve application message boundaries.

The application must define framing.

Common framing strategies include:

### Fixed-Length Messages

```text
Every message = 64 bytes
```

### Length-Prefixed Messages

```text
[Payload Length][Payload]
```

Example:

```text
[0005][HELLO]
[0005][WORLD]
```

### Delimiter-Based Messages

```text
HELLO\n
WORLD\n
```

### Structured Protocol Framing

Protocols such as HTTP and TLS define their own framing rules.

---

## 6. UDP Datagrams

UDP is message-oriented.

If the sender sends two datagrams:

```text
Datagram 1: "HELLO"
Datagram 2: "WORLD"
```

The receiver processes them as separate messages.

UDP preserves datagram boundaries, although datagrams may:

* Arrive
* Be lost
* Be duplicated
* Arrive out of order

A single UDP datagram is not split into multiple application-level receives in the same way as a TCP byte stream.

---

## 7. Reliability

TCP provides reliable delivery while the connection remains valid and both endpoints continue operating.

It uses:

* Sequence numbers
* Acknowledgments
* Retransmission timers
* Checksums
* Duplicate detection

Example:

```text
Sender                               Receiver
   |                                    |
   |--------- Segment 1 --------------->|
   |<------------ ACK 1 ----------------|
   |                                    |
   |--------- Segment 2 ----X           |
   |                                    |
   |------ Retransmit Segment 2 ------->|
   |<------------ ACK 2 ----------------|
```

UDP does not automatically retransmit lost datagrams.

```text
Sender                               Receiver
   |                                    |
   |--------- Datagram 1 -------------> |
   |--------- Datagram 2 -----X         |
   |--------- Datagram 3 -------------> |
```

The receiver sees datagrams 1 and 3 unless the application adds recovery.

---

## 8. Ordered Delivery

TCP presents bytes to the application in order.

Suppose segments arrive as:

```text
Segment 1
Segment 3
Segment 2
```

TCP buffers segment 3 until segment 2 arrives.

```text
Network arrival:
1, 3, 2

Application delivery:
1, 2, 3
```

UDP delivers datagrams as received.

```text
Network arrival:
1, 3, 2

Application may observe:
1, 3, 2
```

The application must add sequence numbers if ordering matters.

---

## 9. Sequence Numbers

TCP assigns sequence numbers to bytes.

Conceptually:

```text
Segment A:
Bytes 1–1000

Segment B:
Bytes 1001–2000

Segment C:
Bytes 2001–3000
```

Sequence numbers help TCP detect:

* Missing bytes
* Duplicate segments
* Out-of-order segments

UDP itself does not include application sequence numbers.

A UDP application may add them:

```json
{
  "sequence": 1042,
  "timestamp": 1722000000,
  "payload": "..."
}
```

---

## 10. Acknowledgments

TCP receivers acknowledge received data.

```text
ACK 3001
```

Conceptually, this means:

```text
All bytes before 3001 have been received.
Send byte 3001 next.
```

UDP has no built-in acknowledgment mechanism.

An application may define its own:

```text
Client:
Message ID 123

Server:
ACK 123
```

---

## 11. Retransmission

TCP retransmits data when it determines that a segment was probably lost.

Loss may be detected through:

* Retransmission timeout
* Duplicate acknowledgments
* Selective acknowledgment information

UDP does not retransmit automatically.

This can be an advantage when late data is useless.

Example:

```text
Old voice packet lost

Better behavior:
Skip it and continue

Poor behavior:
Deliver it late after newer audio
```

---

## 12. Flow Control

TCP flow control prevents a sender from overwhelming a receiver.

The receiver advertises how much data it can currently accept.

```text
Sender                  Receiver

Send Window <---------- Receive Capacity
```

If the receiver's buffer is nearly full, the sender reduces how much unacknowledged data it sends.

UDP has no built-in flow control.

A UDP sender can transmit faster than the receiver can process, causing:

* Kernel buffer overflow
* Packet loss
* Memory pressure
* Increased application latency

---

## 13. Congestion Control

TCP attempts to avoid overwhelming the network.

It adjusts its sending rate based on signals such as:

* Packet loss
* Round-trip time
* Acknowledgment patterns
* Explicit congestion notification

Conceptually:

```text
Network Healthy
      |
      v
Increase Sending Rate

Packet Loss or Congestion
      |
      v
Reduce Sending Rate
```

UDP does not provide built-in congestion control.

Applications sending large volumes over UDP must implement congestion-safe behavior.

Ignoring congestion can degrade performance for the entire network.

---

## 14. Sliding Window

TCP can send multiple segments before waiting for acknowledgments.

```text
Send Window:

[Sent + ACKed][Sent, not ACKed][Allowed to Send][Not Allowed]
```

The window allows high throughput without requiring one acknowledgment after every segment.

Window size is influenced by:

* Receiver capacity
* Congestion state
* Network latency
* TCP configuration

---

## 15. Head-of-Line Blocking

TCP guarantees ordered delivery.

If an early segment is lost, later bytes cannot be delivered to the application until the missing segment is recovered.

```text
Segment 1: Received
Segment 2: Lost
Segment 3: Received
Segment 4: Received

Application receives:
Segment 1

Segments 3 and 4 wait for Segment 2
```

This is called transport-level head-of-line blocking.

UDP does not enforce ordering, so later datagrams can still be processed.

QUIC, built on UDP, supports multiple independent streams so loss in one stream does not block application data in another stream.

---

## 16. TCP Checksums

TCP includes a checksum for detecting corruption in:

* TCP header
* Payload
* Parts of the IP addressing information

If corruption is detected, the segment is discarded and eventually retransmitted.

UDP also includes a checksum.

However, checksum behavior and requirements differ between IPv4 and IPv6 implementations.

A checksum detects accidental corruption; it does not provide cryptographic integrity or authentication.

Use TLS, DTLS, QUIC, or application-layer cryptography when security is required.

---

## 17. UDP Header

UDP has a small header containing:

```text
Source Port
Destination Port
Length
Checksum
```

Conceptually:

```text
┌──────────────────┬──────────────────┐
│ Source Port      │ Destination Port │
├──────────────────┼──────────────────┤
│ Length           │ Checksum         │
├──────────────────┴──────────────────┤
│ Payload                             │
└─────────────────────────────────────┘
```

The minimal header contributes to UDP's low overhead.

---

## 18. TCP Header

TCP carries additional control information.

Common fields include:

```text
Source Port
Destination Port
Sequence Number
Acknowledgment Number
Flags
Window Size
Checksum
Options
```

Conceptually:

```text
┌──────────────────┬──────────────────┐
│ Source Port      │ Destination Port │
├──────────────────┴──────────────────┤
│ Sequence Number                     │
├─────────────────────────────────────┤
│ Acknowledgment Number               │
├──────────┬───────────┬──────────────┤
│ Flags    │ Window    │ Checksum     │
├──────────┴───────────┴──────────────┤
│ Options                             │
├─────────────────────────────────────┤
│ Payload                             │
└─────────────────────────────────────┘
```

This additional information supports reliable and controlled delivery.

---

## 19. Packet Loss

Packet loss can result from:

* Network congestion
* Failing equipment
* Wireless interference
* Buffer overflow
* Routing changes
* Firewall policies
* Rate limiting

TCP usually responds by retransmitting and reducing its sending rate.

UDP leaves loss handling to the application.

Different applications may react differently:

```text
File Transfer:
Retransmit missing data

Voice Call:
Skip old audio packet

Game:
Use latest state update

Telemetry:
Accept occasional loss or batch later
```

---

## 20. Latency

TCP may add latency through:

* Connection setup
* Retransmission
* Ordered delivery
* Congestion control
* Buffering

UDP can send immediately, but using UDP does not guarantee low latency.

A UDP application can still have high latency because of:

* Network distance
* Server overload
* Oversized queues
* Application processing
* Packet fragmentation
* Poor routing
* Retransmission added by the application

Protocol choice is only one part of latency design.

---

## 21. Maximum Transmission Unit

The Maximum Transmission Unit, or MTU, is the largest network-layer packet that can travel over a link without fragmentation.

A common Ethernet MTU is:

```text
1500 bytes
```

Application payload capacity is lower after subtracting:

* IP headers
* TCP or UDP headers
* Encryption overhead
* Protocol-specific metadata

Large UDP datagrams may be fragmented at the IP layer.

If one fragment is lost, the entire original datagram may become unusable.

For reliable production behavior, UDP applications often keep datagrams below the path MTU.

---

## 22. Fragmentation

Consider a UDP datagram larger than the network path supports.

```text
Large UDP Datagram
        |
        v
IP Fragment 1
IP Fragment 2
IP Fragment 3
```

If fragment 2 is lost:

```text
Fragment 1: Received
Fragment 2: Lost
Fragment 3: Received

Result:
Entire datagram cannot be reconstructed
```

Fragmentation increases loss sensitivity and processing overhead.

Prefer application-level segmentation with bounded datagram sizes.

---

## 23. Connection State

TCP endpoints maintain connection state, including:

* Sequence numbers
* Receive windows
* Send windows
* Retransmission state
* Congestion state
* Connection timers

This consumes memory and CPU for every active connection.

UDP servers do not need transport-layer connection state, although the application may create logical session state.

```text
UDP Session State

Client ID
Authentication State
Last Sequence Number
Last Activity Time
Rate Limit State
```

Connectionless transport does not mean stateless application logic.

---

## 24. NAT and Firewalls

Network Address Translation devices and firewalls track traffic flows.

TCP flow state is relatively explicit because connections have:

* Handshakes
* Established states
* Close signals

UDP has no handshake, so network devices often expire UDP mappings after shorter idle periods.

UDP applications may need keepalive traffic to preserve mappings.

```text
Client behind NAT
      |
      | Periodic UDP Keepalive
      v
Public Server
```

Keepalive intervals should be chosen carefully to avoid unnecessary traffic and battery usage.

---

## 25. Security

Neither raw TCP nor raw UDP encrypts application data.

```text
TCP != Encryption
UDP != Encryption
```

Security can be added through:

### TCP

```text
Application
    |
    v
   TLS
    |
    v
   TCP
```

Examples:

* HTTPS
* Secure WebSockets
* Encrypted database connections

### UDP

```text
Application
    |
    v
DTLS or QUIC Security
    |
    v
   UDP
```

Authentication, encryption, and replay protection must be designed explicitly.

---

## Architecture

A backend platform may use both TCP and UDP for different workloads.

```text
                          ┌─────────────────────┐
                          │   Client Devices    │
                          │ Web / Mobile / IoT  │
                          └──────────┬──────────┘
                                     │
                  ┌──────────────────┴──────────────────┐
                  │                                     │
              TCP / TLS                             UDP / QUIC
                  │                                     │
       ┌──────────▼──────────┐              ┌───────────▼──────────┐
       │ TCP Load Balancer   │              │ UDP Load Balancer    │
       │ Connection Tracking │              │ Flow-Aware Routing   │
       └──────────┬──────────┘              └───────────┬──────────┘
                  │                                     │
       ┌──────────▼──────────┐              ┌───────────▼──────────┐
       │ API / Web Services  │              │ Real-Time Gateway    │
       │ HTTP / gRPC / WS    │              │ QUIC / Game / Media  │
       └──────────┬──────────┘              └───────────┬──────────┘
                  │                                     │
                  └──────────────────┬──────────────────┘
                                     │
                     ┌───────────────▼───────────────┐
                     │      Application Services     │
                     │ Auth / Orders / Presence      │
                     └───────────────┬───────────────┘
                                     │
                  ┌──────────────────┼──────────────────┐
                  │                  │                  │
          ┌───────▼───────┐  ┌───────▼──────┐  ┌────────▼───────┐
          │ Database      │  │ Cache        │  │ Event Broker   │
          │ TCP           │  │ TCP          │  │ TCP or Custom  │
          └───────────────┘  └──────────────┘  └────────────────┘
```

---

## TCP Service Architecture

A typical TCP-based backend request follows this path:

```text
Client
   |
   | TCP Handshake
   v
Load Balancer
   |
   | Persistent TCP Connection
   v
Application Server
   |
   ├── Cache
   ├── Database
   └── Downstream Services
```

### TCP Request Flow

1. The client resolves the server address.
2. The client starts a TCP handshake.
3. TLS may run after TCP connection establishment.
4. The client sends application data.
5. TCP divides the byte stream into segments.
6. IP transports packets across the network.
7. The server's TCP stack reorders and reconstructs the stream.
8. The application parses its protocol messages.
9. The server sends a response.
10. TCP handles acknowledgment and retransmission.
11. The connection may remain open for reuse.

---

## UDP Service Architecture

A UDP-based real-time service may look like this:

```text
Game Client
    |
    | UDP Datagrams
    v
UDP Load Balancer
    |
    v
Game Gateway
    |
    ├── Session Manager
    ├── State Engine
    ├── Rate Limiter
    └── Event System
```

### UDP Request Flow

1. The client creates an application datagram.
2. The client may attach a session ID and sequence number.
3. UDP sends the datagram without a handshake.
4. A UDP-aware load balancer routes the flow.
5. The server validates the source and session.
6. The application checks ordering or freshness.
7. The server processes or discards the message.
8. An optional response or acknowledgment is sent.
9. Missing datagrams are handled according to application policy.

---

## Hybrid Architecture

Many production systems use TCP and UDP together.

```text
                      Online Game

Authentication:
Client ──HTTPS/TCP──> Authentication Service

Game State:
Client ───UDP───────> Game Server

Chat:
Client ──WebSocket/TCP──> Chat Service

Purchases:
Client ──HTTPS/TCP──> Commerce Service

Voice:
Client ───UDP───────> Media Relay
```

Each transport is selected based on the workload's guarantees.

---

## Load Balancing Considerations

### TCP Load Balancing

TCP load balancers may operate at Layer 4.

```text
Client Connection
       |
       v
Layer 4 Load Balancer
       |
       v
Selected Backend
```

After selecting a backend, the connection generally remains associated with that backend.

Important considerations:

* Connection duration
* Uneven connection load
* Connection draining
* Source IP preservation
* Health checks
* Idle timeouts

### UDP Load Balancing

UDP has no transport connection, but load balancers often group packets into logical flows using the five-tuple.

```text
UDP Flow
Source IP + Source Port
Destination IP + Destination Port
Protocol
```

Important considerations:

* Flow affinity
* NAT rebinding
* Idle expiration
* Session identifiers
* Packet reordering
* Stateless versus stateful routing

---

## Comparison

## TCP vs UDP

| Category              | TCP                       | UDP                                 |
| --------------------- | ------------------------- | ----------------------------------- |
| Communication model   | Connection-oriented       | Connectionless                      |
| Data model            | Byte stream               | Datagrams                           |
| Connection handshake  | Required                  | Not required                        |
| Reliable delivery     | Yes                       | No built-in guarantee               |
| Ordered delivery      | Yes                       | No built-in guarantee               |
| Retransmission        | Built in                  | Application controlled              |
| Duplicate handling    | Built in                  | Application controlled              |
| Flow control          | Built in                  | Not built in                        |
| Congestion control    | Built in                  | Not built in                        |
| Header overhead       | Higher                    | Lower                               |
| Message boundaries    | Not preserved             | Preserved                           |
| Broadcast support     | No                        | Yes                                 |
| Multicast support     | No                        | Yes                                 |
| Head-of-line blocking | Yes                       | No transport-level ordering         |
| Common use            | APIs, files, databases    | Games, voice, DNS, telemetry        |
| Security              | Commonly TLS              | DTLS, QUIC, or application security |
| Best for              | Correct and complete data | Timely or custom-controlled data    |

---

## TCP vs UDP Reliability

| Requirement               | TCP                       | UDP                                |
| ------------------------- | ------------------------- | ---------------------------------- |
| Every message must arrive | Strong fit                | Must be implemented by application |
| Exact ordering required   | Strong fit                | Must be implemented by application |
| Duplicate protection      | Built in                  | Must be implemented by application |
| Late data is useless      | May retransmit stale data | Application can discard old data   |
| Partial reliability       | Not natively customizable | Can be designed by application     |

---

## TCP vs UDP Latency

| Latency Factor                    | TCP       | UDP                         |
| --------------------------------- | --------- | --------------------------- |
| Initial handshake                 | Yes       | No                          |
| Retransmission delay              | Yes       | Only if application adds it |
| Ordered delivery delay            | Yes       | No                          |
| Congestion response               | Built in  | Application responsibility  |
| Warm persistent connection        | Efficient | Efficient                   |
| Lowest theoretical setup overhead | Higher    | Lower                       |

UDP may reduce transport setup overhead, but this does not automatically make the entire application faster.

---

## TCP vs QUIC

QUIC uses UDP as its underlying transport but implements features commonly associated with TCP and TLS.

| Category                           | TCP with TLS       | QUIC       |
| ---------------------------------- | ------------------ | ---------- |
| Underlying transport               | TCP                | UDP        |
| Encryption                         | TLS layer          | Integrated |
| Multiplexed streams                | Via HTTP/2         | Native     |
| Stream-level head-of-line blocking | Possible           | Reduced    |
| Connection migration               | Difficult          | Supported  |
| Handshake design                   | TCP plus TLS       | Combined   |
| HTTP version                       | HTTP/1.1 or HTTP/2 | HTTP/3     |

QUIC does not behave like raw UDP from an application's perspective.

It is a reliable, secure transport built on UDP.

---

## TCP vs WebSocket

WebSocket is an application protocol that usually runs over TCP.

```text
WebSocket
    |
    v
   TCP
```

| Category                    | TCP                | WebSocket                      |
| --------------------------- | ------------------ | ------------------------------ |
| Layer                       | Transport          | Application                    |
| Message framing             | No                 | Yes                            |
| Browser API                 | No raw TCP access  | Native WebSocket API           |
| Bidirectional communication | Yes                | Yes                            |
| Reliability                 | TCP reliability    | Inherited from TCP             |
| Best use                    | Building protocols | Real-time browser applications |

These are not direct alternatives.

WebSocket uses TCP to provide persistent application messaging.

---

## TCP vs HTTP

HTTP commonly runs over TCP, although HTTP/3 runs over QUIC and UDP.

```text
HTTP/1.1 or HTTP/2
        |
        v
       TCP
```

```text
HTTP/3
   |
   v
 QUIC
   |
   v
  UDP
```

TCP transports bytes.

HTTP defines application semantics such as:

* Methods
* Headers
* Status codes
* Request bodies
* Responses
* Caching

---

## UDP vs WebRTC

WebRTC commonly uses UDP for low-latency media, while adding higher-level protocols for:

* Encryption
* NAT traversal
* Congestion control
* Media transport
* Packet feedback

```text
Audio / Video
      |
      v
RTP / SRTP and Related Protocols
      |
      v
     UDP
```

WebRTC is much more than raw UDP.

---

## Real-World Example: Multiplayer Game Platform

Consider a multiplayer action game with these requirements:

* Player authentication
* Matchmaking
* Frequent position updates
* Shooting events
* Text chat
* Purchases
* Match result storage
* Voice communication

No single protocol is ideal for every operation.

---

### Protocol Selection

| Feature             | Transport Choice                      | Reason                                              |
| ------------------- | ------------------------------------- | --------------------------------------------------- |
| Authentication      | TCP through HTTPS                     | Must be reliable and secure                         |
| Matchmaking         | TCP through HTTPS or WebSocket        | Reliable state transitions                          |
| Player positions    | UDP                                   | Latest state matters most                           |
| Shooting events     | Reliable UDP or TCP depending on game | Event loss may be unacceptable                      |
| Text chat           | TCP through WebSocket                 | Messages must arrive in order                       |
| Purchases           | TCP through HTTPS                     | Correctness is critical                             |
| Match results       | TCP or message broker                 | Must be persisted                                   |
| Voice communication | UDP                                   | Low latency is more important than perfect delivery |

---

### Architecture

```text
                         ┌───────────────────┐
                         │    Game Client    │
                         └─────────┬─────────┘
                                   │
             ┌─────────────────────┼─────────────────────┐
             │                     │                     │
         HTTPS/TCP              UDP State           WebSocket/TCP
             │                     │                     │
   ┌─────────▼────────┐  ┌─────────▼────────┐  ┌─────────▼────────┐
   │ API Gateway      │  │ Game Gateway     │  │ Chat Gateway     │
   │ Auth / Purchase  │  │ Session Routing  │  │ Rooms / Presence │
   └─────────┬────────┘  └─────────┬────────┘  └─────────┬────────┘
             │                     │                     │
             │           ┌─────────▼────────┐            │
             │           │ Game Simulation  │            │
             │           │ Authoritative    │            │
             │           │ State Engine     │            │
             │           └─────────┬────────┘            │
             │                     │                     │
             └─────────────────────┼─────────────────────┘
                                   │
                       ┌───────────▼───────────┐
                       │ Database / Event Bus  │
                       └───────────────────────┘
```

---

### Position Update

The client sends frequent position updates over UDP.

```json
{
  "type": "player_position",
  "sessionId": "match-987",
  "playerId": "player-123",
  "sequence": 15042,
  "timestamp": 1722000000123,
  "position": {
    "x": 42.8,
    "y": 10.4,
    "z": 7.1
  },
  "velocity": {
    "x": 1.2,
    "y": 0.0,
    "z": -0.4
  }
}
```

Suppose packets arrive in this order:

```text
Sequence 15042
Sequence 15044
Sequence 15043
```

The server may discard `15043` because a newer state has already been processed.

```text
Accepted:
15042
15044

Discarded as stale:
15043
```

Retransmitting stale position data would add little value.

---

### Critical Gameplay Event

A shooting event may need stronger delivery guarantees.

```json
{
  "type": "weapon_fired",
  "eventId": "event-456",
  "sequence": 8842,
  "weaponId": "weapon-8",
  "direction": {
    "x": 0.8,
    "y": 0.1,
    "z": 0.5
  }
}
```

The application may use reliable UDP behavior:

```text
Client                              Server
   |                                   |
   |------ Event 8842 ---------------->|
   |                                   |
   |<--------- ACK 8842 ---------------|
```

If no acknowledgment arrives before a short timeout:

```text
Retransmit Event 8842
```

The event ID allows the server to reject duplicates.

```text
First Event 8842:
Process

Duplicate Event 8842:
Acknowledge but do not process again
```

---

### Chat Message

Text chat uses WebSocket over TCP.

```json
{
  "type": "chat_message",
  "messageId": "msg-789",
  "matchId": "match-987",
  "text": "Defend the north entrance."
}
```

Chat messages should:

* Arrive reliably
* Remain ordered
* Be persisted when required
* Avoid duplication

TCP is well suited to these requirements.

---

### Purchase Request

A purchase is sent over HTTPS.

```http
POST /v1/purchases HTTP/1.1
Content-Type: application/json
Idempotency-Key: purchase-attempt-123
```

```json
{
  "playerId": "player-123",
  "itemId": "item-legendary-8"
}
```

Correctness is more important than saving a small amount of transport overhead.

The server uses an idempotency key to prevent duplicate purchases after retries.

---

### Handling Packet Loss

Suppose the client sends:

```text
Position 101
Position 102
Position 103
Position 104
```

Packet `102` is lost.

```text
Server receives:
101, 103, 104
```

The server does not request retransmission because positions `103` and `104` supersede it.

For a critical event:

```text
Fire Event 201
```

If it is lost, the application retries because the event remains meaningful.

This demonstrates partial reliability:

```text
State Updates:
Loss tolerated

Critical Events:
Acknowledged and retransmitted
```

---

### Authoritative Server

The server should remain authoritative for important game state.

The client reports intent:

```text
Move forward
Fire weapon
Use ability
```

The server validates:

* Player identity
* Movement speed
* Collision rules
* Weapon cooldown
* Ammunition
* Match state

Never trust client-provided game outcomes.

```text
Client:
"I hit player B"

Server:
Validates trajectory and state before accepting
```

---

## Best Practices

## 1. Choose Based on Application Semantics

Do not choose UDP only because it is described as fast.

Ask:

* Must all data arrive?
* Must it arrive in order?
* Does old data remain useful?
* Can duplicates occur safely?
* Can the application implement reliability?
* Is browser support required?
* Is multicast needed?

Choose the protocol based on behavior, not reputation.

---

## 2. Use TCP as the General Default

TCP is a strong default for:

* Public APIs
* Database communication
* Financial operations
* Authentication
* File transfer
* Administrative commands
* Business workflows

Use UDP when the application's requirements justify its additional complexity.

---

## 3. Add Message Framing Over TCP

TCP does not preserve message boundaries.

Every custom TCP protocol should define framing.

Example length-prefixed frame:

```text
┌──────────────┬────────────────────┐
│ Length: 1024 │ Payload            │
└──────────────┴────────────────────┘
```

Validate declared lengths before allocating memory.

---

## 4. Keep UDP Datagrams Small

Avoid depending on IP fragmentation.

Design datagrams to fit safely within the expected path MTU after accounting for:

* IP headers
* UDP headers
* Encryption
* Application metadata

Use application-level segmentation for larger messages.

---

## 5. Implement Congestion Control for UDP

High-volume UDP applications must behave responsibly under congestion.

Possible strategies include:

* Adaptive send rates
* Packet-loss feedback
* Round-trip-time measurement
* Bandwidth estimation
* Exponential backoff
* Pacing
* Bounded retries

Raw unrestricted UDP traffic can overload networks.

---

## 6. Add Sequence Numbers When Needed

For UDP applications, sequence numbers help detect:

* Loss
* Duplication
* Reordering
* Stale messages

Example:

```json
{
  "sessionId": "session-123",
  "sequence": 4501,
  "payload": {}
}
```

Sequence numbers may be scoped per:

* Connection
* Session
* Message stream
* Entity
* Channel

---

## 7. Add Idempotency for Important Operations

A retry can produce duplicates at the application layer.

Use:

* Message IDs
* Request IDs
* Idempotency keys
* Deduplication windows

Example:

```text
Operation ID: order-creation-789
```

The server should process the logical operation only once.

---

## 8. Set Timeouts

Network operations should not wait indefinitely.

Use timeouts for:

* TCP connection establishment
* TLS handshake
* Reads
* Writes
* Idle connections
* UDP responses
* Application acknowledgments

Timeouts should reflect actual latency budgets.

---

## 9. Reuse TCP Connections

Persistent TCP connections avoid repeated:

* Three-way handshakes
* TLS handshakes
* Socket setup
* Congestion-window warm-up

Use connection pooling for:

* Databases
* HTTP clients
* gRPC channels
* Internal services

Do not create a new connection for every small operation unless required.

---

## 10. Use Backpressure

For TCP applications, TCP flow control alone may not protect the whole service.

Also use:

* Bounded application queues
* Request limits
* Concurrency limits
* Stream limits
* Load shedding

For UDP, explicitly enforce receive and send limits.

---

## 11. Protect Against Slow Clients

Slow TCP clients can hold:

* Connections
* Memory buffers
* Worker capacity
* File descriptors

Use:

* Write deadlines
* Idle timeouts
* Buffer limits
* Per-client quotas
* Connection limits

Disconnect clients that cannot keep up.

---

## 12. Authenticate Application Sessions

A TCP connection or UDP source address does not prove identity.

Use cryptographically secure authentication.

Do not rely solely on:

* Source IP
* Source port
* Connection existence
* Client-provided user IDs

UDP source addresses can be spoofed in some environments.

---

## 13. Encrypt Sensitive Traffic

Use:

* TLS over TCP
* DTLS where appropriate
* QUIC for secure UDP-based transport
* Application-layer encryption when necessary

Never transmit credentials or private business data through raw plaintext protocols.

---

## 14. Defend Against UDP Amplification

UDP services may be abused in reflection or amplification attacks.

An attacker can spoof a victim's address:

```text
Attacker
   |
   | Small Request with Victim Source IP
   v
UDP Server
   |
   | Large Response
   v
Victim
```

Mitigations include:

* Response-size limits
* Request validation
* Authentication
* Address validation
* Rate limiting
* Avoiding large unauthenticated responses

---

## 15. Design for NAT Rebinding

A UDP client's public source address or port may change.

Do not bind long-lived identity solely to:

```text
IP address + port
```

Use secure session identifiers and validate migration when supported.

QUIC includes mechanisms for connection migration.

---

## 16. Monitor Network-Level Metrics

Track:

* Connection count
* Connection establishment failures
* Retransmissions
* Round-trip time
* Packet loss
* Out-of-order packets
* Duplicate packets
* Send and receive buffer usage
* Reset count
* Timeout count
* Datagram rate
* Bytes transferred
* Handshake latency

Application latency alone may hide transport problems.

---

## 17. Handle Graceful Shutdown

For TCP services:

1. Stop accepting new connections.
2. Mark the instance unhealthy.
3. Drain existing requests.
4. Close idle connections.
5. Close active connections after a deadline.

For UDP services:

1. Stop accepting new logical sessions.
2. Complete critical in-flight operations.
3. Redirect or migrate session state where supported.
4. Shut down after a bounded drain period.

---

## 18. Test Under Realistic Network Conditions

Test with:

* Packet loss
* Reordering
* Duplication
* Variable latency
* Limited bandwidth
* Connection resets
* NAT changes
* Server restarts
* Burst traffic

A protocol that works on localhost may fail under mobile or cross-region conditions.

---

## 19. Avoid Custom Reliable UDP Without Need

Building a reliable UDP protocol is complex.

You may need to implement:

* Handshakes
* Encryption
* Congestion control
* Retransmission
* Stream multiplexing
* Flow control
* Path MTU discovery
* NAT traversal
* Connection migration

Consider existing protocols and libraries such as QUIC before creating a custom transport.

---

## 20. Separate Transport From Business Logic

Keep transport-specific code isolated.

```text
TCP or UDP Handler
        |
        v
Message Validation
        |
        v
Application Service
        |
        v
Domain Logic
```

This makes it easier to:

* Test business rules
* Change transports
* Add alternative clients
* Reuse application logic

---

## Common Mistakes

## 1. Assuming UDP Is Always Faster

UDP has less built-in overhead, but application-level retries, packet loss, poor pacing, and fragmentation can make a UDP system slower or less reliable.

Measure end-to-end performance.

---

## 2. Assuming TCP Sends Messages

TCP sends a byte stream, not application messages.

Without framing, the receiver cannot reliably identify message boundaries.

---

## 3. Assuming One Write Equals One Read

A single TCP `write()` may arrive through multiple `read()` calls.

Multiple writes may also be combined into one read.

Applications must parse the stream correctly.

---

## 4. Ignoring Partial TCP Writes

A socket write may accept fewer bytes than requested.

Production code must continue writing until:

* The complete message is sent
* An error occurs
* A deadline expires

---

## 5. Sending Oversized UDP Datagrams

Large datagrams may be fragmented.

Loss of one fragment can invalidate the entire datagram.

Keep packets bounded.

---

## 6. Implementing UDP Without Congestion Control

Sending UDP at an unrestricted rate can overwhelm:

* Receivers
* Routers
* Links
* Shared infrastructure

Implement pacing and congestion response.

---

## 7. Retrying Every UDP Packet

Some updates become stale quickly.

Retransmitting old voice, video, telemetry, or position packets may increase congestion without improving the user experience.

Retry only data that remains valuable.

---

## 8. Assuming TCP Prevents Application Duplicates

TCP prevents duplicate bytes within one connection.

It does not prevent duplicate business operations caused by:

* Client retries
* Reconnection
* Load balancer retries
* Timeout ambiguity

Use application-level idempotency.

---

## 9. Using Connection State as Authentication

A network connection does not prove authorization.

Authenticate identities and authorize every protected action.

---

## 10. Ignoring TCP Head-of-Line Blocking

One lost segment can delay later application data on the same TCP connection.

Consider multiple connections or a multiplexed transport such as QUIC when independent streams are important.

---

## 11. Creating Too Many TCP Connections

Every connection consumes:

* File descriptors
* Memory
* Kernel state
* Load balancer capacity
* TLS resources

Reuse connections and enforce limits.

---

## 12. Ignoring Idle Timeouts

Load balancers, NAT devices, proxies, and firewalls may close inactive flows.

Configure heartbeats and timeouts consistently across the entire path.

---

## 13. Trusting UDP Source Addresses

UDP source addresses may be spoofed.

Do not perform sensitive actions based only on the packet's claimed source.

Use authentication and address validation.

---

## 14. Using Raw UDP for Critical Data Without Recovery

If every event must arrive, raw UDP alone is insufficient.

Add reliability or use TCP, QUIC, or another appropriate protocol.

---

## 15. Choosing One Protocol for the Entire System

A system may need different transports for different operations.

Example:

```text
Authentication: TCP
Purchases: TCP
Live positions: UDP
Voice: UDP
Chat: TCP
```

Hybrid designs are often more appropriate than forcing every workload through one transport.

---

## Interview Questions

### 1. What is the main difference between TCP and UDP?

TCP provides connection-oriented, reliable, ordered byte-stream delivery. UDP provides connectionless datagrams without built-in delivery or ordering guarantees.

---

### 2. Why is TCP considered reliable?

TCP uses sequence numbers, acknowledgments, retransmissions, duplicate detection, flow control, and ordered delivery to reconstruct the transmitted byte stream.

---

### 3. When would you choose UDP over TCP?

Choose UDP when low latency, multicast, message boundaries, or custom delivery behavior matters more than guaranteed ordered delivery, such as gaming, voice, live media, DNS, or telemetry.

---

### 4. What is TCP head-of-line blocking?

When a TCP segment is lost, later bytes are held back from the application until the missing segment is retransmitted and received, even if later segments have already arrived.

---

### 5. Does UDP guarantee lower latency than TCP?

No. UDP removes connection setup and built-in retransmission overhead, but total latency still depends on network conditions, congestion control, packet loss, application processing, and protocol design.

---

## Key Takeaways

1. **TCP provides reliable, ordered byte-stream communication and is the standard choice for APIs, databases, file transfers, authentication, and correctness-sensitive backend operations.**

2. **UDP provides lightweight, message-oriented communication that is useful when timeliness, multicast, or application-controlled reliability matters more than guaranteed delivery.**

3. **Protocol selection should be based on application semantics: many production systems use TCP for critical workflows and UDP or UDP-based protocols for real-time, latency-sensitive traffic.**
