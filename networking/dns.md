# DNS

DNS is a distributed naming system that maps names to records used by clients and infrastructure.

For system design, DNS matters because it is **cached, hierarchical, failure-prone, and not an instantaneous global control plane**.

## Interview TL;DR

1. Separate recursive resolver behavior from authoritative DNS.
2. DNS responses are cached according to TTLs and resolver policy.
3. “DNS propagation” is usually cache expiry plus resolver behavior, not one synchronized global event.
4. Lower TTL can speed future record changes but increases authoritative query load and does not invalidate already-cached answers.
5. DNS-based failover is not instant because clients/resolvers may continue using cached results and existing connections.
6. Use health-aware global traffic management carefully; application/data failover must also be safe.
7. Negative answers can be cached.
8. DNS may use UDP or TCP depending on message/operation behavior.
9. DNS names are useful for decoupling clients from changing addresses, but they do not replace service health/load balancing.
10. DNS outages can make healthy backends unreachable.

## Resolution Path

Typical recursive lookup:

```text
application
   ↓
stub / OS resolver
   ↓
recursive resolver
   ↓
root
   ↓
TLD
   ↓
authoritative server
```

Caches can short-circuit this path.

## Important Record Types

- `A`: IPv4 address;
- `AAAA`: IPv6 address;
- `CNAME`: alias to another name;
- `MX`: mail exchange;
- `TXT`: arbitrary text/policy/verification;
- `NS`: authoritative name server;
- `SRV`: service location for protocols that use it;
- `CAA`: certificate-authority authorization policy.

Understand semantics before using a record as architecture.

## TTL and Caching

```text
record TTL = 300s
```

A resolver may reuse the answer while fresh.

Changing the authoritative record does not force every client to re-query immediately.

## Negative Caching

NXDOMAIN/negative responses can also be cached.

This matters when creating a previously nonexistent hostname shortly after failed lookups.

## DNS Failover

Example:

```text
api.example.com
   ↓
Region A

Region A fails
   ↓
authoritative DNS changed to Region B
```

Clients may still:

- use cached Region A address;
- hold existing connections;
- use resolver-specific cache behavior.

DNS is useful for coarse traffic steering, not a sub-second failover guarantee.

## Global Traffic Steering

DNS can contribute to:

- latency-based routing;
- geo routing;
- weighted migration;
- region failover.

But the application must also define:

- data ownership;
- session/state behavior;
- regional readiness;
- consistency.

## DNS vs Load Balancer

DNS selects/returns addresses at name-resolution time.

A load balancer makes routing decisions for connections/requests in the traffic path.

They are complementary.

## Internal Service Discovery

Internal systems may use DNS names for services, but discovery can also include:

- service registry;
- Kubernetes Service DNS;
- client-side discovery;
- service mesh.

Do not assume internet DNS semantics exactly match every internal discovery system.

## Security

Consider:

- registrar/account protection;
- DNSSEC where required;
- split-horizon/internal records;
- rebinding/SSRF implications for applications that resolve user-controlled names;
- avoiding hardcoded IPs.

## Common Mistakes

- “change DNS and traffic immediately moves”;
- setting low TTL only at incident time and expecting existing caches to forget old data;
- confusing DNS with application health checks;
- assuming DNS always uses UDP;
- ignoring negative caching;
- using DNS failover without safe data failover.

## 2-Minute Interview Answer

> “DNS gives clients a stable name over changing infrastructure, but it is cached and therefore not an instantaneous traffic-control plane. I define TTLs ahead of time, account for stale resolver caches and existing connections, and use DNS for coarse regional steering while a load balancer handles request/connection routing. Regional DNS failover is only safe if the data and write-ownership model is also safe.”

## References

- [RFC 1034 — DNS Concepts and Facilities](https://www.rfc-editor.org/rfc/rfc1034.html)
- [RFC 1035 — DNS Implementation and Specification](https://www.rfc-editor.org/rfc/rfc1035.html)
