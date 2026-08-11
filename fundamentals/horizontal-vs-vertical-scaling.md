# Horizontal vs Vertical Scaling

Vertical and horizontal scaling are complementary tools.

The design question is:

> **Which resource is saturated, and can the workload be divided safely enough that more nodes remove that limit?**

## Interview TL;DR

| Question | Vertical | Horizontal |
|---|---|---|
| Change | Bigger node | More nodes |
| Initial complexity | Lower | Higher |
| Local transactions/state | Simple | Requires ownership/coordination |
| Hard limit | Machine/platform ceiling | Coordination/shared-resource ceiling |
| Rebalancing | Usually none | Often required |
| Typical use | First-stage DB/app growth | Stateless or partitionable workloads |

Most mature systems use both.

## Vertical Scaling

```text
8 vCPU / 32 GB
      ↓
32 vCPU / 128 GB
```

Good when:

- a cost-effective upgrade path exists;
- local transactions are valuable;
- workload does not partition cleanly;
- operational simplicity matters.

Risks:

- node ceiling;
- larger blast radius;
- nonlinear cost;
- maintenance/failover impact.

## Horizontal Scaling

```text
        Load Balancer
        /    |    \
     App1  App2  App3
```

For stateful data:

```text
key → owner/shard
```

which introduces:

- partition key;
- rebalancing;
- replication;
- consistency;
- cross-shard operations.

## Stateless Does Not Mean “No State”

A stateless request tier means durable/session state is not tied to one process.

State may live in:

- database;
- distributed cache;
- signed client token;
- object store.

## Database Scaling

```text
bigger primary
    ↓
query/index tuning
    ↓
connection pooling
    ↓
cache hot reads
    ↓
read replicas
    ↓
partition large workloads
    ↓
shard only when required
```

## Replication Is Not Sharding

Replication copies the same data for availability/read distribution.

Sharding splits ownership for dataset/write capacity.

## Autoscaling Caveat

If 10 app instances each open 30 DB connections:

```text
300 DB connections
```

Scaling to 100 instances can create:

```text
3,000 DB connections
```

Application autoscaling can overload the DB unless connection budgets are coordinated.

## When Horizontal Scaling Does Not Help

- single hot row/key;
- one leader serializes writes;
- shared DB saturated;
- global lock;
- egress ceiling;
- downstream quota;
- metadata/coordinator bottleneck.

## Decision Framework

1. Which metric is saturated?
2. Can workload be partitioned?
3. Does state need one owner?
4. What coordination appears?
5. What is the largest hotspot?
6. How are nodes added/removed?
7. What happens during partial failure?
8. Can downstream tiers absorb extra concurrency?

## Interview Answer Template

> “I scale vertically while that is the simplest economical capacity increase. Stateless request processing can scale horizontally. For databases, adding replicas is read scaling, not write sharding. I introduce partitioned ownership only when a measured single-node or single-owner limit justifies the coordination and rebalancing cost.”
