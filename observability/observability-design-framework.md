# Observability Design Framework

## TL;DR

Instrument from user objective to component cause. Define good events, boundary telemetry, correlation, cardinality/sampling budgets, dashboards/alerts, and telemetry-pipeline failure behavior. This yields an interview answer that is actionable rather than “add logs and metrics.”

## Framework

### 1. Name critical journeys

For each synchronous and asynchronous journey define terminal success, correctness, latency/age threshold, eligible events, and meaningful cohorts. Link to an [SLO](../reliability/slis-slos-and-error-budgets.md).

### 2. Instrument boundaries

At API, dependency, queue, database/cache, and provider boundaries capture rate, result class, duration/age distribution, in-flight work, retries, and saturation. Separate first attempts from retry attempts and acceptance from completion.

### 3. Correlate causality

Propagate trace context through RPC and messaging. Retain business operation/message/entity versions separately for idempotency and reconciliation. Add deployment, configuration, ownership-epoch, and feature-version events.

### 4. Budget telemetry

List metric label upper bounds, event bytes/rate, trace sample policy, retention, indexing, and sensitive fields. Define redaction and access. Decide which telemetry may be dropped first.

### 5. Design investigation paths

Start with SLO/cohort impact, drill to dependency and saturation, then selected traces/logs/profiles and recent changes. Include topology and runbook links.

### 6. Alert and automate

Page on actionable burn or hard safety-margin exhaustion. Ticket slow trends. Automate well-understood bounded responses, and test both alert firing and recovery.

### 7. Operate the telemetry pipeline

Isolate exporters and collectors, bound buffers, monitor drops/backpressure/schema/cardinality/cost, and make storage/control-plane outage non-fatal to product traffic.

## Design table

| Question | Example answer |
|---|---|
| User SLI | good checkout completions / valid checkout starts |
| Primary dimensions | region, route class, client tier, release—not user ID |
| Correlation | trace ID + stable payment operation ID |
| Causes | provider latency, DB pool wait, queue age, retry ratio |
| Page | multi-window checkout SLO burn |
| Safety page | outbox/log retention headroom below recovery time |
| Sampling | 1% baseline; bounded retention of error/slow traces |
| Degradation | telemetry drops debug logs before product calls |

## Two-minute answer template

“The critical journey is [X], whose good event is [definition]. I measure it by [cohorts] and instrument ingress, dependencies, queue completion, and storage with rate/result/latency/saturation. Standard trace context crosses RPC/messages; [operation ID] supports business reconciliation. Metrics use bounded dimensions, while sampled traces and redacted structured logs carry detail under retention budgets. I page on [burn/safety margin], show dependencies and recent changes, and provide an owned runbook. Exporters/collectors are isolated and bounded, and I monitor telemetry drops, cardinality, schema, latency, and cost.”

## Further study

- [Reliability design framework](../reliability/reliability-design-framework.md)
- [Messaging design framework](../messaging/messaging-design-framework.md)
- [Security design framework](../security/security-design-framework.md)

