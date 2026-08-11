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
- [ ] Add authoritative references to existing deep-dive chapters
- [ ] Tighten oversimplified claims in CAP and trade-off material

## Phase 2 — Highest-ROI Backend Topics

### Caching

- Cache-aside, read-through, write-through, write-behind
- Invalidation and freshness
- Stampede prevention
- Hot keys
- Eviction and memory policy
- Distributed cache design

### Messaging

Build semantics before products:

- Queues vs pub/sub
- Delivery semantics
- Ordering
- Consumer groups
- Idempotent consumers
- Partitioning
- Retry and dead-letter queues
- Backpressure
- Transactional outbox
- Change Data Capture
- Kafka, RabbitMQ, SQS

### Distributed Systems

- Replication
- Partitioning and sharding
- Consistent hashing
- Quorums
- Consensus
- Leader election
- Logical clocks
- Distributed transactions
- Saga
- Multi-region data ownership

### Reliability Engineering

- Timeouts
- Retries and exponential backoff
- Jitter
- Circuit breakers
- Bulkheads
- Load shedding
- Backpressure
- Graceful degradation
- SLI/SLO/error budgets
- RPO/RTO and disaster recovery

## Phase 3 — Complete Case Studies

Prioritized order:

1. Notification Service
2. Realtime Chat
3. News Feed
4. Object Storage / Dropbox
5. Payment System
6. Ticket Booking
7. Analytics Pipeline
8. Video Streaming

Each case study must include requirements, estimation, API, data model, architecture, critical flows, scaling, failure modes, consistency, security, observability, trade-offs, and follow-up questions.

## Phase 4 — Production Architecture

- Security
- Observability
- Architecture patterns
- Multi-region systems
- Cloud implementation patterns
- Kubernetes and infrastructure only after the architectural concepts they implement

## Documentation Site

After navigation and content standards stabilize:

- Add MkDocs Material
- Add search and sidebar navigation
- Enable Mermaid
- Deploy using GitHub Pages
