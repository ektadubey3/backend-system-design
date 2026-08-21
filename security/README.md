# Security Architecture

Security design preserves confidentiality, integrity, availability, authenticity, and accountability across trust boundaries. It is a system property: identity, authorization, data flow, operations, recovery, and abuse resistance must be designed together.

## Learning path

1. [Threat modeling and trust boundaries](threat-modeling-and-trust-boundaries.md) — assets, actors, entry points, misuse, and layered controls.
2. [Identity, authorization, and API security](identity-authorization-and-api-security.md) — sessions, OAuth/OIDC, service identity, and object-level checks.
3. [Data, secrets, isolation, and abuse](data-secrets-isolation-and-abuse.md) — encryption, key/secrets lifecycle, tenant boundaries, supply chain, and availability attacks.
4. [Security design framework](security-design-framework.md) — interview sequence and review checklist.

## Scope

This curriculum focuses on architecture decisions, not a substitute for organization-specific secure coding standards, legal review, penetration testing, or incident response. Current protocol requirements should be checked against their primary specifications and maintained libraries.

## Questions every design should answer

- What assets and actions matter, and who may perform them?
- Where does untrusted input cross a trust boundary?
- Which component authenticates, and which resource authorizes?
- What happens after credential, token, key, or tenant-isolation failure?
- Which data is collected, retained, encrypted, logged, and deleted?
- How are abuse, audit, rotation, revocation, and incident containment operated?

