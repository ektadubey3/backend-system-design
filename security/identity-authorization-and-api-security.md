# Identity, Authorization, and API Security

## TL;DR

Authentication proves an identity under an assurance level; authorization decides whether that identity may perform this action on this resource now. OAuth delegates API access, while OpenID Connect adds an authentication identity layer. Validate tokens completely, keep browser sessions protected, and enforce authorization at the resource boundary—not only at the edge.

## Human sessions

For browser applications, an opaque, high-entropy session ID in a `Secure`, `HttpOnly`, appropriately `SameSite` cookie keeps credentials out of JavaScript. Regenerate sessions after authentication/privilege change, limit idle and absolute lifetime, revoke on risk, and protect state-changing requests against cross-site request forgery according to the cookie/request design.

Password storage uses a maintained adaptive password hash with per-password salt and tuned work factor. Prefer phishing-resistant MFA where risk warrants it. Account recovery is an authentication flow and often the weakest one; rate-limit, notify, audit, and avoid knowledge questions.

## OAuth and OpenID Connect

OAuth access tokens authorize a client to call a resource server; they are not automatically proof of user authentication. OpenID Connect defines ID tokens and authentication semantics. Use authorization code flow with PKCE for interactive public clients and follow the current OAuth security best current practice. Do not use the resource-owner-password grant.

At a resource server validate signature/algorithm, issuer, audience, expiration/not-before, token type, and required scope/claims. Pin allowed algorithms and trusted issuers; do not accept token-provided key URLs without a safe trust policy. Cache signing keys with rotation behavior that does not turn the identity provider into a per-request dependency.

Access tokens should be short-lived and restricted by audience and privilege. Refresh tokens require secure storage, rotation/replay response, revocation, and risk-based lifetime. Logout semantics must distinguish local session destruction from provider/global revocation.

## Authorization models

- **RBAC:** roles group permissions; manageable until roles multiply across contexts.
- **ABAC/policy:** evaluates subject, resource, action, and environment attributes; expressive but harder to reason about.
- **Relationship-based:** uses graph relationships such as owner, member, viewer; useful for sharing products.
- **Capability:** possession of an unforgeable scoped grant authorizes an action; signed URLs are an example.

Regardless of model, the resource service derives or loads ownership and checks the exact object/action. Prevent insecure direct-object reference by never treating an unguessable ID as authorization. Use policy versions and audit decisions for sensitive changes.

## Service identity and APIs

Give workloads distinct identities and narrow credentials. Mutual TLS can authenticate transport peers; signed workload tokens can carry audience and short expiry. Transport identity still needs application authorization.

API defenses include strict parsing, body/decompression limits, pagination limits, idempotency for mutations, replay protection for signed callbacks, rate/cost limits, safe outbound URL policy, and generic errors that do not expose secrets. Verify webhooks against the raw body, timestamp/nonce, known key, and replay window.

## Failure modes

- API accepts an ID token where an audience-scoped access token is required.
- Gateway authenticates, but downstream trusts a caller-controlled identity header.
- Role grants tenant-wide access where object ownership was intended.
- Long-lived bearer token appears in URLs or logs.
- Key rotation fails because verifiers cache one key forever or fetch on every request.
- Authorization service outage defaults to allow for sensitive actions.

## Interview prompts

- Is the token authenticating a user or authorizing an API client?
- Where is resource ownership loaded and enforced?
- How are tokens revoked/rotated without a per-request identity-provider call?
- What does the service do if policy state is unavailable?

## Two-minute answer

Use a central identity provider for authentication and issue short-lived audience- and privilege-restricted tokens; OIDC supplies identity semantics and OAuth supplies delegated API access. Validate issuer, audience, signature algorithm/key, time, token type, and permissions at each resource service. Browser sessions use protected cookies and CSRF controls. Workloads get distinct short-lived identities. Enforce object/action authorization from server-owned data, bound input and request cost, verify/replay-protect callbacks, and design key rotation, revocation, audit, and fail-closed behavior for sensitive operations.

## References

- [RFC 9700 — OAuth 2.0 Security Best Current Practice](https://www.rfc-editor.org/rfc/rfc9700.html)
- [OpenID Connect Core 1.0](https://openid.net/specs/openid-connect-core-1_0.html)

