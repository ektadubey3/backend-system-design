# Content Delivery Network (CDN)

A CDN moves content delivery and selected request processing closer to users through distributed edge locations.

For system design, the hard parts are **cache keys, freshness, invalidation, origin protection, dynamic caching, security, and failure behavior**.

## Interview TL;DR

1. A CDN reduces origin traffic and network distance when requests are cacheable or edge-processable.
2. Cache hit ratio alone is not enough; monitor **origin offload, byte hit ratio, p95/p99, and miss amplification**.
3. Cache keys must include every request dimension that changes the representation.
4. Long TTL improves hit ratio but increases stale-data risk.
5. Purge/invalidation is not instantaneous everywhere; design versioned URLs or bounded staleness when possible.
6. Use validators and `stale-while-revalidate`/stale serving where product semantics allow.
7. Protect the origin from cache-miss storms with request collapsing/origin shields/limits.
8. Never cache authorization-sensitive responses in a shared cache without an explicit safe key/policy.
9. A CDN improves edge availability, but origin/data dependencies can still fail.
10. CDN and load balancer solve different layers of the request path.

## Basic Path

```text
user
 ↓
CDN edge
 ├─ cache hit → response
 └─ cache miss
       ↓
   origin / shield
       ↓
     response
```

## Cache Key

Potential dimensions:

```text
scheme
host
path
query parameters
selected headers
content encoding
device/language only if representation truly differs
```

A wrong cache key can cause:

- poor hit rate;
- cache fragmentation;
- serving one user's/private content to another.

## Freshness

Example:

```http
Cache-Control: public, max-age=300
```

Trade-off:

```text
long TTL
→ high hit rate / low origin load
→ slower freshness

short TTL
→ fresher
→ more revalidation/origin traffic
```

## Versioned Assets

For immutable static assets:

```text
/app.4f3a91.js
```

can safely use a very long TTL because a new version gets a new URL.

This is often safer than frequent purge.

## Revalidation

Validators:

```http
ETag:
Last-Modified:
```

allow the edge to check whether stale content changed without re-downloading the entire object.

## Stale Serving

For suitable content, stale responses can preserve availability during origin trouble.

Policies may include:

- stale while revalidating;
- stale on origin error.

Never apply this automatically to correctness-critical state such as authorization or final inventory decisions.

## Cache Miss / Stampede

A hot object expires:

```text
many edge/client misses
       ↓
many origin requests
```

Mitigations:

- request collapsing;
- origin shield;
- jittered freshness;
- background refresh;
- stale serving;
- origin concurrency limits.

## Dynamic/API Caching

Can work when:

- representation is read-heavy;
- cache key is stable;
- privacy semantics are explicit;
- staleness is acceptable.

Examples:

- public catalog;
- anonymous configuration;
- popular article/feed fragments.

Do not assume “API = uncacheable.”

## Media and Large Objects

CDNs help with:

- range requests;
- edge delivery;
- egress reduction;
- signed URLs/cookies;
- origin shielding.

Media pipelines still need storage/transcoding/origin design.

## Security

Consider:

- DDoS/WAF capabilities;
- signed URLs;
- cache poisoning;
- Host/header normalization;
- origin access restriction;
- sensitive query parameters;
- TLS.

Keep the origin from being directly reachable when the CDN is meant to enforce security/policy.

## CDN vs Load Balancer

CDN:

```text
global edge delivery/caching
```

Load balancer:

```text
select healthy backend/region/instance for traffic that reaches service
```

Often:

```text
User → CDN → Load Balancer → Application
```

## Common Mistakes

- cache key ignores auth/user variation;
- “purge is instant” assumption;
- no origin protection on a hot miss;
- caching mutable inventory as final authority;
- measuring only hit count rather than bytes/origin load;
- treating CDN as replacement for regional failover/data strategy.

## 2-Minute Interview Answer

> “I use a CDN when network distance and repeatable reads justify edge delivery. I define the cache key and freshness contract first, prefer immutable versioned URLs for static assets, and protect the origin from miss stampedes using request collapsing/shielding and bounded stale serving where safe. Shared caches never serve authorization-sensitive content unless the cache policy/key proves isolation.”

## References

- [RFC 9111 — HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111.html)
