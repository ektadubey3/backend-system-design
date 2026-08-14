# Retries, Dead Letters, and Backpressure

## TL;DR

Retries are traffic, not recovery magic. Classify failures, cap attempts and elapsed time, add randomized delay, and ensure only one layer owns the retry policy. A dead-letter destination is useful only when it has context, alerts, replay controls, and an owner. Backpressure must eventually reject, defer, degrade, or shed work before queues consume every resource.

## Failure classification

| Failure | Example | Policy |
|---|---|---|
| Transient | brief network reset, leader election | retry with bounded exponential backoff and jitter |
| Overload | dependency returns throttling or queue age rises | honor retry hints, reduce concurrency, shed low-priority work |
| Permanent | validation error, missing required field | do not retry unchanged input |
| Poison | deterministic consumer bug for one record | quarantine after a small attempt budget |
| Ambiguous | timeout after external request | query operation status using the same idempotency key |

Retry counts alone are weak. Also bound total elapsed time: an order action may be relevant for minutes, while a compliance export might be relevant for days.

## Delayed retry topology

Do not hold an unacknowledged record while sleeping; it ties up consumer capacity and can trigger redelivery. A common design moves failures to delay buckets or a scheduled retry topic carrying:

- original message ID and business key;
- attempt count and first-failure time;
- next-visible time;
- failure class and sanitized diagnostic context;
- original topic, partition, and position when useful.

Exponential backoff spreads attempts in time. Jitter prevents many consumers affected by the same incident from waking together. A retry budget limits additional traffic during a widespread dependency failure.

## Dead-letter workflow

A dead-letter queue (DLQ) is quarantine, not a graveyard. Define:

1. **Entry rule:** permanent failure or exhausted attempt/age budget.
2. **Ownership:** the team and runbook responsible for triage.
3. **Observability:** rate, age, reason, source, and affected business entities.
4. **Correction:** deploy a fix, repair data, or explicitly discard under policy.
5. **Replay:** rate-limited, idempotent, auditable, and targeted—not “replay all.”
6. **Retention:** long enough for response objectives, bounded for cost and privacy.

Preserve the original envelope. Do not leak secrets or full sensitive payloads into diagnostic headers.

## Backpressure from consumer to producer

Backlog is a buffer with a finite time horizon. Useful signals include oldest-message age, arrival rate, completion rate, per-partition lag, in-flight work, retry rate, and saturation of the consumer's dependencies.

Responses, roughly from least to most disruptive:

- reduce consumer prefetch/in-flight concurrency when a dependency is saturated;
- scale consumers until the partition or downstream limit is reached;
- coalesce obsolete work, such as repeated “refresh this projection” commands;
- prioritize critical topics or tenants;
- slow or rate-limit producers;
- reject low-value work with an explicit outcome;
- degrade the feature rather than destabilize the core path.

An unbounded queue converts overload into memory exhaustion and extreme latency. A bounded queue exposes pressure while the system can still choose what to protect.

## Retry-storm failure chain

```text
dependency slows -> handlers time out -> retries increase arrivals
-> queue age grows -> more timeouts -> more retries -> system cannot recover
```

Break the loop with deadlines, one retry owner, a retry budget, circuit breaking, bounded concurrency, and load shedding. See [Reliability Engineering](../reliability/README.md) for the synchronous side of the same problem.

## Interview prompts

- Which errors are retryable, and who decides?
- What is the maximum number of attempts and wall-clock age?
- How does an operator safely replay ten corrected records?
- What happens when consumers are slower than producers for six hours?

## Two-minute answer

Classify failures before retrying. Use capped exponential backoff with jitter, a total age limit, and one layer that owns a finite retry budget. Move delayed attempts out of active consumer slots. Quarantine permanent or exhausted records in a DLQ with reason, ownership, alerting, retention, and rate-limited idempotent replay. Monitor oldest age and arrival-versus-completion rate, then propagate backpressure through concurrency limits, priority, producer throttling, rejection, or degradation.

## References

- [Google SRE — Addressing cascading failures](https://sre.google/sre-book/addressing-cascading-failures/)
- [RabbitMQ — Dead letter exchanges](https://www.rabbitmq.com/docs/dlx)

