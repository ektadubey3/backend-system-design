# Roadmap

The priority is interview value and system-design reasoning, not raw topic count.

## Phase 1 — Make the Repository Interview-Ready

- [x] Accurate README and curriculum status
- [x] 45-minute interview framework
- [x] Capacity-estimation guide
- [x] Decision frameworks
- [x] Question bank and scoring rubric
- [x] URL Shortener case study
- [x] Rate Limiter case study
- [x] Contribution standard
- [x] Documentation checks
- [x] Add authoritative references to existing deep-dive chapters

## Phase 2 — Highest-ROI Backend Topics

### [Caching](caching/README.md)

- [x] Cache-aside, read-through, write-through, write-behind
- [x] Invalidation and freshness
- [x] Stampede prevention
- [x] Hot keys
- [x] Eviction and memory policy
- [x] Distributed cache design

### [Messaging](messaging/README.md)

Build semantics before products:

- [x] Queues vs pub/sub
- [x] Delivery semantics
- [x] Ordering
- [x] Consumer groups
- [x] Idempotent consumers
- [x] Partitioning
- [x] Retry and dead-letter queues
- [x] Backpressure
- [x] Transactional outbox
- [x] Change Data Capture
- [x] Kafka, RabbitMQ, SQS

### [Distributed Systems](distributed-systems/README.md)

- [x] Replication
- [x] Partitioning and sharding
- [x] Consistent hashing
- [x] Quorums
- [x] Consensus
- [x] Leader election
- [x] Logical clocks
- [x] Distributed transactions — canonical coverage in `databases/sql/transactions.md`
- [x] Saga — canonical coverage in `databases/sql/transactions.md`
- [x] Multi-region data ownership

### [Reliability Engineering](reliability/README.md)

- [x] Timeouts
- [x] Retries and exponential backoff
- [x] Jitter
- [x] Circuit breakers
- [x] Bulkheads
- [x] Load shedding
- [x] Backpressure
- [x] Graceful degradation
- [x] SLI/SLO/error budgets
- [x] RPO/RTO and disaster recovery

## Phase 3 — Complete Case Studies

Prioritized order:

1. [x] [Notification Service](case-studies/03-notification-system.md)
2. [x] [Realtime Chat](case-studies/04-realtime-chat.md)
3. [x] [News Feed](case-studies/05-news-feed.md)
4. [x] [Object Storage / Dropbox](case-studies/06-object-storage-dropbox.md)
5. [x] [Payment System](case-studies/07-payment-system.md)
6. [x] [Ticket Booking](case-studies/08-ticket-booking.md)
7. [x] [Analytics Pipeline](case-studies/09-analytics-pipeline.md)
8. [x] [Video Streaming](case-studies/10-video-streaming.md)

Each case study must include requirements, estimation, API, data model, architecture, critical flows, scaling, failure modes, consistency, security, observability, trade-offs, and follow-up questions.

## Phase 4 — Production Architecture

- [x] [Security](security/README.md)
- [x] [Observability](observability/README.md)
- [x] [Architecture patterns](architecture-patterns/README.md)
- [x] [Multi-region systems](distributed-systems/multi-region-data-ownership.md)
- [x] [Cloud implementation patterns](cloud-patterns/README.md)
- [x] [Kubernetes and infrastructure](cloud-patterns/compute-containers-and-kubernetes.md) after the architectural concepts they implement

## Documentation Site

Deferred until content and navigation have had real reader feedback. These are intentionally not part of the curriculum-completion change:

- [ ] Add MkDocs Material
- [ ] Add search and sidebar navigation
- [ ] Enable Mermaid
- [ ] Deploy using GitHub Pages
