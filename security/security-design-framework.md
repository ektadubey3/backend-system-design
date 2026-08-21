# Security Design Framework

## TL;DR

In a system-design interview, integrate security after requirements and revisit it for every component. The compact sequence is: assets, actors, boundaries, identity, authorization, data, abuse, operations, and recovery. Tie controls to concrete misuse cases instead of listing acronyms.

## Framework

### 1. Classify assets and obligations

Name sensitive data, money/inventory actions, availability, administrative authority, audit evidence, residency, retention, and deletion. Rank impact.

### 2. Draw actors and trust boundaries

Include users, devices, workloads, administrators, CI/CD, third parties, data/control planes, backups, and support tooling. Mark untrusted inputs and outbound calls.

### 3. Authenticate at the correct assurance

Choose session, token, MFA, workload identity, and third-party verification by risk. Define lifecycle: issue, validate, rotate, expire, revoke, recover.

### 4. Authorize action and object

Use deny-by-default policy based on server-owned subject/resource relationships. Scope privileges and credentials. Audit sensitive decisions and break-glass access.

### 5. Protect the data lifecycle

Minimize, encrypt, tokenize, redact, classify, retain, delete, and restore. State who can decrypt and where sensitive data may not flow.

### 6. Bound input and abuse cost

Validate schema/content, limit size/decompression/fan-out, rate and concurrency limit, isolate tenants, sandbox risky processing, and select fail-open/closed per feature.

### 7. Secure change and operations

Protect dependencies, artifacts, configuration, deployment authority, administrative paths, and audit logs. Stage rollout and rehearse credential/key rotation.

### 8. Detect, contain, and recover

Monitor authentication/authorization anomalies, privilege changes, data access, key use, cross-tenant denials, abuse, and control changes. Predefine revocation, isolation, forensics, notification, and restore.

## Threat-to-control table

| Threat | Prevent | Detect | Recover |
|---|---|---|---|
| stolen token | short life, audience/scope, sender constraint where warranted | anomalous use/replay | revoke session/token family; rotate |
| object authorization bypass | centralized policy and resource check | denied/access audit and tests | disable path, review exposure |
| tenant leak | tenant-derived keys and row/object policy | canary/negative tests, anomaly audit | isolate cell, revoke, investigate |
| secret compromise | short-lived scoped identity | secret/key usage anomaly | rotate/revoke and redeploy |
| resource abuse | cost limits, quotas, sandbox, bulkhead | saturation and identity patterns | shed, block, restore headroom |
| corrupt release | signed artifact, approval, staged rollout | integrity and behavior monitoring | rollback, revoke artifact/credentials |

## Two-minute answer template

“The highest-value assets are [X] and the key misuse cases are [Y]. Trust boundaries are [A/B]. Users/workloads authenticate through [mechanism] with rotation and revocation; each resource service enforces [object/action policy] from server-owned data. Sensitive fields are minimized, encrypted under [key boundary], excluded from telemetry, and deleted through [lifecycle]. Inputs and expensive work are bounded, tenants have [isolation], and sensitive failures fail [closed/degraded]. CI/admin paths use scoped audited authority. We monitor [security outcomes] and can contain [credential/tenant/component] independently.”

## Further study

- [Distributed rate limiter case study](../case-studies/02-rate-limiter.md)
- [Payment system case study](../case-studies/07-payment-system.md)
- [Reliability design framework](../reliability/reliability-design-framework.md)

