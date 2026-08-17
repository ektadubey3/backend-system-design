# Coordination, Leases, and Fencing

## TL;DR

A lease limits how long an authority grants ownership; it cannot force a paused old holder to stop. Every protected write should include a monotonically increasing fencing token that the resource rejects if stale. Prefer idempotent work queues, conditional writes, or partition ownership over a distributed lock when possible.

## The paused-holder problem

```text
worker A gets lease token 41
A pauses longer than lease
worker B gets token 42 and writes
A resumes and writes stale work
```

Both workers may sincerely believe they are allowed to act. TTL alone does not preserve safety. The target resource must remember the largest accepted token and reject token 41 after observing 42.

## Lease contract

Define:

- authority that issues monotonically increasing epochs/tokens;
- lease duration and renewal deadline;
- whether expiry is evaluated only by the authority;
- maximum clock uncertainty if time-based assumptions are used;
- what the holder must stop doing when renewal fails;
- how every external mutation is fenced;
- how abandoned work is detected and resumed.

Use a session tied to the coordination service where possible, but assume process pauses and delayed packets still occur.

## Better primitives first

| Need | Prefer |
|---|---|
| Prevent lost update | conditional write / compare-and-swap on version |
| Ensure a job eventually completes | durable queue plus idempotent handler |
| Serialize one entity | partition leader or database row transaction |
| Elect one scheduler | consensus-backed lease plus fencing |
| Avoid duplicate cron effects | unique operation key in business storage |

A lock around a remote API call is fragile: the lease may expire while the remote call continues, and the API may not understand the fencing token. Use provider idempotency and reconciliation instead.

## Fencing implementation

The coordinator increments a token on every successful acquisition. The storage operation includes it:

```text
UPDATE resource
SET value = ?, last_fence = 42
WHERE id = ? AND last_fence < 42;
```

For object storage or another service, use a conditional version/etag or place a fencing gateway in the write path. If the protected resource cannot validate ownership, the “lock” offers best-effort coordination, not a hard safety guarantee.

## Lock service availability

During loss of coordinator quorum, do not grant new exclusive leases. Existing lease behavior depends on protocol and clock assumptions; conservative clients stop when renewal cannot be confirmed. Separate lock-service availability from business-service correctness.

Avoid deleting a lock by name without verifying its owner token. Otherwise a timed-out holder may delete a newer holder's lease.

## Failure modes

- Lease duration chosen from average pause time rather than a safe bound.
- Renewal thread stays healthy while the worker is wedged.
- Old owner writes to a resource that does not check the fencing token.
- Failover resets the token counter and makes old tokens appear current.
- One coarse global lock serializes unrelated work and expands blast radius.
- Cleanup releases another holder's renewed lock.

## Interview prompts

- Can the protected resource reject a stale owner?
- What happens if the holder pauses after checking the lease?
- Is a conditional write sufficient instead of a lock?
- How does token monotonicity survive coordinator failover?

## Two-minute answer

First try to remove the distributed lock using partition ownership, a unique operation key, or a compare-and-swap. If exclusive coordination remains necessary, obtain a lease from a consensus-backed authority that returns a monotonically increasing fencing token. Include that token with every protected mutation and have the resource reject older tokens. Stop on failed renewal, never use client wall time as authority, release only with owner/token validation, and define safe behavior when coordinator quorum is unavailable.

## Further study

- [SQL locks: distributed locks and fencing](../databases/sql/locks.md)
- [Consensus and leader election](consensus-and-leader-election.md)

