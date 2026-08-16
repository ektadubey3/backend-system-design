# Replication and Quorums

## TL;DR

Replication creates copies for availability, locality, and read scale; it also creates lag, failover, repair, and conflict problems. Quorums overlap read and write replica sets only under stated assumptions. They do not by themselves guarantee linearizability, fresh reads, or correct failover.

## Replication models

### Single leader

One leader orders writes and followers copy its log. It is easy to reason about, but leader failure requires detection and safe election. Asynchronous followers can serve stale reads or lose acknowledged writes during failover; synchronous acknowledgement reduces that window at latency and availability cost.

### Multi-leader

Several leaders accept writes, often per region. It improves local write availability but concurrent updates require conflict detection/resolution. Unique constraints and auto-increment keys become distributed concerns.

### Leaderless

Clients or coordinators write to several replicas and read from several. Versions and read repair reconcile disagreement. Sloppy quorums and hinted handoff improve availability but weaken simple overlap reasoning because the write may land on substitute nodes.

## Quorum arithmetic

For `N` replicas, write acknowledgement from `W`, and reads from `R`, `R + W > N` creates an overlap between the selected sets. `W > N/2` makes write sets overlap. But correctness also depends on:

- versions that identify the newest value correctly;
- membership agreement about which nodes form `N`;
- no stale coordinator accepting writes under an old configuration;
- repair of missed replicas;
- the precise consistency model and read algorithm.

Quorum is not synonymous with consensus. Consensus orders a sequence of decisions despite failures; a quorum is a set-size/overlap technique used inside many protocols.

## Lag and read semantics

Name the read guarantee:

- eventual: a replica converges if writes stop and repair succeeds;
- read-your-writes: a session sees its completed writes;
- monotonic reads: a session does not move backward;
- bounded staleness: age or version lag stays under a declared limit;
- linearizable: operations appear in one real-time-respecting order.

Sticky sessions, version tokens, leader reads, quorum reads, or waiting for a replica position can implement different guarantees. “Read replica” alone specifies none.

## Repair and anti-entropy

Replicas missed during failure must catch up by log replay, snapshot, read repair, or background anti-entropy. Track byte and time lag, not only “replica connected.” A long outage can exceed retained logs and force a full rebuild. Merkle-tree-style comparisons can narrow differing ranges, but repair still consumes network and storage I/O.

## Failover correctness

Before promotion, ask whether the candidate contains every acknowledged write required by policy. Prevent the old leader from continuing writes using consensus-backed terms, leases with fencing, or storage-level fencing. DNS or health-check failover alone does not prove authority.

## Tradeoffs

| Choice | Gain | Cost |
|---|---|---|
| Synchronous replicas | smaller loss window | write latency and reduced availability |
| Asynchronous replicas | fast/local writes | lag and possible failover loss |
| Leader reads | freshness and simple order | leader load/latency |
| Follower reads | scale and locality | explicit staleness/session policy |
| More replicas | failure tolerance/read locality | write/repair/storage cost |

## Interview prompts

- What does the write acknowledgement prove?
- Can a promoted replica lose an acknowledged write?
- What read guarantee does the user need after a write?
- How is a replica repaired after it falls off the retained log?

## Two-minute answer

Pick replication from the business loss and latency objectives. A single leader gives simple ordering; followers may acknowledge synchronously for critical durability or asynchronously for latency. State the exact read guarantee and how sessions obtain it. If using quorums, give `N/R/W` plus version, membership, and repair assumptions rather than relying on arithmetic alone. Promotion must prove an up-to-date candidate and fence the old leader. Monitor lag, retained-log headroom, repair, and failover data loss.

## References

- [Amazon Dynamo paper](https://www.amazon.science/publications/dynamo-amazons-highly-available-key-value-store)
- [Raft paper](https://raft.github.io/raft.pdf)

