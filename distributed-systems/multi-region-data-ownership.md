# Multi-Region Data Ownership

## TL;DR

Multi-region design begins with ownership and user-visible failure behavior. A single write region is simpler and consistent but adds remote latency and a failover event. Multi-writer designs improve local write availability only by adding conflict semantics and global-invariant coordination. Declare RPO, RTO, residency, routing, and failback before choosing replication.

## Deployment shapes

| Shape | Write path | Strength | Cost |
|---|---|---|---|
| Active/passive | one primary region; standby receives copies | simplest ownership and failover reasoning | idle cost, failover RTO, possible async RPO |
| Active/read-local | one write region; local read replicas/caches | faster reads with single authority | stale reads and remote writes |
| Home-region sharding | each tenant/entity has one write home | local writes for most users; scoped failover | routing metadata and cross-home workflows |
| Active/active multi-writer | several regions write same logical data | local availability for commutative/mergeable data | conflicts and global constraints are hard |

“Active/active” is incomplete unless it says which data can be written in more than one region and how concurrent writes resolve.

## Ownership model

Assign a home region per tenant, account, or shard in strongly consistent routing metadata. Requests carry an ownership epoch. A move is a state machine: copy, catch up, stop or fence old writes, advance epoch, route, verify, and retain rollback/forwarding state.

Global uniqueness can use region-prefixed IDs without coordination. Global inventory or balance invariants cannot be solved by unique IDs; use a single authority, preallocated regional escrow, or a consensus boundary whose latency is accepted.

## Failure and ambiguity

Regional health is not binary. A region may serve users but lose replication or control-plane connectivity. Define triggers based on user impact and data safety, not one probe. Automatic failover can turn a network partition into two writers, so promotion requires a durable ownership decision and old-region fencing.

After promotion, failback is another migration—not a DNS reversal. Reconcile divergent or queued operations, establish a new replication baseline, and move ownership under a new epoch.

## Consistency and sessions

For remote asynchronous replicas, provide a session token containing a log/version position. A receiving region can wait, route to the authority, or return an explicit stale response. Do not promise read-your-writes merely because “data replicates globally.”

Caches and search indexes are derived regional state. Rebuild them from authoritative records rather than treating them as the failover source.

## RPO, RTO, and residency

- **RPO:** maximum acceptable committed-data loss, measured in time or operations.
- **RTO:** maximum acceptable time to restore the declared service capability.

Measure both in drills. Include DNS/traffic convergence, credential/control-plane dependencies, data catch-up, consumer backlog, and operator decision time. Data-residency rules may constrain replica location, backups, logs, support access, and failover targets—not only the primary database.

## Failure modes

- Both regions accept writes after an automated split-brain promotion.
- Global dependency such as identity, secrets, or routing prevents regional independence.
- Async lag exceeds the promised RPO unnoticed.
- Failover succeeds for APIs but not queues, scheduled jobs, or third-party allowlists.
- Failback overwrites writes made in the recovery region.
- Cross-region synchronous calls multiply tail latency and failure coupling.

## Interview prompts

- What is the unit of write ownership and how is it fenced?
- Which operations remain available during inter-region partition?
- What exact data can be lost at promotion?
- How is failback rehearsed without creating two authorities?

## Two-minute answer

Choose a topology from write locality and data-loss objectives. Default to one authority per shard or tenant; asynchronously replicate to other regions and use session positions for explicit freshness. Promotion advances an ownership epoch through a durable control plane and fences the old region. Multi-writer is reserved for mergeable data or uses scoped escrow/coordination for hard invariants. State and drill RPO/RTO, include every dependency, respect residency for replicas and telemetry, and treat failback as a controlled migration.

