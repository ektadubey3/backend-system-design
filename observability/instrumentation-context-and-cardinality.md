# Instrumentation, Context, Sampling, and Cardinality

## TL;DR

Instrumentation is a versioned contract. Use common semantic names, propagate trace context through RPC and messages, preserve business operation IDs separately, control metric-label cardinality, and sample deliberately. Telemetry must be safe, affordable, and useful during the failure that produces the most data.

## Context propagation

A trace ID correlates a distributed operation; span IDs encode parent/child work. Instrumentation injects context into outbound carriers and extracts it at receivers. For asynchronous messages, link processing to the producer span or represent causal links when one batch has many parents. Do not assume thread-local context survives async/task boundaries without library support.

Trace context is not business idempotency. A retry may use a new trace but the same payment operation ID. Keep request/operation/message IDs as separate, validated fields.

Treat propagated baggage as untrusted cross-service input: allowlist keys, cap size, avoid sensitive data, and prevent callers from forging internal authorization or sampling policy.

## Semantic conventions

Standardize service identity, environment, version, route, operation, dependency, result/status class, messaging destination, and database system. A metric renamed differently by each team cannot power cross-service dashboards or alerts. Version internal conventions and test instrumentation in integration paths.

## Cardinality budget

Metric time series roughly multiply across label values. A metric with method (5), route (100), status (10), region (5), version (10), and tenant (1M) is not bounded by the small dimensions. Never put unbounded identifiers in labels.

Use:

- stable route templates instead of raw paths;
- status/error classes instead of exception strings;
- coarse tenant tier instead of tenant ID;
- exemplars linking a metric bucket to selected traces;
- logs/traces for high-cardinality operation IDs under retention/access policy;
- top-K or heavy-hitter pipelines when named offenders matter.

Set per-service series, event, byte, and retention budgets and alert before collectors/backends shed unpredictably.

## Sampling

Head sampling decides at trace start and is cheap, but cannot know the final outcome. Tail sampling can retain slow/error/rare traces after observing them, at buffering and collector cost. Combine a small baseline sample with policies for errors, high latency, rare routes, and selected cohorts—while keeping the decision bounded during an incident.

Sampling is not suitable for exact audit, billing, or security-event counting. Those require durable purpose-built events. Record sample rate so statistical use can weight correctly where appropriate.

## Export pipeline

Application instrumentation sends through bounded queues/batches to collectors. Export should time out, drop according to priority, and never exhaust product worker pools. Collectors add resource metadata, redact, sample, batch, and route to storage. Monitor dropped telemetry, queue utilization, export latency, backend throttling, schema violations, and cost.

## Failure modes

- Trace headers from the public internet are trusted as internal authorization context.
- One exception message creates millions of metric series.
- Tail sampler runs out of memory during the incident it was meant to observe.
- Instrumentation records full SQL or message payloads by default.
- Batch exporter blocks shutdown or drops every last incident event without counters.
- Different services use “success” for accepted versus completed async work.

## Interview prompts

- Which context crosses a queue and how is it trusted?
- What is the cardinality upper bound of every metric dimension?
- How are errors retained if head sampling missed the trace?
- Can telemetry failure consume product capacity?

## Two-minute answer

Adopt versioned semantic conventions and instrument ingress, egress, queues, and storage consistently. Propagate standard trace context but keep business operation/idempotency identity separate and treat baggage as bounded untrusted input. Budget metric cardinality and use templates/classes, sending high-cardinality detail to controlled logs/traces. Mix baseline head sampling with bounded tail policies for errors/latency. Export asynchronously through isolated bounded collectors with redaction, backpressure, loss counters, and health telemetry.

## References

- [OpenTelemetry — Context propagation](https://opentelemetry.io/docs/concepts/context-propagation/)
- [OpenTelemetry specification overview](https://opentelemetry.io/docs/specs/otel/overview/)

