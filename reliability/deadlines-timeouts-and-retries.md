# Deadlines, Timeouts, and Retries

## TL;DR

A deadline is the remaining budget for an end-to-end operation; a timeout bounds one wait. Propagate the deadline, cancel work that can no longer matter, retry only transient and idempotent operations, use capped exponential backoff with jitter, and allocate one finite retry budget at one layer.

## Budget the request path

If a user request has 800 ms remaining, three sequential dependencies cannot each receive a 700 ms timeout. Reserve time for application work and response transmission, then allocate sub-budgets. Downstream services should receive the absolute deadline or remaining budget and stop work that cannot finish usefully.

```text
end-to-end deadline
  = queue wait + application work + dependency attempts + response margin
```

Connect, TLS, response-header, idle, and total-operation timeouts protect different phases. A single library default may be infinite or inappropriate for the service objective.

## What a timeout means

A timeout says the caller stopped waiting. The remote operation may not have started, may be running, or may have committed while the response was lost. Retrying a non-idempotent action with a new identity can duplicate it.

Use a stable operation ID. Store or query final status for ambiguous outcomes. Cancel downstream work when possible, while recognizing that cancellation delivery can also fail.

## Retry decision

Retry only when all are true:

- the error is plausibly transient;
- another attempt can complete before the deadline;
- the operation is idempotent or protected by a stable idempotency key;
- the dependency is not signaling overload that retry would worsen;
- the retry budget has capacity.

Do not retry validation, authorization, or deterministic not-found outcomes unless the contract says state may change within the budget. Honor explicit retry-after information.

## Backoff and jitter

Exponential backoff spaces later attempts; a cap prevents excessive delay. Jitter randomizes scheduling so a fleet does not retry in lockstep after one outage. Bound both attempts and total elapsed time.

Retries at several layers multiply. Four total attempts at each of three layers can generate `4^3 = 64` attempts at the lowest dependency. Let the highest layer that understands the user deadline and idempotency own retries, or explicitly partition one shared attempt budget.

## Hedged requests

A hedge starts a duplicate request after a high-latency threshold and accepts the first response. It can reduce tail latency for read-only or idempotent work with spare capacity, but increases load exactly when the service may be slow. Limit hedges with a small budget and cancel losers. Never use them as a default for effectful writes.

## Failure modes

- Timeout longer than the upstream deadline, so all results arrive too late.
- Retrying overload increases queueing and prevents recovery.
- New idempotency key on every attempt creates duplicate business operations.
- Fixed retry delay synchronizes a large client fleet.
- Client times out but server continues expensive orphan work.
- Aggressive hedging consumes the headroom that protected the dependency.

## Interview prompts

- What is the end-to-end deadline and how is it divided?
- Which error codes are retryable and why?
- What makes the operation idempotent across an ambiguous timeout?
- How many attempts can reach the deepest dependency?

## Two-minute answer

Set an end-to-end deadline from the user SLO, propagate remaining time, and give each hop a shorter phase-specific timeout. Treat timeouts as ambiguous. Retry only transient, idempotent operations while time and a shared retry budget remain, using capped exponential backoff with jitter and one retry-owning layer. Cancel doomed work and monitor attempts per original request, timeout phase, late completions, and retry success versus amplification.

## References

- [Google SRE — Addressing cascading failures](https://sre.google/sre-book/addressing-cascading-failures/)

