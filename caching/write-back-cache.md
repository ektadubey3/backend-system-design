# Write-Back Cache

Write-back, also called write-behind, acknowledges a write at a cache or buffer before the authoritative store has necessarily persisted it. This can improve throughput and latency, but it creates an asynchronous durability system—not merely a faster cache.

## Interview TL;DR

1. Separate fast acknowledgment from durable acknowledgment.
2. A cache-accepted write may be lost before asynchronous persistence.
3. Define ordering, retries, duplicates, conflict handling, backpressure, replay, and recovery.
4. If the cache is authoritative until flush, operate it with the durability rigor of a database or durable log.
5. Coalescing writes improves throughput but can lose intermediate state that some consumers need.
6. Queue growth is a correctness and capacity signal, not only a latency metric.
7. Choose write-back only when the business explicitly accepts its durability and consistency model.

## Mental Model

```text
Write
  |
  v
Cache acknowledges
  |
  | asynchronous
  v
Persistence worker / queue
  |
  v
Authoritative store
```

Two acknowledgment points must not be confused:

```text
fast acknowledgment
= accepted into volatile or partially durable cache/buffer

durable acknowledgment
= persisted to the required failure boundary
```

The required failure boundary might be local memory, local disk, replicated log, quorum, or another system. State it. “The cache accepted it” does not establish durability.

## How It Works

A write-back system needs more than a cache entry:

```text
ingress
  |
  v
ordered/idempotent pending-write representation
  |
  v
asynchronous flusher
  |
  +--> retry / dead-letter / alert
  |
  v
authoritative store
```

Each pending write should have enough identity to answer:

- Which entity and source version does it represent?
- Is this an absolute value or a non-commutative delta?
- Can the persistence operation be retried safely?
- Has a newer write superseded it?
- What client acknowledgment was already issued?
- How will it be replayed after failure?

### Write coalescing

For repeated updates:

```text
v1 -> v2 -> v3 -> persist only v3
```

This can save writes when only final state matters. It is unsafe when intermediate transitions are auditable events, trigger side effects, or must preserve order.

### Batching

The flusher can group many writes to amortize network and storage overhead. Larger batches improve efficiency but increase pending data, recovery work, and time before durable visibility.

## Durability Models

### Volatile write-back

Acknowledgment occurs after memory-only acceptance. Node/process loss can lose acknowledged writes. This is acceptable only when the data is reconstructable or the product explicitly tolerates that loss.

### Logged write-back

Pending writes enter a local or replicated durable log before acknowledgment. This can reduce loss risk, but the log's fsync, replication, failover, truncation, and recovery semantics define the actual guarantee.

### Queue-backed write-behind

A durable broker holds write intents and workers update the store. The cache may serve the newest view while persistence catches up. This resembles asynchronous event processing and needs idempotency, ordering, lag SLOs, and reconciliation.

If durable intent already exists elsewhere, clarify which system is authoritative while the database lags.

## When To Use It

Write-back can fit when:

- write latency or throughput is the measured bottleneck;
- temporary source lag is acceptable;
- writes can be safely batched or coalesced;
- pending writes have a durable recovery path matching the required RPO;
- conflicts and ordering are defined per key;
- the team can operate backlog and replay safely.

Examples may include aggregated counters, non-critical telemetry, or buffered state whose durable intent exists in a log. It is usually inappropriate as an unexplained shortcut for payments, inventory ownership, or other strict invariants.

## When Not To Use It

Avoid write-back when:

- an acknowledged write must already be durable in the authoritative store;
- cross-key ordering or transactions are required but unsupported;
- the cache can evict dirty entries;
- replay and idempotency are undefined;
- unbounded backlog can accumulate during a source outage;
- readers outside the cache require immediate source visibility;
- the organization cannot reconcile lost or divergent state.

## Architecture

A robust design separates clean cached values from dirty pending state:

```text
Cached representation
        !=
Durable pending-write log
```

If dirty data is kept in the cache, eviction must never silently remove it. Isolate it from disposable entries and define replication, persistence, backup, restore, and failover behavior.

For multi-region systems, decide one write owner per key or define conflict resolution. Independent regional write-back caches can accept conflicting values during a partition.

## Scaling Characteristics

Write-back decouples ingress rate from persistence rate temporarily:

```text
backlog growth rate = accepted write rate - durable flush rate
```

If the average accepted rate exceeds sustainable persistence, the backlog grows without bound. Burst absorption is not capacity creation.

Track:

- pending item/byte count;
- oldest pending-write age;
- enqueue and flush rate;
- retry and duplicate rate;
- coalescing ratio;
- durable acknowledgment latency;
- recovery time for current backlog.

## Failure Modes

### Cache node fails before flush

Acknowledged writes are lost unless replicated or durably logged to the required boundary.

### Persistence is slow or unavailable

Backlog grows, memory/disk fills, and the system must backpressure, reject, shed, or degrade writes before resources are exhausted.

### Duplicate persistence

A timeout after source commit can lead to retry. Use idempotency keys, source versions, or conditional writes.

### Reordering

Version 12 may arrive after version 13 due to retries or multiple workers. Enforce per-key ordering or reject stale versions.

### Conflict

An external writer changes the authoritative store while the cache holds pending state. Define optimistic concurrency, merge rules, or single ownership.

### Poison write

One invalid item repeatedly fails and blocks a partition or batch. Isolate it, alert, and preserve enough context for repair.

### Recovery storm

After restart, replay competes with live writes and overloads the source. Rate-limit recovery and preserve priority rules.

## Trade-offs

- Earlier acknowledgment improves user latency but increases acknowledged-data-loss exposure.
- Batching improves source efficiency but widens visibility lag.
- Coalescing reduces writes but may discard meaningful transitions.
- Replicated durable logging reduces loss risk but adds latency and system complexity.
- Allowing a larger backlog absorbs bursts but increases recovery time and resource needs.

## Production Gotchas

- State the RPO and exact acknowledgment boundary in design docs and APIs.
- Never permit eviction of unpersisted dirty state.
- Encrypt sensitive pending data and restrict operational access.
- Use idempotent persistence and monotonic versions where possible.
- Bound queue size and age; apply backpressure before exhaustion.
- Test crash-after-ack, duplicate delivery, out-of-order replay, source outage, failover, and full recovery.
- Reconcile the source against pending/logged writes after ambiguous failures.

## Alternatives

- Write-through provides later acknowledgment and simpler visibility at the cost of write latency.
- Direct source writes plus cache invalidation keep durability ownership clear.
- A durable message log with materialized views may express asynchronous authority more honestly.
- Batch at the database client without serving uncommitted cached state.

## Interview Questions

### Why is write-back faster?

It moves source persistence out of the synchronous request path and may batch or coalesce writes. The latency gain comes from changing acknowledgment semantics.

### Can acknowledged writes be lost?

Yes, unless the acknowledgment waits for a durable failure boundary that survives the failures in scope. Define that boundary.

### What happens when the source is down?

Backlog grows until a configured limit. The system must apply backpressure, reject, or degrade; it cannot buffer indefinitely.

## 2-Minute Interview Answer

> “I would choose write-back only if source write latency or throughput is the real bottleneck and the business accepts asynchronous persistence. I would define whether acknowledgment means memory acceptance, a local durable log, or replicated durable intent. Pending writes carry idempotency and monotonic versions so retries and reordering cannot overwrite newer state. The backlog is bounded and monitored by bytes and oldest age; a source outage triggers backpressure before storage exhaustion. Dirty state cannot be evicted, recovery is rate-limited, and reconciliation handles ambiguous commits. If the requirement says an acknowledged write must already be durable in the authoritative store, I would use direct writes or write-through instead.”

## Senior-Level Follow-ups

- Which intermediate writes may be coalesced safely?
- How do you recover after a crash between source commit and local acknowledgment?
- What is the maximum backlog and recovery time?
- How are conflicting regional writers handled?
- Does replication survive the exact failure in the stated RPO?

## References

- [Internal: Fault Tolerance and Resilience](../fundamentals/fault-tolerance.md)
- [Internal: Data Consistency Boundaries](../databases/consistency-boundaries.md)
- [Internal: Read-Through and Write-Through](read-through-and-write-through.md)
- [Redis documentation — Replication](https://redis.io/docs/latest/operate/oss_and_stack/management/replication/)

