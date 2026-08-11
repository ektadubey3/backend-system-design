# Fault Tolerance and Resilience

A fault is an abnormal condition in a component. A system is fault tolerant when it can continue providing the required service despite specified faults.

Resilience is broader: it includes absorbing faults, degrading deliberately, recovering, and returning to normal operation.

## Interview TL;DR

1. Name the failure you are designing for.
2. Redundancy without failure isolation can create correlated failure.
3. Use deadlines before retries.
4. Retry only safe operations and cap the retry budget.
5. Circuit breakers can reduce repeated pressure on an unhealthy dependency; they do not repair it.
6. Bulkheads and concurrency limits isolate resource exhaustion.
7. Backpressure slows producers; load shedding rejects excess work.
8. Graceful degradation preserves critical features.
9. Failover requires authority/fencing for stateful systems.
10. Synchronous replication does **not** automatically mean zero data loss under every failure model.

## Fault → Error → Failure

```text
fault occurs
    ↓
internal error/state deviation
    ↓
user-visible failure if not contained
```

## Failure Domains

Examples:

- process;
- node;
- availability zone;
- region;
- network path;
- control plane;
- dependency;
- data corruption.

## Timeouts

Every remote call can fail by taking too long.

```text
client deadline
   ↓
gateway budget
   ↓
service budget
   ↓
DB/downstream budget
```

## Retries

Safer:

- transient connection reset;
- selected throttling/unavailable responses;
- idempotent reads.

Dangerous without protection:

- payment capture;
- order creation;
- inventory mutation.

Use limited attempts, backoff, jitter, deadline, and idempotency.

## Circuit Breaker

```text
CLOSED
  ↓ repeated failures
OPEN
  ↓ recovery probe
HALF-OPEN
  ↓
CLOSED or OPEN
```

Use when repeated failing calls consume resources or worsen recovery.

## Bulkheads

Partition capacity so one workload cannot consume everything.

```text
checkout       60%
catalog        30%
background     10%
```

Apply to threads, concurrency, queues, connections, or replicas.

## Backpressure

A downstream system causes producers to slow down.

Examples:

- bounded queue;
- stream flow control;
- broker lag triggers admission changes.

## Load Shedding

Reject lower-priority/excess work early rather than accepting work that will time out.

## Graceful Degradation

Preserve critical operations while disabling optional ones.

## Replication and Failover

### Asynchronous replication

Lower write latency, but recent acknowledged writes may not exist on a promoted replica depending on system semantics.

### Synchronous/quorum acknowledgement

Can reduce acknowledged-write loss risk, but adds latency and may reduce write availability.

Always define:

- what “acknowledged” means;
- storage durability;
- RPO;
- promotion/fencing.

## Health Checks

Distinguish:

- liveness;
- readiness;
- dependency health;
- business correctness.

An optional dependency failure should not necessarily make every instance unready.

## Recovery

```text
detect
isolate
serve degraded / fail over
repair
rejoin
reconcile
```

## Failure Testing

Test:

- instance loss;
- zone loss;
- cache outage;
- replica lag;
- queue backlog;
- slow dependency;
- DNS/TLS failure;
- retry storm.

## Common Mistakes

- “synchronous replication = no data loss”;
- retries without deadlines;
- circuit breaker as capacity planning;
- one giant shared resource pool;
- health checks causing cascades;
- failover without fencing.

## Interview Answer Template

> “I define the target fault first. Calls have deadlines and only safe transient failures are retried with jitter. Critical capacity is isolated with bulkheads; overload sheds optional work. Stateful failover needs explicit write authority and RPO, not just a replica. I also explain rejoin and reconciliation.”

## References

- [Google SRE — Addressing Cascading Failures](https://sre.google/sre-book/addressing-cascading-failures/)
- [Google SRE — Handling Overload](https://sre.google/sre-book/handling-overload/)
