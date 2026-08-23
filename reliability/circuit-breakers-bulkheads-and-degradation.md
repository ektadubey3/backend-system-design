# Circuit Breakers, Bulkheads, and Graceful Degradation

## TL;DR

Circuit breakers stop repeated calls to a dependency that is predictably failing; bulkheads prevent one dependency or traffic class from consuming every resource; graceful degradation returns a smaller truthful product. These mechanisms complement deadlines and admission control—they do not replace them.

## Circuit breaker state

```text
closed --failure threshold--> open --cooldown--> half-open
  ^                                          success | failure
  +-----------------------------------------------+  +-> open
```

Closed passes calls and measures outcomes. Open fails fast or uses a fallback. Half-open permits a small probe budget to test recovery. Use sliding windows and minimum sample counts; one error on one request should not open a low-traffic circuit indefinitely.

Scope circuits by dependency endpoint, operation, and sometimes tenant/region. A global circuit can hide a healthy shard because another shard fails. Count only relevant outcomes: client validation errors do not indicate dependency health.

## Bulkheads

Separate scarce pools so failure cannot consume everything:

- thread/async concurrency per dependency;
- connection pools per downstream;
- queues per priority or tenant tier;
- partitions for heavy workflows;
- cell-based deployments for bounded blast radius.

Isolation reduces utilization efficiency because spare capacity in one pool may not help another. That is the price of predictable failure containment. Keep a small emergency/control pool for health, cancellation, and recovery.

## Graceful degradation

A fallback must be cheaper, bounded, and semantically honest. Examples:

- serve a labeled stale catalog but never stale authorization;
- omit recommendations while checkout continues;
- accept an order into durable pending state instead of claiming completion;
- return partial search with explicit missing sources;
- disable expensive personalization during overload.

A fallback that calls the same failing database through another route is not isolation. A default “allow” on authorization failure is dangerous; security and money paths commonly fail closed.

## Recovery behavior

When a dependency returns, releasing every waiting request recreates overload. Half-open probes, concurrency ramp-up, jittered retries, and queue draining smooth recovery. Invalidate or revalidate stale fallback data according to its business policy.

## Failure modes

- Breaker opens on all 4xx responses, hiding application bugs.
- Every process probes at the same instant and overwhelms recovery.
- Fallback cache has no freshness label or authorization boundary.
- Shared connection pool lets a slow optional dependency starve critical calls.
- Static degradation flag is never tested and fails during the incident.
- Circuit hides failure until no one notices the dependency has been broken for days.

## Interview prompts

- What exact outcomes open the circuit, over which window?
- Which resource pools are isolated and why?
- Is the fallback correct when stale, incomplete, or unauthorized?
- How does traffic ramp after recovery?

## Two-minute answer

Put deadlines first, then scope a circuit breaker to the failing operation and endpoint. Open it only from meaningful failure/latency signals, fail fast, and recover through a small randomized probe budget. Bulkhead concurrency, connections, queues, and tenants so one failure cannot exhaust the service. Define a cheaper, truthful degraded product per feature, fail closed for sensitive decisions, and test both entry and recovery paths.

