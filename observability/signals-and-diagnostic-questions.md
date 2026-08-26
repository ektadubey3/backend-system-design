# Signals and Diagnostic Questions

## TL;DR

Metrics efficiently show that a population changed, traces show the causal path of sampled operations, logs explain discrete events with context, profiles show where resources are spent, and change/audit events explain what altered the system. Design them together around questions; no single signal is “observability.”

## Signal strengths

| Signal | Best at | Weakness |
|---|---|---|
| Metrics | rates, distributions, saturation, SLO math | labels cannot carry arbitrary identity safely |
| Traces | causal latency/failure across boundaries | sampling may miss rare events; storage cost |
| Logs | detailed discrete state and diagnostics | volume/search cost; weak aggregate math |
| Profiles | CPU/allocation/lock hot paths | less business context; sampling overhead |
| Change/audit events | deployments, flags, policy/admin actions | require disciplined emission and correlation |

Start from questions: Are users failing? Which cohort? Which dependency or queue contributes? What changed? Is saturation causing amplification? Can the operation be reconciled?

## Service boundary instrumentation

For inbound requests record rate, result class, duration distribution, and in-flight/concurrency. For outbound dependencies add target, operation, timeout, attempt number, result, and duration. For queues measure accepted/completed rate, oldest age, lag per partition/group, retries, DLQ, and processing duration. For data stores measure operation class, pool wait, latency, error/timeout, saturation, and replica/repair lag.

Avoid raw URL, SQL, user ID, message ID, or exception text as metric labels. Normalize route and operation names.

## Latency distributions

Averages hide tails and bimodal behavior. Use histograms or distribution-capable metrics with buckets aligned to SLOs. Client-observed latency includes queues, network, retries, and gateway time that server-handler latency misses.

Track first attempt separately from all attempts. A service can show stable successful latency while retry amplification and user end-to-end latency deteriorate.

## Structured events

Logs should be structured records with timestamp, severity, service/version/environment, trace/span or operation ID, normalized event type, result, and sanitized diagnostics. Use stable event names that survive wording changes. Rate-limit repeated errors and preserve a counter of suppressed entries.

Audit logs are not debug logs. They need controlled writers, tamper resistance, actor/action/resource/outcome, policy version, retention, and access review.

## Telemetry failure modes

- Metrics backend outage blocks the request path.
- Per-user labels create cardinality and cost explosion.
- Sampling drops every failed or slow trace because it is decided too early.
- Retry attempts appear as independent requests, hiding amplification.
- Logs contain tokens, message bodies, or payment/personal data.
- Clock skew and missing propagation break cross-service timelines.

## Interview prompts

- Which signal tells you users are harmed, and which explains why?
- Where is queue waiting time visible?
- How do you investigate one operation without a user-ID metric label?
- What happens to the product if telemetry storage is unavailable?

## Two-minute answer

Instrument each user journey and service/dependency boundary with rate, result, latency distribution, and saturation. Metrics and SLOs show population impact; sampled distributed traces locate causal latency; structured logs supply bounded diagnostic detail; profiles explain resource use; deployment/config/audit events answer what changed. Normalize names, correlate by trace or operation ID, keep high-cardinality/sensitive fields out of metric labels, and ensure telemetry export is buffered and non-blocking with an explicit loss policy.

