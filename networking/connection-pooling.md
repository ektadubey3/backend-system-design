# Connection Pooling

A connection pool reuses and **bounds** connections to a downstream system.

The important insight is that a pool is not only a latency optimization—it is also an **admission-control and capacity boundary**.

## Interview TL;DR

1. Reusing connections avoids repeated DNS/TCP/TLS/auth/session setup.
2. The maximum pool size should follow downstream sustainable concurrency, not application request concurrency.
3. When the pool is full, requests queue; pool-wait latency is part of request latency.
4. Increasing the pool can reduce waiting only until the downstream saturates; after that it may reduce throughput and worsen p99.
5. Multiply per-instance pool size by application instance count.
6. Bound pool wait time/queue length so overload fails predictably.
7. Validate/recycle stale connections with sensible idle/max lifetime.
8. HTTP/2/gRPC multiplexing means “connection pool size” has different semantics from a SQL pool.
9. Database transaction pooling can differ from session pooling and may break session-scoped assumptions.
10. Monitor pool utilization, waiters, acquire latency, connection errors, and downstream saturation together.

## Why Pool?

Without reuse:

```text
request
  ↓
DNS
  ↓
TCP
  ↓
TLS
  ↓
authentication/session init
  ↓
operation
  ↓
close
```

With a pool:

```text
request
  ↓
borrow/reuse connection
  ↓
operation
  ↓
return
```

## Pool as Admission Control

Suppose the DB can efficiently process ~80 concurrent queries.

A pool of 1,000 does not create 1,000× capacity.

It can create:

```text
too many active queries
→ context switching / lock pressure / memory
→ worse latency
```

A bounded pool protects the downstream.

## Pool Wait Queue

When all connections are busy:

```text
incoming request
      ↓
pool wait queue
      ↓
connection available
```

Define:

- acquire timeout;
- max queue/waiters;
- cancellation;
- priority where needed.

Do not let requests wait longer than their end-to-end deadline.

## Fleet Multiplication

```text
50 application instances
× 30 DB connections each
= 1,500 possible DB connections
```

Autoscaling can surprise the database.

Use a fleet-wide budget or an external pooler when appropriate.

## Database Pools

A SQL connection often represents server-side session/process/thread resources and transaction/session state.

Consider:

- max connections;
- transaction duration;
- prepared statements;
- temp/session state;
- failover behavior.

External poolers can provide session or transaction pooling with different semantics.

## HTTP Client Pools

HTTP/1.1 often uses a pool of persistent connections per destination.

For HTTP/2:

```text
one connection
→ many concurrent streams
```

so stream limits and channel distribution matter as much as connection count.

## gRPC Channels

A gRPC channel may multiplex many RPCs over HTTP/2 connections.

Long-lived channels can pin traffic to existing backends depending on resolver/load-balancer behavior.

## Idle and Max Lifetime

Recycle connections to handle:

- server restart;
- certificate/credential rotation;
- network changes;
- load balancing;
- stale NAT/firewall state.

Avoid synchronization where every connection expires at once; jitter can help.

## Health Validation

Strategies:

- validate on borrow only when necessary;
- observe I/O failures;
- background health/lifetime management.

A validation query on every borrow can add avoidable database load.

## Backpressure

Pool exhaustion should be an explicit signal.

If downstream is saturated, fail/shed/queue within a bound instead of opening unlimited new connections.

## Metrics

Track:

```text
active
idle
max
waiters
acquire p95/p99
timeouts
connection creation rate
connection errors
downstream CPU/locks/I/O
```

Pool metrics without downstream metrics are incomplete.

## Common Mistakes

- pool size = number of request threads;
- increasing pool whenever acquire wait rises;
- ignoring fleet multiplication;
- unbounded wait queue;
- no acquire timeout;
- using SQL pool mental model for HTTP/2/gRPC;
- max lifetime synchronized across all connections.

## 2-Minute Interview Answer

> “I treat the connection pool as downstream admission control. I reuse connections to avoid setup cost, but size the active pool from the database/service's sustainable concurrency, not app request count. I account for pool size across the whole fleet, bound acquire wait by the request deadline, recycle stale connections, and monitor pool wait together with downstream saturation before increasing capacity.”

## References

- [PostgreSQL — Resource Consumption / Connections](https://www.postgresql.org/docs/current/runtime-config-connection.html)
- [gRPC Core Concepts — Channels](https://grpc.io/docs/what-is-grpc/core-concepts/)
