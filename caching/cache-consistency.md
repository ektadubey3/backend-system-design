# Cache Consistency

Cache consistency defines what readers may observe while an authoritative value changes and updates propagate to cached copies. “Eventually consistent cache” is incomplete until the acceptable stale window, session guarantees, convergence path, and failure behavior are specified.

## Interview TL;DR

1. Identify the authoritative owner for every invariant.
2. Define consistency per operation, not per product or cache cluster.
3. Quantify tolerated staleness where possible.
4. Read-your-writes and monotonic reads are distinct from eventual convergence.
5. A cache may be safe for display while unsafe for mutation or authorization decisions.
6. Versions, routing, invalidation, and source revalidation can provide targeted guarantees.
7. CAP does not explain every ordinary cache-staleness race; use it specifically for partition behavior.

## Mental Model

```text
Authoritative store: version 42
Cache:               version 40
```

The cache is stale by two versions, but whether that matters depends on the operation:

```text
render product description -> perhaps acceptable
reserve last inventory unit -> not authoritative enough
```

Ask:

```text
Who decides the correct value?
How old may a cached read be?
Which user/session guarantees are required?
How does the cache converge?
What does the system do when convergence fails?
```

## Consistency Semantics

### Eventual consistency

If writes stop and propagation succeeds, cached copies eventually converge to the authoritative value. This says nothing by itself about whether convergence takes milliseconds, hours, or requires manual repair.

Add a consistency SLO, for example:

```text
99.9% of profile updates visible through cache within 5 seconds
99.99% within 60 seconds
```

### Bounded staleness

A cached value is no more than a stated time or version lag behind:

```text
maximum age since validated <= 30 seconds
```

The effective bound includes upstream replica lag, event lag, and every cache level—not just one TTL.

### Read-your-writes

After a client completes a write, its subsequent read observes that write or a newer value.

Possible mechanisms:

- update/invalidate the cache before responding, with defined failure handling;
- return the written representation and cache it locally for the session;
- route reads to the authoritative store until a version watermark is visible;
- include a version token and require cache/source version at least that high;
- bypass cache for a bounded period.

### Monotonic reads

After seeing version 42, the client should not later see version 40 because another instance or L1 cache is older.

Use session stickiness, client version watermarks, source fallback, or version-aware routing when this user experience matters.

### Strong/authoritative decision

The operation validates against the component that owns the invariant. Cached state may assist, but it does not make the final decision.

Examples:

- inventory display can be cached; reservation checks authoritative inventory;
- balance display may use a projection; ledger mutation uses the ledger;
- profile photo can be stale; permission revocation may require source or version validation.

## Divergence Causes

Cache and source can diverge because:

- TTL has not elapsed after a source update;
- invalidation failed or was delayed;
- an old reader repopulated stale data;
- events arrived out of order;
- a source read used a lagging replica;
- L1 and L2 have different versions;
- cache failover exposed older replicated data;
- a deployment changed representation or key semantics.

Name which failure is in scope before selecting a mitigation.

## Architecture Patterns

### Versioned cached values

```json
{
  "entityId": "p42",
  "sourceVersion": 18,
  "cachedAt": "...",
  "payload": {}
}
```

Versions let readers reject regressions and consumers ignore old updates. They do not guarantee the latest value unless the reader knows the latest required version.

### Critical read revalidation

```text
cache -> fast candidate/display
source -> final mutation or authorization decision
```

This separates user experience from invariant enforcement.

### Session watermark

After a write returns version 18, the client or service carries `minimumVersion=18`. A cached version below 18 triggers source routing or a wait-until-applied policy.

### Bounded stale serving

During source failure, serve a previously validated value only if its age and data class permit it. Record stale serving so availability does not silently weaken correctness.

## CAP and PACELC

Do not explain every stale cache entry with CAP. A missed invalidation between a database and a cache is normally a dual-write and propagation problem, even without a network partition affecting replicated availability.

CAP is relevant when a distributed cache or multi-region system cannot coordinate during a partition and must decide which operations continue. PACELC helps discuss normal-case latency versus consistency, such as a fast local cached read versus coordinated source validation.

See [CAP Theorem](../fundamentals/cap-theorem.md) and [PACELC](../fundamentals/pacelc.md).

## When To Cache

Cache data when the permitted semantics can be stated and implemented:

- immutable/versioned content;
- bounded-stale product or profile display;
- derived feed objects where temporary lag is acceptable;
- configuration with a documented propagation SLO;
- expensive query results used for advisory rather than authoritative decisions.

Do not cache, or cache only for display, when:

- a stale value can allocate a unique resource twice;
- revocation must take effect before access is allowed;
- an account balance is used to authorize a mutation;
- no safe reconciliation or source fallback exists;
- key construction cannot preserve tenant/security boundaries.

## Scaling Characteristics

Stronger freshness usually requires more coordination, shorter residency, more invalidations, or more source reads. These increase latency and load.

```text
local cached read
    -> lowest latency, weaker shared freshness

version/source validation
    -> higher latency/load, stronger targeted guarantee
```

Apply stronger guarantees only to the operations that need them. A product can mix authoritative mutations, read-your-writes user flows, bounded-stale displays, and eventual analytics.

## Failure Modes

### Propagation lag exceeds SLO

Route affected critical reads to the source, degrade, or fail according to the invariant. Alert on oldest version/event age.

### Version regression

An old event or failover causes a lower version to appear. Reject it or fall back to source.

### Source unavailable

Choose between bounded stale, fail open, fail closed, or degraded behavior per data class. “Use the cache” is unsafe without an age bound.

### Tenant or authorization leak

An incomplete key serves a value across identity boundaries. This is a security incident, not merely stale data.

### Cache becomes de facto authority

Teams size down or stop validating the source until cache loss makes the service unable to operate. Document ownership and test reconstruction.

## Trade-offs

- Read-your-writes improves user experience but can require sticky routing, version tokens, or source reads.
- Monotonic reads avoid visible regression but retain session state or metadata.
- Bounded stale serving improves availability but weakens freshness explicitly.
- Source validation protects invariants but reduces cache offload for critical operations.
- Shorter propagation SLOs increase event and operational requirements.

## Production Gotchas

- Put source versions and representation versions in telemetry.
- Measure propagation lag and stale-age distribution, not only hit rate.
- Define consistency SLOs by operation and data class.
- Keep cache keys tenant- and authorization-safe.
- Make event consumers idempotent and version-aware.
- Test delayed, duplicate, and reordered updates plus lagging source replicas.
- Audit which endpoints use cached data to make decisions.

## Alternatives

- Direct authoritative reads for critical paths.
- Read replicas with known consistency semantics.
- Materialized projections with version watermarks.
- Immutable/content-addressed representations.
- No cache when the coordination cost eliminates its value.

## Interview Questions

### Is this cache eventually consistent?

State how it converges, the expected and maximum lag, the repair mechanism, and which operations may observe stale data.

### How do you provide read-your-writes?

Carry the committed version and bypass, route, or wait until the read path can return at least that version.

### Should account balance be cached?

It may be cached for display with a clear age indicator, but authoritative spending or ledger decisions should validate against the system that owns the invariant.

## 2-Minute Interview Answer

> “I define cache consistency per operation. The database or owning service remains authoritative, cached values carry a source version, and product-display reads may tolerate 30 seconds of staleness. After a user update I carry the committed version so subsequent reads either return at least that version or route to the source, which provides read-your-writes. Consumers reject out-of-order versions, and event lag has a measurable SLO and reconciliation path. Critical inventory, payment, and authorization decisions revalidate against their authority. I use CAP only for actual partition behavior; ordinary cache drift is handled as asynchronous projection consistency.”

## Senior-Level Follow-ups

- How do you calculate effective staleness across source replica, events, L2, and L1?
- Can version numbers be compared across independent writers?
- What does the system do when the consistency SLO is breached?
- How do mobile clients carry a minimum version?
- Which data may serve stale during regional isolation?

## References

- [Internal: Data Consistency Boundaries](../databases/consistency-boundaries.md)
- [Internal: CAP Theorem](../fundamentals/cap-theorem.md)
- [Internal: PACELC](../fundamentals/pacelc.md)
- [Internal: Cache Invalidation](cache-invalidation.md)
- [RFC 9111 — HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111.html)

