# Case Study: Distributed Rate Limiter

## Prompt

Design a rate limiter for a public API used by many application servers across multiple availability zones.

## Requirements

- Apply limits by API key, user, IP, endpoint, or tenant.
- Return a clear rejection when the limit is exceeded.
- Support different plans and limits.
- Work across many stateless API servers.
- Add minimal latency to the request path.
- Fail in a deliberate way if the limiter dependency is unhealthy.

## Back-of-the-Envelope Estimates

Assume 1 million requests/second at peak, 10 million active limit keys in a peak window, and a 1 ms target for a colocated decision.

```text
counter operations ~= one atomic decision per request ~= 1M/s
active state floor ~= 10M * (key + counters + expiry metadata)
```

At this scale, one global counter service or cross-region round trip is not credible. Regional enforcement, partitioned hot-key-aware state, and local coarse protection become architectural requirements.

## API and Policy Contracts

```text
check(subject, route, cost, policy_version) ->
  {allowed, remaining, retry_after, decision_id, policy_version}
```

Configuration is versioned and distributed separately from the high-rate decision path. A decision log samples ordinary outcomes but durably records policy changes and fail-open/fail-closed events.

## Policy Data Model

```text
Policy(policy_id, version, subject_scope, route_pattern,
       algorithm, capacity, refill_rate, failure_mode, effective_at)
Allocation(subject_key, region, leased_tokens, expires_at, epoch)
```

## Where It Runs

Common placements:

- CDN/edge for coarse abuse protection;
- API gateway for shared API limits;
- application for domain-specific quotas.

Large systems often use multiple layers.

## Algorithms

### Fixed Window

Simple counter per time window.

Pros: cheap and easy.

Con: allows bursts around window boundaries.

### Sliding Log

Store each request timestamp.

Pros: accurate.

Con: expensive at high traffic.

### Sliding Window Counter

Approximate a sliding window using neighboring counters.

Good balance of accuracy and storage.

### Token Bucket

Tokens replenish at a configured rate and requests consume tokens.

Supports controlled bursts and is a strong general-purpose choice.

## High-Level Architecture

```mermaid
flowchart LR
    C[Client] --> G[API Gateway]
    G --> RL[Rate Limit Decision]
    RL --> STORE[(Distributed Counter Store)]
    RL -->|allowed| API[Backend API]
    RL -->|rejected| R429[429 Response]
    RL --> M[Metrics]
```

## Key Model

```text
rate:{tenant}:{route}:{policy-version}
```

The key must match the policy boundary. Avoid a key that causes unrelated tenants or endpoints to contend on the same hot counter.

## Atomicity

A read-then-write sequence is unsafe:

```text
GET counter
counter = counter + 1
SET counter
```

Concurrent requests can oversubscribe the limit.

Use an atomic increment/expiry operation, server-side script, transaction, or purpose-built rate-limit primitive.

## Consistency

Perfect global consistency can be expensive and add cross-region latency.

Decide what the quota protects:

- login/brute-force control may favor stricter enforcement;
- ordinary API fairness may tolerate bounded overshoot;
- billing quotas may require a durable accounting path separate from fast admission control.

## Multi-Region Options

### Regional budgets

Allocate each region part of the global quota.

Pros: low latency and region independence.

Cons: unused quota in one region cannot immediately be used elsewhere.

### Central global counter

Pros: precise global policy.

Cons: cross-region latency and availability dependency.

### Hybrid

Lease quota to regions and periodically reconcile. Often a strong compromise for large global systems.

## Failure Policy

Explicitly choose:

### Fail open

Allow traffic when the limiter is unavailable.

Use when availability is more important and downstream systems can protect themselves.

### Fail closed

Reject traffic when the limiter is unavailable.

Use for sensitive operations where abuse risk exceeds availability cost.

Different endpoints may use different policies.

## Hot Keys

A massive tenant can create one hot counter. Mitigations include:

- hierarchical limits;
- regional/local token buckets;
- counter sharding with bounded approximation;
- tenant-specific partitions;
- edge enforcement before centralized enforcement.

## Response Contract

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 12
```

Expose limit metadata when useful, but do not reveal unnecessary anti-abuse details.

## Observability

Track:

- limiter latency;
- allowed/rejected counts by policy;
- dependency error rate;
- fail-open/fail-closed events;
- hot-key concentration;
- store saturation;
- policy configuration errors.

## Trade-offs

Rate limiting is not just an algorithm question. The important design decisions are enforcement scope, atomicity, acceptable overshoot, failure policy, and hot-key behavior.

## Request Walkthrough

The gateway resolves a cached versioned policy, forms the subject key, and performs one atomic token-bucket decision on the owning partition. Allowed requests continue with decision context; rejected requests return `429` and a bounded retry hint. If the store is slow, a local emergency bucket protects the backend and the endpoint-specific fail policy decides whether traffic continues. Policy refresh never replaces a newer version with an older one.

## Bottlenecks and Evolution Triggers

- Central store latency or availability dominates: move to regional budgets with bounded overshoot.
- One tenant becomes a hot key: allocate hierarchical tenant/route budgets or lease tokens to gateway cells.
- Limits must react to backend saturation: add a separate adaptive concurrency controller; avoid silently changing contractual quotas.
- Monthly billing enforcement appears: keep a durable usage ledger and reconciliation path; the fast limiter remains an admission estimate.
- Configuration fleet is large: use signed/versioned snapshots, staged rollout, and rollback with policy-decision audit.

## Interview Check

State the maximum possible overshoot under concurrency and regional leasing, whether enforcement is a security or fairness boundary, and why fail-open versus fail-closed must be selected per operation rather than globally.

## Follow-ups

- Enforce one global quota across 20 regions.
- A single tenant sends 30% of all requests.
- The counter store is down for 3 minutes.
- Add dynamic limits based on server load.
- Add a monthly billing quota without putting the billing database on the request path.
