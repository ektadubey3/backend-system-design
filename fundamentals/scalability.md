# Scalability

Scalability is the ability to support growth in workload or data **while continuing to meet defined service objectives at an acceptable cost**.

“Add more servers” is not a scalability strategy until you name the bottleneck and explain how the added capacity removes it.

## Interview TL;DR

1. Identify the scale dimension: reads, writes, data, connections, tenants, fan-out, or geography.
2. Scale the constrained resource, not every box.
3. Vertical scaling is often the simplest first step.
4. Horizontal scaling introduces coordination, partitioning, load balancing, and failure complexity.
5. Replication scales some reads and availability; partitioning scales dataset/write ownership.
6. Caching removes repeated work but creates freshness and failure concerns.
7. Async processing decouples arrival rate from processing rate but creates backlog and delivery semantics.
8. Hot keys/tenants/partitions can defeat even distribution.
9. Horizontal scaling is not unlimited.
10. State the metric that triggers the next architecture change.

## What Can Grow?

```text
requests/sec
writes/sec
concurrent connections
bytes/sec
dataset
growth/day
largest tenant
fan-out per event
regions
```

A system with 100 million users but 50 writes/sec differs from one with 5 million persistent connections.

## Find the First Bottleneck

Possible limits:

- app CPU;
- database CPU/I/O;
- connections;
- WAL/binlog throughput;
- locks;
- cache hot key;
- queue partition;
- egress;
- third-party quota.

## Vertical Scaling

Benefits:

- simple;
- preserves local transactions;
- avoids partitioning.

Limits:

- hardware ceiling;
- larger failure domain;
- nonlinear cost;
- maintenance constraints.

Vertical scaling is often the cheapest early architecture.

## Horizontal Scaling

Works when:

- workload can be partitioned;
- request state is not tied to one process or has explicit ownership;
- data has a viable partition key;
- distributed failure modes are acceptable.

Costs:

- routing;
- rebalancing;
- replication;
- hot partitions;
- cross-node coordination;
- cross-shard queries/transactions.

## Read vs Write Scaling

### Read replicas

Useful for stale-tolerant reads.

They do not automatically scale:

- primary writes;
- consistency-sensitive reads;
- lock-heavy transactions.

### Partitioning / sharding

Can increase storage/write capacity.

Trade-offs:

- cross-shard work;
- rebalancing;
- skew;
- operational complexity.

## Caching

Use when:

```text
high read repetition
× expensive source read
× acceptable staleness
```

New problems:

- invalidation;
- stampede;
- hot keys;
- eviction;
- source overload during cache failure.

## Async Processing

```text
request
  ↓
persist intent/event
  ↓
acknowledge
  ↓
worker
```

Benefits: burst absorption and independent worker scale.

Costs: backlog, retries, duplicates, eventual completion.

## Hotspots

Always estimate the largest key/tenant, not only average load.

## Elasticity

Autoscaling needs:

- useful signal;
- startup lead time;
- readiness;
- draining;
- downstream capacity.

Autoscaling app instances cannot fix a saturated database.

## Evolution

```text
single service + DB
      ↓
query/index/pool tuning
      ↓
horizontal stateless tier
      ↓
cache measured hot reads
      ↓
read replicas
      ↓
partition only when ownership/capacity requires it
```

## Common Mistakes

- “horizontal scaling is unlimited”;
- sharding before query/index optimization;
- replicas for read-your-writes flows;
- autoscaling a tier whose downstream cannot scale;
- ignoring skew;
- using microservices as a scaling mechanism without an ownership need.

## Interview Answer Template

> “I’ll identify the growth dimension and first saturating resource. I prefer vertical scaling and query/index/pool improvements while one node is sufficient. I’ll horizontally scale stateless processing, use replicas for stale-tolerant reads, and partition state only when a measured single-owner limit justifies the complexity.”

## References

- [Google SRE — Handling Overload](https://sre.google/sre-book/handling-overload/)
