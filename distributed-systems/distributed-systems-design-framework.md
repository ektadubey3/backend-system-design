# Distributed-Systems Design Framework

## TL;DR

For every distributed decision, walk through invariant, authority, failure model, protocol, client semantics, repair, and operations. This prevents vague claims such as “eventually consistent,” “quorum,” or “active/active” from hiding the hard behavior.

## The framework

### 1. Define invariants and consistency boundaries

List what must never happen and scope it to the smallest entity set. Separate authoritative state from projections, caches, indexes, and notifications.

### 2. Choose ownership and partitioning

Assign each boundary to a row/version, shard leader, consensus group, or home region. State the routing key, skew risks, ownership epoch, and how stale clients redirect.

### 3. Declare the failure model

Name process, node, zone, region, network partition, pause, corruption, and operator failures. State how many failures can be tolerated and which component stops making progress without quorum.

### 4. Select replication and consistency

Choose leader, multi-leader, or leaderless replication. Define acknowledgement durability, read semantics, staleness/session behavior, and repair. If using `N/R/W`, also explain version and membership assumptions.

### 5. Handle time and ambiguous outcomes

Use deadlines for waiting, monotonic clocks for duration, and versions/log positions for order. Give every retry the same operation identity. Query or reconcile when a response is lost after a possible commit.

### 6. Coordinate only where required

Use consensus for critical ordered control state. Prefer conditional writes and idempotent queues over broad locks. If leases protect external state, include monotonically increasing fencing tokens.

### 7. Design movement and recovery

Describe replica catch-up, resharding, regional promotion, failback, backup restore, and derived-state rebuild as versioned state machines. Define checkpoints, verification, rollback, and load limits.

### 8. Make operations observable

Track replication lag in time/bytes, quorum health, leader terms/elections, shard skew, repair backlog, stale-token rejections, routing epochs, and recovery objectives. Alert on user impact and exhausted safety margins.

## Failure review table

| Event | Safety response | Availability response | Recovery evidence |
|---|---|---|---|
| leader dies | elect only quorum-approved current leader | brief write pause | new term and committed index |
| network partition | one authority for hard invariant | minority rejects or degrades | membership/quorum restored |
| retry after timeout | same operation ID | retry within budget | queryable final operation state |
| stale lease holder resumes | resource rejects old fencing token | new owner proceeds | rejection metric/audit |
| shard moves | one routing epoch is authoritative | forward or bounded pause | checksum, lag zero, cutover record |
| region lost | promote under ownership epoch | declared degraded feature set | measured RPO/RTO and reconciliation |

## Two-minute answer template

“The hard invariant is [X], scoped by [key], and [authority] serializes it. Data is partitioned by [key] and replicated across [failure domains] using [model]; acknowledgement proves [durability]. Reads provide [guarantee], implemented by [leader/quorum/session position]. Timeouts are ambiguous, so retries keep [operation ID]. Coordination uses [conditional write/consensus], and stale owners are fenced by [epoch]. During [partition/failure], the system [rejects/degrades/continues] to preserve [safety/liveness]. Recovery uses [log/snapshot/repair], verified by [metrics/checks].”

## Follow-up questions

- Which assumption breaks if the network is delayed rather than disconnected?
- Can the old leader still reach the storage system?
- How are concurrent multi-region updates resolved?
- How long can repair lag grow before recovery requires a full snapshot?
- What happens when control-plane metadata is unavailable but data nodes are healthy?

## Further study

- [CAP theorem](../fundamentals/cap-theorem.md)
- [PACELC](../fundamentals/pacelc.md)
- [SQL transactions: distributed transactions and saga](../databases/sql/transactions.md)
- [Reliability engineering](../reliability/README.md)
