# SLIs, SLOs, and Error Budgets

## TL;DR

An SLI measures user-visible behavior; an SLO sets the target over a window; the error budget is the tolerated fraction of bad events. Choose critical journeys, count good events over eligible events, segment where experiences differ, and alert on budget burn fast enough to act. An SLO is product policy, not the same as a provider SLA.

## From user journey to indicator

Examples:

| Journey | Good event | Eligible event |
|---|---|---|
| Read profile | correct response under 300 ms | valid profile reads |
| Submit payment | durable, queryable accepted/rejected result under 2 s | valid submissions |
| Process notification | delivered to provider within 60 s | accepted notification jobs |
| Search | response under 800 ms with minimum result quality | valid search requests |

Use a ratio:

```text
SLI = good eligible events / all eligible events
error budget fraction = 1 - SLO target
```

For a 99.9% event-based SLO, 0.1% of eligible events may be bad during the window. Time-based “minutes down” can hide partial failures; request/event ratios usually reflect user experience better.

## Define eligibility and correctness

Exclude invalid client requests only with a stable rule. A server returning a fast incorrect result is not good. For asynchronous workflows, measure from durable acceptance to meaningful completion and expose terminal failures, not only queue publication.

Segment by region, endpoint class, tenant tier, or operation when aggregation hides a harmed cohort. Avoid an SLO per low-volume endpoint if the sample is too noisy; group around critical journeys.

## Window and budget policy

A rolling 28- or 30-day window reflects recent behavior continuously; a calendar window aligns to reporting but resets abruptly. The organization decides what happens when budget is consumed: pause risky launches, require reliability work, or accept the risk explicitly. The budget is not permission to cause random outages; it balances change velocity with reliability investment.

## Burn-rate alerting

Burn rate compares actual bad-event rate with the rate that would exactly consume the budget over the SLO window. High burn over a short and a longer confirmation window detects severe incidents quickly without paging on momentary noise. Lower burn over longer windows catches slow exhaustion.

Alert when a human can take a defined action. Ticket on long-term risk; page on imminent user-impacting budget loss. Keep cause metrics—CPU, queue depth, replica lag—for diagnosis, but page primarily on symptoms or exhaustion of a directly protective resource.

## SLO versus SLA

An internal SLO guides engineering decisions. An SLA is an external or contractual commitment with specific measurement and consequences. Keep internal targets tighter than contractual limits so teams have response margin.

## Failure modes

- Measuring load balancer 200 responses while the body contains application errors.
- Averaging latency instead of counting requests below the user threshold.
- One global metric hides a failed region or customer tier.
- SLO target chosen as 100%, leaving no rational change budget.
- Page fires on CPU although users and safety margins are healthy.
- Async job considered successful when queued, even if it expires unprocessed.

## Interview prompts

- What user journey does the SLI represent?
- What belongs in the denominator?
- How fast would this incident consume the budget?
- What engineering decision changes when budget is exhausted?

## Two-minute answer

Select a critical journey and define a good event with correctness plus a latency or completion threshold. Compute good over eligible events, segment meaningful cohorts, and set a product-approved target and rolling window. The remaining bad-event fraction is the error budget. Use multi-window burn-rate alerts for actionable fast and slow exhaustion, diagnose with cause metrics, and attach a written release/reliability policy to budget consumption. Keep contractual SLA measurement separate.

## References

- [Google SRE Workbook — Alerting on SLOs](https://sre.google/workbook/alerting-on-slos/)
- [Google SRE Book — Service level objectives](https://sre.google/sre-book/service-level-objectives/)

