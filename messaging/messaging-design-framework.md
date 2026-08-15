# Messaging Design Framework

## TL;DR

In an interview, walk from business event to failure recovery. The sequence is: intent, topology, contract, ordering, durability, processing transaction, retry/backpressure, replay, and operations. Broker choice comes after these requirements.

## Step-by-step framework

### 1. Name the message and owner

Is it a command with one logical owner, a domain event with independent subscribers, a disposable signal, or a retained change log? State who owns the schema and the business outcome.

### 2. Define scale and time

Estimate average and peak messages per second, message size, burst duration, retention, subscriber count, processing cost, and acceptable oldest-message age. Use sustained drain capacity, not only ingestion throughput.

```text
backlog growth per second = arrival rate - completion rate
drain time = backlog / (completion rate - new arrival rate)
```

If completion never exceeds arrivals after the burst, the system never recovers.

### 3. Choose topology

Use competing consumers for work distribution, separate subscriptions for independent fan-out, and a retained stream for replayable history. Split traffic by priority or failure policy when one class must not starve another.

### 4. Define contract and evolution

Specify event ID, business key, type, version, occurrence time, payload, and trace context. Prefer additive schema evolution and state the compatibility window. Decide payload versus reference and minimize sensitive data.

### 5. Scope ordering and partitioning

Name the invariant—usually per aggregate—and select the partition or group key. Estimate hot-key throughput. Include an entity version if consumers must detect gaps or stale updates.

### 6. Close publication and processing gaps

Use a [transactional outbox or CDC](transactional-outbox-and-cdc.md) when database state and publication must agree. Process at least once, committing a deduplication/business key with the state transition before acknowledgement. Treat external outcomes as ambiguous until queried or reconciled.

### 7. Design failure policy

Classify transient, overload, permanent, poison, and ambiguous failures. Bound attempts and age, use backoff with jitter, assign one retry owner, and define the DLQ entry, alert, repair, and replay workflow.

### 8. Plan overload and replay

Monitor lag and oldest age. Bound in-flight work and queues, scale within partition/downstream limits, and propagate pressure to producers. Make projection rebuilds separate from irreversible side effects and rate-limit replay.

### 9. Choose and operate the broker

Compare the required model to [broker choices](broker-selection.md). Define replication/durability acknowledgement, regional failure behavior, quotas, capacity headroom, backup or reconstruction plan, and ownership.

## Example decision record

```text
Message: OrderPlaced domain event
Topology: retained topic; notification and fulfillment groups
Key/order: order_id; monotonically increasing order_version
Publish: order row + outbox row in one transaction; CDC relay
Process: inbox key + state transition in one transaction; ack after commit
Retry: 5 attempts over 20 minutes; jitter; permanent errors to owned DLQ
Retention: 14 days; projections snapshot daily
SLO: 99.9% published within 10 s; 99% consumed within 60 s
Overload: priority isolation, bounded consumer concurrency, admission control upstream
```

## Failure-injection questions

- The broker accepted a publish but the producer timed out. What identifies a duplicate?
- The consumer committed and crashed before acknowledgement. What prevents a repeated effect?
- One partition is ten times hotter. Which invariant blocks further sharding?
- The schema owner deploys while a consumer is offline for a week. Can it recover?
- A replay starts during peak traffic. What protects live work?

## Two-minute answer template

“I’ll model this as [command/event/log] because [reason]. Peak load is X and the backlog must remain younger than Y. Records are keyed by Z, which gives per-Z order and parallelism elsewhere. Publication uses [direct/outbox] durability; consumers process at least once with a transactional idempotency key and acknowledge after commit. Retries are classified, capped, jittered, and end in an owned DLQ. Contracts are additive and replay suppresses irreversible effects. I’ll monitor publish latency, oldest age, lag, retries, DLQ rate, duplicates, and broker headroom, then select the broker whose native model fits those constraints.”

## Further study

- [SQL transactions: idempotency, outbox, CDC, saga, and 2PC](../databases/sql/transactions.md)
- [Reliability design framework](../reliability/reliability-design-framework.md)
- [Observability design framework](../observability/observability-design-framework.md)

