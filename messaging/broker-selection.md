# Broker Selection: Kafka, RabbitMQ, and SQS

## TL;DR

Choose by semantics and operational constraints, not brand recognition. Kafka centers a retained partitioned log and consumer positions. RabbitMQ centers exchanges, queues, routing, acknowledgements, and broker-managed delivery. Amazon SQS centers a managed queue with Standard and FIFO modes. All can deliver work; their natural replay, ordering, routing, and operations models differ.

## Comparison

| Dimension | Kafka-style log | RabbitMQ-style broker | SQS-style managed queue |
|---|---|---|---|
| Natural model | retained append log | routed messages into queues | managed work queue |
| Consumer progress | offsets per group | acknowledgements remove/settle deliveries | visibility timeout plus delete |
| Replay | reset/read retained positions | usually republish or use retained stream feature | redrive retained messages, not arbitrary log navigation |
| Ordering | within a partition | queue order can be affected by concurrency/redelivery | best-effort Standard; FIFO scope for FIFO queues |
| Routing | topics and keys | exchanges, bindings, routing keys | queue plus optional SNS fan-out |
| Scaling unit | partitions | queues/consumers and broker topology | service-managed; FIFO grouping affects parallelism |
| Operations | cluster/storage/partition planning unless managed | broker topology, memory/disk, queue type | minimal broker operations; cloud quotas and semantics |

The table describes default mental models, not every optional feature.

## Choose from requirements

Favor a retained log when consumers need independent positions, long replay, stream processing, or an ordered change history. Favor a routed broker when flexible delivery topology, per-message routing, and classic task queues dominate. Favor a managed queue when the team wants a simple operational surface and its cloud coupling, quotas, latency, payload, and ordering model are acceptable.

Ask:

1. Must consumers replay arbitrary historical positions?
2. Is ordering per entity, per queue, or unnecessary?
3. Is fan-out independent and durable per subscriber?
4. How much backlog and retention must be stored?
5. Are delayed delivery, priorities, request/reply, or routing patterns central?
6. What failure domain and operator burden can the team own?
7. What are payload, throughput, partition/group, and API quotas?

## Important semantic corrections

Publisher confirms and consumer acknowledgements protect different hops; one does not imply the other. A visibility timeout makes a message temporarily unavailable, not completed; deletion/acknowledgement after work matters. FIFO or transactional broker features do not automatically make an external side effect exactly once. Consumer concurrency can reorder completion even when delivery order is stable.

## Migration and portability

A thin wrapper can hide API calls but not the model. Code that depends on offset rewind, exchange bindings, FIFO group IDs, or visibility extension carries semantic coupling. Document these assumptions explicitly rather than claiming broker independence.

Keep the business message envelope portable: stable event ID, business key, type, version, occurrence time, trace context, and payload. Put provider-specific delivery fields at the adapter boundary.

## Failure modes

- Selecting Kafka for a small task queue, then inheriting partition and retention operations with no replay need.
- Selecting a queue for an audit log, then discovering acknowledged history cannot be navigated.
- Treating one broker cluster as a failure-proof boundary.
- Ignoring per-key hot spots, cloud quotas, disk headroom, or cross-region data transfer.
- Comparing headline throughput without message size, durability acknowledgements, consumer work, and failure behavior.

## Interview prompts

- Why is replay a data-model requirement rather than a checkbox?
- Which ordering scope limits consumer parallelism?
- What application guarantees remain after selecting a FIFO queue?
- How would the choice change if the team cannot operate stateful clusters?

## Two-minute answer

First establish queue versus pub/sub versus retained-log semantics. Then compare replay, ordering scope, acknowledgement model, routing, retention, operational ownership, failure domains, and quotas. Kafka is natural for retained partitioned histories and independent consumer positions; RabbitMQ for routed queues and broker-managed delivery; SQS for a managed queue model. Regardless of product, design idempotent consumers, bounded retries, backpressure, schema evolution, and observability.

## References

- [Apache Kafka — Documentation](https://kafka.apache.org/documentation/)
- [RabbitMQ — Documentation](https://www.rabbitmq.com/docs)
- [Amazon SQS — Queue types](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-queue-types.html)

