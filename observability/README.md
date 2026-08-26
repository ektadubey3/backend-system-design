# Observability

Observability is the ability to infer a system's internal state from its outputs. Production systems need telemetry that answers user-impact, dependency, saturation, and change questions without collecting unbounded or sensitive data.

## Learning path

1. [Signals and diagnostic questions](signals-and-diagnostic-questions.md) — metrics, logs, traces, profiles, events, and when each helps.
2. [Instrumentation, context, and cardinality](instrumentation-context-and-cardinality.md) — semantic contracts, correlation, sampling, and cost control.
3. [SLO alerting and incident learning](slo-alerting-and-incident-learning.md) — actionable alerting, dashboards, runbooks, and post-incident feedback.
4. [Observability design framework](observability-design-framework.md) — interview-ready plan and telemetry budget.

## Boundary with reliability

[SLIs, SLOs, and error budgets](../reliability/slis-slos-and-error-budgets.md) is the canonical product-policy treatment. This section focuses on the telemetry pipeline and diagnostic workflow that make those objectives operable.

## Core principles

- Instrument user journeys and component boundaries, not every line of code.
- Keep stable low-cardinality dimensions in metrics; put high-cardinality detail in sampled traces or indexed logs with retention controls.
- Propagate correlation context across synchronous and asynchronous boundaries.
- Treat telemetry as a production data pipeline with schemas, capacity, security, loss policy, and its own health.
- Page only when a human must act now; dashboards and tickets serve different purposes.

