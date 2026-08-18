# Threat Modeling and Trust Boundaries

## TL;DR

Threat modeling turns a diagram into explicit claims about assets, attackers, trust boundaries, abuse cases, controls, and residual risk. Do it before technology selection and revisit it when data flow or authority changes. “Internal” is a location, not an identity or authorization policy.

## Start with assets and outcomes

List what an attacker could steal, change, deny, or abuse:

- credentials, sessions, signing keys, payment tokens, personal data;
- authorization policy and tenant ownership;
- money, inventory, quotas, or audit records;
- service capacity and recovery mechanisms;
- software artifacts, configuration, and deployment authority.

Then write misuse cases: “a user reads another tenant's invoice,” “a compromised worker mints admin sessions,” or “an attacker exhausts image processing with cheap requests.” These are more testable than “the system is secure.”

## Data-flow and trust boundaries

Draw processes, data stores, external actors, third parties, control planes, and flows. Mark every boundary where identity, ownership, or administrative control changes: browser/API, edge/service, service/service, tenant/shared store, production/control plane, cloud/third party, and build/runtime.

At each crossing ask:

- How are both endpoints authenticated?
- Is transport confidential and integrity-protected?
- Is authorization checked for the specific action and object?
- Is input size, shape, and rate bounded?
- Can the receiver safely replay or deduplicate it?
- What sensitive data enters logs, traces, queues, caches, or backups?

## STRIDE as a prompt

STRIDE is one useful checklist:

| Category | Example | Control direction |
|---|---|---|
| Spoofing | stolen session impersonates user | strong authentication, token binding/rotation where applicable |
| Tampering | request changes tenant ID | integrity protection and server-derived ownership |
| Repudiation | operator denies a money change | immutable, access-controlled audit event |
| Information disclosure | cache key omits tenant | isolation, minimization, encryption |
| Denial of service | expensive endpoint is cheap to invoke | cost-based admission, quotas, degradation |
| Elevation of privilege | normal user invokes admin action | deny-by-default resource authorization |

The model does not prioritize for you. Rank by impact, likelihood, exploitability, detection, and recovery cost.

## Layer controls and reduce blast radius

Prevent, detect, respond, and recover. Authentication may prevent many unauthorized requests; object-level authorization contains a valid but over-broad token; audit detects misuse; scoped credentials and cells reduce impact; key rotation and restore support recovery.

Least privilege applies to humans, workloads, database users, cloud roles, CI, and third parties. Separate production administration from normal request handling. Time-bound elevated access, require strong approval for irreversible or money-moving operations, and record it in tamper-resistant audit storage.

## Common failure modes

- Trusting a tenant/user ID supplied in the body without binding it to identity.
- One powerful service credential shared by every workload.
- Authorization only at the gateway while internal object access is unchecked.
- Secrets, tokens, or personal payloads copied into telemetry and DLQs.
- A security control depends on the same component it must contain.
- Threat model omits build pipeline, operators, backups, or recovery keys.

## Interview prompts

- Which trust boundary is most consequential?
- What can a valid low-privilege identity abuse?
- How is a compromised component contained and revoked?
- Which control prevents, detects, and recovers from the top threat?

## Two-minute answer

Identify the valuable assets and concrete misuse outcomes, draw data/control flows, and mark identity or ownership boundaries. For each boundary, require authenticated endpoints, object-level deny-by-default authorization, input/cost bounds, data minimization, and audit. Rank risks, layer preventive/detective/recovery controls, scope credentials and tenant blast radius, and include operators, CI, third parties, backups, and incident revocation. Revisit the model when authority or data flow changes.

