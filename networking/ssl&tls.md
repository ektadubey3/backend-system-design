# SSL/TLS

Modern secure transport uses **TLS**. SSL and TLS 1.0/1.1 are legacy terminology/protocols and should not be the basis of a modern design.

TLS provides a secure channel with confidentiality, integrity, and peer authentication according to the configured identity model.

## Interview TL;DR

1. TLS protects data **in transit between TLS peers**; identify where TLS terminates.
2. Server authentication is standard; client authentication is optional and can use mTLS.
3. Certificates bind identities to public keys through a trust chain.
4. Modern TLS uses asymmetric signatures/key agreement during handshake and symmetric AEAD keys for application traffic.
5. TLS 1.3 reduces handshake complexity and supports session resumption.
6. 0-RTT/early data can be replayed; do not use it blindly for non-idempotent operations.
7. Certificate rotation, trust-store rotation, expiry monitoring, and private-key protection are production requirements.
8. `HTTPS at the edge` does not automatically mean internal traffic is encrypted.
9. SNI and ALPN affect virtual hosting and application-protocol negotiation.
10. As of 2026, RFC 9846 is the current TLS 1.3 specification and obsoletes RFC 8446.

## Trust Boundary

```text
client
  ↓ TLS
CDN / load balancer
  ↓ ?
service
  ↓ ?
database
```

Ask which hops require:

- TLS;
- mTLS;
- network policy;
- service identity.

## Handshake Mental Model

Simplified:

```text
ClientHello
  ↓
ServerHello + negotiated parameters
  ↓
server identity proof / certificate
  ↓
key agreement
  ↓
Finished verification
  ↓
encrypted application traffic
```

Exact message flow depends on TLS version and resumption.

## Certificates

A certificate typically includes:

- public key;
- subject/subject alternative names;
- issuer;
- validity period;
- signature.

Client validation checks the identity against the intended server name and a trusted chain.

## TLS Termination

### Edge termination

```text
client --TLS--> edge --HTTP/TLS--> backend
```

Benefits:

- centralized certificate management;
- crypto offload;
- routing visibility.

Risk:

- plaintext internal hop if TLS is not re-established.

### End-to-end / re-encryption

Useful when internal network/trust requirements demand protected service-to-service transport.

## mTLS

Mutual TLS authenticates both peers.

Useful for:

- service-to-service identity;
- administrative/internal endpoints;
- some zero-trust designs.

mTLS does not replace application authorization. A service identity may be authenticated but still not permitted to access a resource.

## SNI and ALPN

### SNI

Allows the client to indicate the target hostname during TLS negotiation, enabling multiple certificate/virtual-host configurations on one address.

### ALPN

Negotiates the application protocol, such as HTTP/2.

These details matter at load balancers and gateways.

## Session Resumption

Resumption can reduce handshake work and latency.

Plan for:

- ticket/key rotation;
- multi-instance consistency;
- expiry;
- security policy.

## 0-RTT / Early Data

Early data improves latency but has replay considerations.

Treat non-idempotent application operations carefully. A transport optimization must not create duplicate business effects.

## Private-Key Operations

Protect private keys using appropriate secret/key management.

Avoid:

- source control;
- container image baking;
- logs;
- broad filesystem permissions.

## Certificate Rotation

Monitor:

- expiry;
- failed renewal;
- trust-chain changes;
- client trust-store rollout;
- certificate mismatch;
- handshake failure rate.

Rotation must work without a maintenance outage.

## Common Mistakes

- saying TLS “encrypts with the public key” for all application data;
- assuming TLS termination location does not matter;
- mTLS treated as authorization;
- forgetting certificate/trust-store rotation;
- using 0-RTT for arbitrary writes;
- relying on legacy SSL/TLS versions.

## 2-Minute Interview Answer

> “I define the TLS trust boundary first. The client authenticates the server certificate and the handshake establishes symmetric traffic keys; internal hops may need re-encryption or mTLS depending on service-identity requirements. I monitor and automate certificate/key rotation. For TLS 1.3 resumption I treat 0-RTT carefully because replayable early data can be dangerous for non-idempotent operations.”

## References

- [RFC 9846 — TLS 1.3](https://www.rfc-editor.org/rfc/rfc9846.html)
- [RFC 9525 — Service Identity in TLS](https://www.rfc-editor.org/rfc/rfc9525.html)
