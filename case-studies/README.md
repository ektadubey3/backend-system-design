# System Design Case Studies

These case studies apply the handbook's concepts to interview-sized systems. They emphasize explicit assumptions and failure behavior; the numbers are sample inputs to demonstrate reasoning, not claims about a particular company's production architecture.

## How to Practice

1. Read only the prompt and requirements.
2. Design the system in 45 minutes.
3. Compare your answer with the model.
4. Score yourself using `../interview/scoring-rubric.md`.
5. Repeat with a new follow-up constraint.

## Foundation designs

1. [URL Shortener](01-url-shortener.md) — read-heavy lookup, identifiers, cache, and hot links.
2. [Distributed Rate Limiter](02-rate-limiter.md) — atomic counters, bounded overshoot, and failure policy.

## Roadmap-priority designs

3. [Notification System](03-notification-system.md) — multi-channel orchestration, preferences, retries, and provider ambiguity.
4. [Real-time Chat](04-realtime-chat.md) — connections, conversation ordering, offline sync, and presence.
5. [News Feed](05-news-feed.md) — fan-out tradeoffs, ranking, celebrity skew, and freshness.
6. [Object Storage / Dropbox](06-object-storage-dropbox.md) — chunking, metadata, synchronization, and conflict handling.
7. [Payment System](07-payment-system.md) — money invariants, ledger, idempotency, and reconciliation.
8. [Ticket Booking](08-ticket-booking.md) — contention, temporary holds, payment saga, and fairness.
9. [Analytics Pipeline](09-analytics-pipeline.md) — event contracts, partitioning, late data, replay, and serving.
10. [Video Streaming](10-video-streaming.md) — upload, transcoding, manifests, CDN delivery, and QoE.

## A consistent answer shape

Every design should make the following visible:

- functional and non-functional requirements;
- back-of-the-envelope estimates and the decision they influence;
- API/event contracts and the authority for each invariant;
- data model and partition/ordering keys;
- critical and asynchronous flows;
- failure handling, overload behavior, security, and observability;
- tradeoffs, bottlenecks, evolution triggers, and follow-up questions.

Use the [45-Minute Interview Framework](../interview/45-minute-framework.md) to practice and the [Scoring Rubric](../interview/scoring-rubric.md) to review the result.
