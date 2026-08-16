# Time and Logical Clocks

## TL;DR

Physical clocks measure approximate time; they do not reliably prove which distributed event caused another. Use monotonic time for local durations, synchronized wall time for human timestamps with uncertainty, and logical versions/clocks when causal or serialization order matters.

## Three different needs

| Need | Appropriate mechanism |
|---|---|
| Timeout or elapsed duration inside one process | monotonic clock |
| Human/audit timestamp | wall clock plus synchronization and uncertainty handling |
| Causal or authoritative order | sequence number, log position, term/index, Lamport or vector clock |

Wall clocks can jump due to correction, VM migration, misconfiguration, or leap handling. Even well-synchronized clocks have nonzero error. Comparing timestamps from two machines as a tie-breaker creates silent lost-update risk.

## Happens-before and Lamport clocks

Event A happens-before B if A precedes B in one process, or A sends a message received before B, including transitive chains. A Lamport clock increments locally and advances beyond a received timestamp. If A happens-before B, then `L(A) < L(B)`. The reverse is not guaranteed: a lower number does not prove causality by itself, and concurrent events can be assigned an arbitrary order.

A pair `(logical_counter, node_id)` can form a deterministic total order, but that order is not automatically the business truth. It is useful for logs and conflict tie-breaking only when arbitrary resolution is acceptable.

## Vector clocks and versions

A vector clock tracks a counter per participant and can distinguish “A preceded B” from “A and B are concurrent.” Its metadata grows with participants, so real systems prune, scope, or replace it with version vectors tied to replicas.

Often the simplest solution is an authority-issued entity version:

```text
update entity
set version = version + 1
where id = ? and version = expected
```

The conditional write detects a race; the new version orders changes for that entity. A partition log offset orders records inside its partition. A consensus term/index identifies decisions in a replicated state machine.

## Expiry and leases

TTL and lease decisions require care. A client clock should not decide that it owns a server-issued lease. The authority evaluates expiry, and resource writes carry a fencing token so a delayed previous holder cannot mutate state after reassignment. See [coordination, leases, and fencing](coordination-leases-and-fencing.md).

## Event-time processing

For analytics, preserve event time separately from ingestion and processing time. Late records require watermarks or an explicit lateness window. Recomputing a historical result based on current processing time can make replay nondeterministic.

## Failure modes

- Last-write-wins silently discards a valid concurrent edit.
- A lease holder pauses longer than its TTL, resumes, and corrupts state.
- An NTP correction makes elapsed time appear negative when wall time was used.
- A timestamp sorts events visually but contradicts the authority's commit order.
- A vector clock grows without a bounded membership model.

## Interview prompts

- Do you need human time, duration, causality, or a total order?
- Which authority issues an entity version?
- Are concurrent writes merged, rejected, or resolved by policy?
- How does a replay treat late event time?

## Two-minute answer

Use monotonic clocks for local deadlines and wall clocks only for approximate real-world time. For correctness, order at the narrowest authority: entity versions, partition offsets, or consensus log positions. Lamport clocks capture a causality-compatible order but cannot identify concurrency; vectors can, with metadata cost. Never rely on client time for lease ownership—evaluate expiry at the authority and fence stale holders.

## References

- [Lamport — Time, Clocks, and the Ordering of Events in a Distributed System](https://lamport.azurewebsites.net/pubs/time-clocks.pdf)

