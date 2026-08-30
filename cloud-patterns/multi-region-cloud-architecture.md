# Multi-Region Cloud Architecture

## TL;DR

Multi-region cloud architecture is valuable when latency, sovereignty, or regional recovery objectives justify duplicated capacity and operational complexity. Make a region a largely independent data-plane unit, define data ownership and promotion epochs, remove hidden global dependencies, and drill failover plus failback. DNS alone is not a disaster-recovery protocol.

## Regional stack

A regional cell commonly includes edge/gateway, compute, cache, queue, data replicas/authority, telemetry buffer, secrets/config cache, and capacity to operate through another zone loss. Global services should be few: traffic/placement control, identity authority, artifact distribution, and selected data coordination.

Existing assignments and safe cached identity/config should permit bounded serving during global control-plane loss. Changes such as new tenant placement may pause.

## Traffic routing

Options include latency/geo DNS, anycast/global load balancing, or application routing by tenant home. Health must reflect the ability to serve the critical journey and preserve data safety, not a shallow endpoint. Gradual weighted evacuation is safer than instant global shift when the destination may lack warm cache or quota.

Clients and recursive DNS cache records, connections persist, and some traffic continues to the old region. Application ownership epochs and redirects/forwarding handle convergence; TTL is not fencing.

## Data topology

Use the [Multi-region data ownership](../distributed-systems/multi-region-data-ownership.md) chapter as the canonical consistency model. Common cloud implementations are:

- one write region with cross-region replicas/backups;
- home-region shards/tenants with asynchronous remote copies;
- multi-writer only for explicitly mergeable data or globally coordinated invariants.

Queues, object stores, search indexes, caches, and scheduled jobs need their own regional semantics. Avoid synchronous cross-region calls in the steady-state critical path unless the invariant truly requires their latency/availability tradeoff.

## Evacuation and failback

Pre-provision quotas, artifacts, keys, endpoints, networking, third-party allowlists, and minimal data/capacity. During promotion, advance durable ownership, fence old writers, shift a cohort, observe, then increase. Measure data loss and backlog.

Failback chooses an authority, reconciles operations, rebuilds replication, and migrates ownership under a new epoch. Do not reverse routing while divergent writes exist.

## Residency and governance

Residency applies to replicas, backups, logs, support access, analytics, and disaster targets. Route tenant data using durable policy metadata and audit movements. Encryption keys may be region-scoped, requiring tested recovery that does not violate residency.

## Cost and operations

Active/passive retains unused capacity; active/active can use it but makes failover load assumptions tricky. Model inter-region replication and egress, duplicate managed services, observability, idle headroom, drills, and on-call expertise. A cheaper topology that cannot meet measured RPO/RTO is not equivalent.

## Failure modes

- Global identity, secret, artifact, or telemetry dependency blocks every region.
- Health-based routing sends traffic to a stale or non-authoritative database.
- Destination quota/autoscaling/control plane fails during evacuation.
- Old clients continue writes after DNS shift because no fencing epoch exists.
- Failover is tested, but failback and queued/scheduled work are not.
- Cross-region logs/backups violate residency policy.

## Interview prompts

- Which global dependency is on the per-request path?
- How is the old region prevented from accepting writes?
- Can the destination serve failover load before autoscaling reacts?
- What data and asynchronous work are included in RPO/RTO?

## Two-minute answer

Build mostly independent regional data-plane cells and minimize global per-request dependencies. Route by latency or tenant home, but make data authority explicit with ownership epochs and old-writer fencing. Replicate according to RPO/latency, keep non-authoritative reads session-aware, and include queues, objects, jobs, identity, secrets, and third parties. Pre-provision quota/headroom, evacuate gradually, measure real RPO/RTO, reconcile before failback, and enforce residency across telemetry/backups—not only primary rows.

