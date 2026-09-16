# Merged Documentation Review

Baseline: `37140b8c9dc3cfa36604e306c7c4473741da5188` on `main`.

Scope: the reliability, bottleneck, latency/throughput, LLD, and trade-off fundamentals; adjacent availability/fault-tolerance/PACELC notes; interview preparation; and related reliability, observability, messaging, architecture-decision, and consistency-boundary chapters. This is a focused cross-chapter review, not a line-by-line audit of every database, networking, or case-study chapter.

## Findings and Changes

Priority reflects potential to teach a wrong design or measurement decision, not production incident severity. All rows below are addressed by this change; the final section identifies remaining scope limits.

| Priority / category | Source and evidence at baseline | Why it matters | Correction |
|---|---|---|---|
| High / ambiguity | [Latency vs throughput — Retry Amplification](../fundamentals/latency-vs-throughput.md#retry-amplification) labels 3×, 2×, 2× as retries but concludes 12 attempts | Additional retries imply 36 total deepest calls, not 12 | Define total attempts, show both interpretations, link retry ownership |
| High / incomplete boundary | [LLD — Domain Invariant and Concurrency](../fundamentals/lld.md#concurrency) gives an object check plus blind `save` without an enforcing example | Two processes can both pass the local check | Add conditional version/state update, conflict semantics, provider boundary, and atomic outbox guidance |
| High / terminology | [LLD — Interfaces](../fundamentals/lld.md#interfaces) uses `authorize` beside `markPaid` | Authorization need not mean capture; timeout need not mean payment failure | Use capture for the example and explicitly define pending/unknown outcomes and omitted flows |
| High / incomplete boundary | [Reliability — Idempotency](../fundamentals/reliability.md#idempotency) says to persist the key with the result without describing races, scope, or expiry | Durable storage alone does not prevent concurrent duplicate effects | Add uniqueness, atomic local mutation, payload matching, pending state, retention, and external reconciliation |
| Medium / measurement gap | [Latency — Tail Latency and Latency Budget](../fundamentals/latency-vs-throughput.md#measurement-contract) lacks fleet aggregation and percentile-composition cautions | Averaged instance p99s or summed hop p99s can misrepresent users | Define measurement contract; aggregate distributions; measure end-to-end; distinguish goodput |
| Medium / weak explanation | [Bottlenecks — Little's Law](../fundamentals/bottleneck-identification.md#littles-law) gives only the equation; overload chapter lacks stability qualification | Readers may use p99 or growing backlog to size pools | Cross-link canonical definitions, add example and boundary, distinguish transient backlog growth |
| Medium / missing application | [Bottlenecks](../fundamentals/bottleneck-identification.md) mostly lists signals and possible fixes | Signals alone do not establish causation | Add competing hypotheses, controlled experiment, aggregate pool limit, and load-generator caveat |
| Medium / cross-example mismatch | [Trade-off URL shortener](../fundamentals/trade-off-analysis.md#decision-matrix-example--url-shortener) uses 100M/day while [capacity estimation](capacity-estimation.md#example--url-shortener) uses 100M/month | Unmarked workload changes make conclusions hard to compare | Align baseline to month, calculate peak assumptions, add a daily-volume drill variant |
| Medium / vague guarantee | [Trade-offs — Consistency vs Availability](../fundamentals/trade-off-analysis.md#1-consistency-vs-availability) labels ledger needs “Strong” | A consistency label alone does not enforce a business invariant | Specify atomic posting/isolation needs and distinguish linearizability from serializability |
| Medium / weak quantitative explanation | [SLO budgets](../reliability/slis-slos-and-error-budgets.md#burn-rate-alerting) explains burn qualitatively | Candidate cannot demonstrate budget reasoning | Add a numeric example with traffic and remaining-budget qualifications |
| Low / overlap | [Decision card](decision-frameworks.md), [trade-off catalog](../fundamentals/trade-off-analysis.md), and [architecture framework](../architecture-patterns/architecture-decision-framework.md) repeat comparison structure | Learners cannot tell which version to use | Assign rehearsal, detailed comparison, and migration/reversal roles; link instead of expanding duplicates |
| Low / overlap and navigation | Fundamentals reliability/availability/fault tolerance overlap with reliability and observability curricula; [rapid review](rapid-review.md) lists nouns without applied rehearsal | More reading does not ensure interview reasoning | Keep useful layers, add ownership links and one consolidated scenario guide with shared vocabulary |

## Topic Ownership

| Topic | Canonical explanation | Applied layer |
|---|---|---|
| Performance terms and Little's Law | [Latency vs Throughput](../fundamentals/latency-vs-throughput.md) | [Bottleneck diagnosis](../fundamentals/bottleneck-identification.md) |
| Business outcomes under failure | [Reliability fundamentals](../fundamentals/reliability.md) | [Reliability engineering](../reliability/README.md) |
| SLI definitions and budget math | [SLIs/SLOs](../reliability/slis-slos-and-error-budgets.md) | [Alerting and response](../observability/slo-alerting-and-incident-learning.md) |
| Duplicate delivery and side effects | [Delivery semantics](../messaging/delivery-semantics-and-idempotency.md) | Payment drill and LLD transaction boundary |
| Design comparison | [Trade-off analysis](../fundamentals/trade-off-analysis.md) | Decision card, architecture migration framework |
| Concise revision | [Scenario Study Guide](scenario-study-guide.md) | [Rapid Review](rapid-review.md) as final checklist |

Overlap is retained when it serves a different learning task; it is not treated as an error merely because a term recurs. The guide links to deeper material instead of copying whole chapters.

## Validation and Limits

- Technical checks use [PostgreSQL transaction isolation](https://www.postgresql.org/docs/current/transaction-iso.html), [Prometheus histogram guidance](https://prometheus.io/docs/practices/histograms/), and [Google SRE burn-rate guidance](https://sre.google/workbook/alerting-on-slos/).
- Arithmetic checked: retry bounds, concurrency, backlog growth/drain, cache-loss amplification, monthly request rates, and error-budget burn.
- Relative Markdown links and heading anchors in changed files checked against the repository; whitespace checked with `git diff --check`.
- SQL is a scoped documentation example, not an implemented or database-tested payment system. Multi-row invariants, provider contracts, and isolation-specific retries require their own design.
- Database/networking product claims outside this scope still need separate review. The new vocabulary clarifies the reviewed learning path; it does not claim every older chapter now uses identical wording.
