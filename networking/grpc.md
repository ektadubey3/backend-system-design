# gRPC

gRPC is a typed RPC framework commonly used for service-to-service communication. It typically uses Protocol Buffers for contracts/serialization and HTTP/2 for transport.

Its interview value is in **deadlines, cancellation, schema evolution, streaming, load balancing, retries, and the danger of treating remote calls as cheap local function calls**.

## Interview TL;DR

1. A `.proto` contract defines services and messages; generated stubs improve type safety.
2. gRPC supports unary, server-streaming, client-streaming, and bidirectional-streaming RPCs.
3. Set explicit deadlines. gRPC clients may otherwise wait much longer than intended depending on language/API defaults.
4. Propagate cancellation/deadlines to downstream work.
5. A cancelled RPC does not roll back side effects already committed.
6. Retries require service configuration/policy and operation idempotency; do not blindly retry mutations.
7. Protocol Buffer field numbers are part of wire compatibility—do not reuse removed field numbers.
8. One HTTP/2 connection/channel may carry many concurrent RPC streams; connection and load-balancing behavior matters.
9. Browser support usually requires gRPC-Web or another gateway pattern.
10. gRPC is strongest for controlled typed service ecosystems, not automatically for every public API.

## Contract

```proto
syntax = "proto3";

service OrderService {
  rpc GetOrder(GetOrderRequest) returns (Order);
  rpc WatchOrder(WatchOrderRequest) returns (stream OrderEvent);
}
```

Generated stubs make request/response types explicit.

## RPC Types

### Unary

```text
one request → one response
```

### Server streaming

```text
one request → stream of responses
```

### Client streaming

```text
stream of requests → one response
```

### Bidirectional streaming

```text
stream ↔ stream
```

Streaming introduces flow control, cancellation, and backpressure concerns.

## Deadlines

A remote call needs a time budget.

```text
incoming request deadline
       ↓
service A
       ↓ remaining budget
service B
       ↓
database
```

A deadline should be propagated rather than each layer inventing a full new timeout.

When a deadline expires, stop downstream work where possible.

## Cancellation

Client cancellation means the result is no longer needed.

Servers should cancel child operations.

Important:

> Cancellation does not undo writes or external side effects already performed.

Use idempotency/transaction semantics separately.

## Retries

Retry only:

- selected transient status codes;
- within deadline;
- with bounded attempts/backoff;
- when business semantics are safe.

An RPC method name such as `CreatePayment` is not magically safe because gRPC retries are available.

## Protocol Buffer Evolution

Safe evolution generally includes:

- add fields with new field numbers;
- preserve existing numbers/types where compatible;
- reserve removed field numbers/names.

Do not reuse an old field number for a new meaning.

## Channels and Connections

A gRPC channel represents connectivity to a target.

HTTP/2 multiplexes multiple RPCs over connections.

Watch:

- max concurrent streams;
- connection age/health;
- DNS/load-balancer behavior;
- long-lived connection affinity;
- backend draining.

Scaling backend instances may not rebalance existing long-lived channels immediately.

## Load Balancing

Possible patterns:

- L4/L7 proxy;
- client-side discovery/load balancing;
- service mesh.

Ensure health and draining behavior works for long-lived streams.

## Streaming Failure

For long streams define:

- heartbeat/keepalive policy;
- backpressure;
- reconnect;
- resume cursor/sequence;
- duplicate handling;
- ordering.

Do not assume reconnect continues exactly where the previous stream ended.

## gRPC vs REST

Use gRPC when:

- internal typed contracts matter;
- low-overhead binary serialization is useful;
- streaming is first-class;
- controlled clients can use generated stubs.

REST may be easier for broad public/web integrations.

## Common Mistakes

- no deadline;
- treating remote RPC like a local method;
- retrying non-idempotent mutations;
- reusing protobuf field numbers;
- ignoring channel/load-balancer affinity;
- assuming cancellation rolls back work;
- using gRPC internally but forgetting browser/public compatibility.

## 2-Minute Interview Answer

> “I use gRPC for strongly typed internal service contracts and streaming. Every RPC gets a realistic deadline that propagates downstream, and cancellation stops unnecessary child work but does not undo committed side effects. Retries are policy-driven and only safe for idempotent operations. I preserve protobuf field-number compatibility and account for long-lived HTTP/2 channels when designing load balancing and draining.”

## References

- [gRPC Core Concepts](https://grpc.io/docs/what-is-grpc/core-concepts/)
- [gRPC Deadlines](https://grpc.io/docs/guides/deadlines/)
- [gRPC Cancellation](https://grpc.io/docs/guides/cancellation/)
- [Protocol Buffers Programming Guides](https://protobuf.dev/programming-guides/)
