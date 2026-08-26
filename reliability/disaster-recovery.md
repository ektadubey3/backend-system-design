# Disaster Recovery: RPO, RTO, and Restore Evidence

## TL;DR

Disaster recovery is a tested path from declared loss to declared service. RPO bounds acceptable data loss; RTO bounds recovery time for a named capability. Replicas support availability, backups support recovery from corruption and deletion, and neither matters until restore/failover drills produce evidence.

## Define the objective precisely

- **RPO (recovery point objective):** maximum acceptable loss of committed data, expressed as time or operations.
- **RTO (recovery time objective):** maximum acceptable duration to restore a specified product capability.

Avoid one objective for the whole company. Authentication, payment acceptance, historical analytics, and recommendation training have different business impacts and dependency graphs.

## Threats require different controls

| Threat | Primary control |
|---|---|
| instance/node loss | replicas and automated replacement |
| zone failure | independent zonal replicas and routing |
| region loss | regional copy plus ownership/failover plan |
| operator delete or bad migration | point-in-time backup and protected recovery copy |
| application corruption | delayed/immutable backup, validation, replay |
| credential compromise/ransomware | isolated account, immutability, separate authorization |
| derived index loss | rebuild from authoritative source and checkpoint |

A synchronously replicated bad write is present on every replica. Backup isolation and retention are separate decisions.

## Backup design

Define full/incremental or snapshot/log strategy, frequency, retention, encryption, immutability, location, access control, and deletion policy. Keep the database log needed for point-in-time recovery. Back up schema, configuration, keys, routing metadata, and object manifests—not only rows.

Application-consistent recovery may require coordinated snapshots or a known log position across services. Otherwise a restored order database may refer to payments or objects from a different point in time. Prefer rebuilding derived data from authoritative events/state.

## Recovery runbook

1. Declare incident scope and target recovery point.
2. Fence unsafe writers and preserve evidence.
3. Provision a clean recovery environment.
4. Restore base snapshot and replay logs to the target point.
5. Validate integrity, counts, invariants, and sampled business journeys.
6. Reconnect messaging, jobs, caches, and indexes under controlled rate.
7. Shift traffic and observe SLOs.
8. Reconcile ambiguous operations and document actual RPO/RTO.

The runbook needs named owners, credentials available during identity/control-plane failure, and safe abort points.

## Drills and metrics

Run scheduled restore drills into isolated environments. Measure backup age, log gap, restore duration, data validation failures, dependency readiness, backlog drain, and operator decision time. A successful backup job is not restore evidence.

Chaos testing of instance loss complements but does not replace full data recovery. Regional failover and failback should include queues, secrets, DNS, third-party endpoints, scheduled jobs, and observability.

## Failure modes

- Backups share credentials/account with production and are deleted by the same compromise.
- Encryption key is unavailable in the recovery environment.
- Restore is fast, but rebuilding indexes exceeds RTO.
- Backup captures files without a transaction-consistent point.
- Failover meets API RTO but loses scheduled and asynchronous work.
- No one has permission to execute the runbook during an identity outage.

## Interview prompts

- Which data loss does the RPO permit, and who approved it?
- Can a corrupt write be recovered after replication spreads it?
- What was the measured duration of the last restore drill?
- Which hidden control-plane dependency blocks recovery?

## Two-minute answer

Set per-capability RPO and RTO from business impact. Use replicas for component availability and independent, immutable, encrypted backups plus logs for point-in-time recovery from corruption or deletion. Include schema, keys, configuration, queues, and derived-state rebuild. The runbook fences writers, restores to a chosen point, validates business invariants, controls backlog/traffic, and reconciles ambiguous work. Regular isolated drills measure actual RPO/RTO and test both failover and failback.

