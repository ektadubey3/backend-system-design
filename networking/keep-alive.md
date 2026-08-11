# Keep-Alive and Persistent Connections

Connection reuse avoids paying connection-establishment cost for every operation.

Be precise about the term **keep-alive**: HTTP persistent connections, TCP keepalive probes, gRPC/HTTP2 pings, and application heartbeats are related but different mechanisms.

## Interview TL;DR

1. HTTP/1.1 uses persistent connections by default unless the connection is closed.
2. HTTP/2 and HTTP/3 multiplex multiple streams over a connection; reuse is fundamental to their efficiency.
3. TCP keepalive probes detect dead peers at the transport/socket level; they are not the same as HTTP persistent connections.
4. Application heartbeats can detect semantic/session liveness faster than OS TCP keepalive settings.
5. Idle timeout mismatches across client, proxy, load balancer, NAT, and server cause resets on reused connections.
6. Long connection lifetime improves reuse but can create backend affinity and stale DNS/load-balancing distribution.
7. Draining must handle persistent connections deliberately.
8. Connection reuse saves handshake/CPU/latency but consumes file descriptors, memory, NAT state, and backend capacity.
9. A reused connection can become stale after remote restart/network change; clients must handle reconnect.
10. Keep-alive does not replace connection pooling or request concurrency limits.

## HTTP/1.1 Persistent Connections

Simplified:

```text
TCP + TLS establishment
   ↓
request 1 → response 1
request 2 → response 2
request 3 → response 3
   ↓
close / idle timeout
```

This avoids repeated TCP/TLS setup.

## HTTP/2 / HTTP/3

Multiple concurrent streams can share a connection.

```text
connection
 ├─ stream A
 ├─ stream B
 └─ stream C
```

This is different from HTTP/1.1 sequential reuse.

## TCP Keepalive vs HTTP Keep-Alive

### HTTP persistent connection

Application/protocol connection reuse.

### TCP keepalive

OS/socket-level probes after inactivity to detect dead peers.

TCP keepalive intervals are often much longer than an application wants to wait, so do not rely on it as the only failure detector.

## Application Heartbeat

WebSocket/realtime applications may exchange:

```text
ping / pong
heartbeat event
```

to detect dead sessions and refresh presence.

Heartbeat frequency is a capacity decision.

## Idle Timeout Mismatch

Example:

```text
client assumes connection valid for 120s
load balancer closes after 60s
client reuses at 90s
→ reset/failure
```

Coordinate or robustly handle:

- client idle timeout;
- proxy timeout;
- server timeout;
- NAT/firewall timeout.

## Maximum Lifetime

Connections may be recycled to:

- rebalance traffic;
- rotate certificates/credentials;
- avoid stale network state;
- respect infrastructure policy.

Too-short lifetime destroys reuse; too-long lifetime can pin traffic.

## Draining

When backend is removed:

```text
stop new connections/requests
      ↓
allow active work to finish
      ↓
signal/close long-lived connections deliberately
```

HTTP/2/gRPC and WebSockets need special care because connections can live far longer than one request.

## Resource Limits

Persistent connections consume:

- file descriptors;
- socket memory;
- TLS/session state;
- load-balancer connection tables;
- NAT ports/state;
- backend concurrency.

Estimate concurrent connections separately from RPS.

## Common Mistakes

- “without keep-alive every modern HTTP request always opens TCP”;
- TCP keepalive confused with HTTP connection reuse;
- no handling for stale reused sockets;
- idle timeout mismatch across proxy layers;
- infinite connection lifetime causing poor rebalancing;
- heartbeats so frequent they become major baseline traffic.

## 2-Minute Interview Answer

> “I reuse connections to avoid repeated TCP/TLS setup, but I distinguish HTTP persistence from TCP keepalive and application heartbeats. I coordinate or tolerate idle-timeout differences across clients, proxies and servers, bound connection lifetime/resources, and design graceful draining. For HTTP/2/gRPC, long-lived multiplexed channels can pin traffic, so connection lifecycle affects load balancing.”

## References

- [RFC 9112 — HTTP/1.1](https://www.rfc-editor.org/rfc/rfc9112.html)
- [RFC 9293 — TCP](https://www.rfc-editor.org/rfc/rfc9293.html)
