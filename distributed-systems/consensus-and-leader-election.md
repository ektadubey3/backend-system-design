# Consensus and Leader Election

## TL;DR

Consensus lets a group agree on one ordered history despite crashes and message delay, provided a quorum can communicate. Leader election is part of that protocol, not merely choosing the first node that responds. Use consensus for small, critical control state and serialization points; avoid putting every data operation through one global group.

## Replicated state machine model

Clients propose commands. A leader appends each command to a replicated log. Once the entry is accepted under the protocol's quorum rule, replicas apply committed entries in the same order to deterministic state machines.

```text
client -> current leader -> replicated log quorum -> commit index -> apply
```

Terms or ballots distinguish leadership generations. Log positions order commands. A node with an old term cannot legitimately overwrite a committed decision.

## What a majority gives

Two majorities of the same stable membership intersect, so competing decisions cannot both gain a majority without sharing at least one participant. The protocol adds rules about log freshness, voting once per term, commit advancement, and recovery to turn that overlap into safety.

With three voters, the group tolerates one unavailable voter; with five, two. More voters add failure tolerance but also replication latency and operations cost. Place voters across independent failure domains, while considering correlated network and control-plane failures.

## Election is not authority by itself

A naive lease table, health check, or “smallest node ID wins” may create two leaders during partitions or pauses. A consensus-backed election establishes a monotonically increasing term. Resources outside that consensus group still need fencing tokens if an old leader can write to them.

Leadership does not imply every follower is current, and a newly elected leader may need to establish authority before serving linearizable reads. Protocol-specific read-index, lease, or quorum mechanisms matter.

## Membership changes

Changing voters is itself a consensus decision. Switching directly from old to new membership can create disjoint majorities. Protocols use joint consensus or another staged rule so old and new configurations overlap during transition. Treat disaster replacement and region evacuation as tested procedures, not a configuration edit improvised during an incident.

## Where to use it

Good uses include shard leadership, configuration versions, metadata ownership, job leases, schema epochs, and lock services. A consensus service is not automatically a high-throughput business database. Keep values small, watch transaction/log size, and partition independent authorities into separate groups.

## Failure modes

- Three voters placed on two physical hosts, so one host loss removes quorum.
- A client retries a committed command with a new operation ID and applies it twice.
- The old leader can still reach storage after a new term elects another leader.
- Large values or slow disk make heartbeats/elections unstable.
- Automated membership removal reacts to a transient partition and destroys quorum.
- A read from an isolated former leader is presented as current.

## Interview prompts

- What state requires one agreed order?
- How many voters and which failure domains?
- What happens without quorum: reject, read stale, or degrade?
- How are external resources fenced by leadership term?

## Two-minute answer

Use a replicated state machine: the current-term leader proposes log entries, a quorum replicates them, and deterministic replicas apply committed entries in order. Majority intersection protects safety within an agreed membership; staged membership changes preserve that overlap. Without quorum, the system sacrifices progress for safety. Scope consensus to critical metadata or per-shard authorities, carry idempotent client operation IDs, fence external writes by term, and test election, catch-up, and quorum-loss behavior.

## References

- [Raft paper](https://raft.github.io/raft.pdf)
- [The Chubby lock service](https://research.google.com/archive/chubby.html)

