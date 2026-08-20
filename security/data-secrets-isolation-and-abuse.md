# Data, Secrets, Isolation, and Abuse Resistance

## TL;DR

Collect less data, encrypt it across named boundaries, control keys separately, rotate short-lived secrets, and enforce tenant identity in every storage/cache/message path. Availability and cost are security properties: bound expensive work and isolate noisy or compromised tenants.

## Data lifecycle

Classify data at collection: purpose, owner, sensitivity, allowed consumers, geography, retention, and deletion method. Propagate classification to events, caches, indexes, logs, traces, backups, and test environments. A deletion feature that only removes the primary row is incomplete.

Encryption in transit protects a connection; encryption at rest protects stored media under a key model. Neither prevents an authorized but compromised application from reading plaintext. Reduce access and isolate keys. Envelope encryption commonly uses a data-encryption key per object/tenant/domain, wrapped by a centrally controlled key-encryption key.

Define rotation, disable/revoke, backup, regional availability, and disaster access for keys. Losing the only encryption key is data loss; granting every service decrypt permission defeats separation.

## Secrets lifecycle

Prefer workload identity and short-lived credentials to copied static secrets. When secrets remain:

- generate high entropy in a managed secret store;
- scope by workload/environment and least privilege;
- deliver at runtime without source control or image layers;
- never log; detect accidental exposure;
- rotate automatically with overlap where protocol requires it;
- revoke and audit access;
- test dependency behavior during rotation/outage.

Kubernetes Secret objects are an API mechanism, not a complete secret-management policy; storage encryption, RBAC, namespace and node access, and external rotation still matter.

## Tenant isolation

Choose pooled tables with mandatory `tenant_id`, separate schemas/databases, or dedicated cells based on sensitivity, scale, and blast radius. Enforce tenant identity from authenticated context and include it in:

- database keys/row policies and unique constraints;
- cache keys and invalidation;
- object prefixes and signed URLs;
- topics/partitioning and consumer authorization;
- search filters and analytics datasets;
- rate limits, quotas, and encryption/audit domains.

Test cross-tenant negative cases automatically. Admin/support impersonation needs explicit time-bound authority and audit.

## Abuse and resource economics

Attackers choose cheap requests that cause expensive fan-out, parsing, decompression, search, image/video processing, or third-party calls. Authenticate before expensive work when possible, cap body/expansion/fan-out, apply per-identity and global budgets, queue sandboxed work, and protect shared downstreams with concurrency limits.

Rate limiting is one layer. Add anomaly/risk signals, reputation, proof/challenge where appropriate, content validation, quotas, and manual response. Fail policy depends on the asset: authorization often fails closed; a public content read may serve a safe cache.

## Supply chain and operations

Pin and verify dependencies/artifacts, generate provenance and software inventories, scan continuously, sign releases, separate build from deploy authority, and use staged rollout. Patch prioritization considers reachability and impact, not only severity score. Recovery keys and CI credentials deserve the same threat model as production.

## Failure modes

- Cache key omits tenant and leaks another customer's object.
- Secret rotation breaks all instances simultaneously.
- Personal data is redacted from logs but remains in traces and DLQs.
- Per-user limit exists, but attackers create unlimited users and exhaust a shared provider quota.
- Deletion removes current data but not backups/derived indexes under declared policy.
- One tenant's unbounded export consumes every worker and database connection.

## Interview prompts

- Which component can decrypt, and how is that permission revoked?
- Does tenant identity participate in every derived key and query?
- What is the most asymmetric attacker-to-server cost path?
- How does deletion interact with immutable backup retention?

## Two-minute answer

Minimize and classify data at collection, then carry retention/deletion policy into every derived store and telemetry system. Encrypt named transport/storage boundaries with a separately controlled, rotatable key hierarchy. Prefer short-lived workload identities; scope and audit remaining secrets. Derive tenant identity from authentication and enforce it in database, cache, object, message, search, and quota keys. Bound request cost and isolate tenants/pools so abuse cannot turn cheap inputs into unbounded work. Include build artifacts, CI, backups, and recovery credentials in the same control lifecycle.

## References

- [Kubernetes — Security concepts](https://kubernetes.io/docs/concepts/security/)

