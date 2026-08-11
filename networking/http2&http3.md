# HTTP/2 and HTTP/3

HTTP/2 and HTTP/3 preserve HTTP semantics but change how messages are transported.

- HTTP/2: binary framing and multiplexed streams, commonly over TLS + TCP.
- HTTP/3: HTTP semantics over QUIC, which runs over UDP and provides secure multiplexed streams.

## Interview TL;DR

1. HTTP semantics do not change because the transport version changes.
2. HTTP/2 multiplexes many HTTP streams over one TCP connection.
3. TCP packet loss can stall all HTTP/2 streams sharing that TCP connection.
4. HTTP/3 uses QUIC streams, so loss on one stream does not create the same connection-wide transport head-of-line blocking.
5. QUIC integrates TLS 1.3-style secure handshake behavior and supports connection migration.
6. Fewer connections can improve efficiency but makes per-connection flow-control and failure behavior important.
7. HTTP/3 is not automatically faster for every workload; path quality, RTT, loss, CPU, and infrastructure support matter.
8. Application retries, caching, and idempotency remain HTTP/application concerns.

## HTTP/2 Mental Model

```text
HTTP request A ─┐
HTTP request B ─┼─ HTTP/2 streams
HTTP request C ─┘
        ↓
one TCP connection
```

HTTP/2 adds:

- binary framing;
- multiplexing;
- header compression (HPACK);
- flow control;
- stream prioritization mechanisms.

## TCP Head-of-Line Effect

HTTP/2 streams are independent at the HTTP framing layer, but TCP exposes one ordered byte stream.

A lost TCP packet can delay delivery of later bytes for all multiplexed HTTP/2 streams on that connection.

## HTTP/3 Mental Model

```text
HTTP request A → QUIC stream A
HTTP request B → QUIC stream B
HTTP request C → QUIC stream C
            ↓
        QUIC over UDP
```

QUIC provides:

- secure connection establishment;
- independent reliable streams;
- flow control;
- congestion control;
- loss recovery;
- connection migration.

## HTTP/3 and Head-of-Line Blocking

Loss can still delay data **within the affected stream**, but another independent stream can continue when its required packets are available.

Application dependencies can still create their own head-of-line behavior.

## Connection Establishment

HTTP/3/QUIC can reduce connection-establishment latency, especially for resumed connections.

Be careful with early/0-RTT data: replayability matters for non-idempotent operations.

## Connection Migration

QUIC connection identifiers allow a connection to survive some network-path changes, useful for mobile clients moving between networks.

This is a transport capability, not an application session guarantee.

## Header Compression

HTTP/2 uses HPACK.

HTTP/3 uses QPACK, designed for QUIC's stream model.

Do not optimize protocol choice purely from header compression; payload and application work often dominate.

## Load Balancers and Proxies

Infrastructure must support the desired version end to end or terminate/translate it.

Example:

```text
client HTTP/3
    ↓
edge terminates QUIC
    ↓
HTTP/2 to origin
```

That is normal. “The website supports HTTP/3” does not imply every internal hop uses HTTP/3.

## When HTTP/3 Helps Most

Potentially valuable when:

- RTT is significant;
- networks are lossy/mobile;
- connection migration matters;
- many concurrent streams share a connection.

Measure rather than assume.

## Common Mistakes

- “HTTP/2 removes all head-of-line blocking”;
- “HTTP/3 is unreliable because it uses UDP”;
- “HTTP/3 changes REST semantics”;
- treating 0-RTT as safe for arbitrary writes;
- assuming origin traffic uses the same HTTP version as the client edge connection;
- choosing protocol version before checking proxy/CDN/client support.

## 2-Minute Interview Answer

> “HTTP/2 and HTTP/3 keep the same HTTP semantics. HTTP/2 multiplexes streams over one TCP connection, so TCP loss can stall the connection. HTTP/3 maps HTTP onto QUIC, which provides independent secure streams over UDP and avoids that connection-wide transport stall. I would treat HTTP/3 as a latency/resilience optimization to measure, not a new application architecture, and I would still design retries and idempotency at the HTTP/business layer.”

## References

- [RFC 9113 — HTTP/2](https://www.rfc-editor.org/rfc/rfc9113.html)
- [RFC 9114 — HTTP/3](https://www.rfc-editor.org/rfc/rfc9114.html)
- [RFC 9000 — QUIC](https://www.rfc-editor.org/rfc/rfc9000.html)
