# CAP Theorem

The **CAP Theorem** (also known as **Brewer's Theorem**) states that a distributed system **cannot simultaneously guarantee all three** of the following properties:

* **Consistency (C)** – Every read receives the most recent write or an error.
* **Availability (A)** – Every request receives a non-error response, even if it is not the latest data.
* **Partition Tolerance (P)** – The system continues operating despite network failures (partitions) between nodes.

Since **network partitions are unavoidable in distributed systems**, a system experiencing a partition must choose between **Consistency** and **Availability**.

> **Rule:** During a network partition, you can have either **CP** or **AP**, but not both.


## Why and When Should You Use It?

CAP Theorem is not something you "implement." Instead, it is a **design principle** used to make architectural decisions when building distributed systems.

Use CAP when designing:

* Distributed databases
* Microservices
* Multi-region deployments
* Cloud-native applications
* Distributed caches
* Event-driven architectures
* Replicated storage systems

### Choose CP when:

* Data correctness is critical.
* Financial systems
* Banking
* Inventory management
* Healthcare
* Payment systems

Example:
A bank account should never show two different balances.

### Choose AP when:

* System uptime is more important.
* Social media
* Messaging apps
* Recommendation systems
* Analytics platforms

Example:
Seeing an older Facebook like count is acceptable.

---

# Core Concepts

### 1. Consistency (C)

Every client sees the same data after a write.

Example:

```
Node A = Balance = $100
Write → $150

Immediately:

Node B → $150
Node C → $150
Node D → $150
```

All nodes return the latest value.


### 2. Availability (A)

Every request gets a response.

Even if some nodes are down:

```
Client
   |
   v
Server responds

✓ Success
```

The response may contain stale data.


### 3. Partition Tolerance (P)

Nodes continue working despite communication failures.

Example:

```
        Network Failure

Node A  XXXXXXXXX  Node B
```

Although Node A cannot communicate with Node B, both continue serving requests.

---

## Why Partition Tolerance is Mandatory

In cloud environments:

* Machines fail
* Switches fail
* Routers fail
* Network latency occurs
* Regions become unreachable

Therefore, **P is not optional**.

The real choice is:

* CP
* AP

---

# Architecture

Example distributed database:

```
              Client
                  |
            Read / Write
                  |
        --------------------
        |                  |
      Node A            Node B
        |                  |
        ------Network-------
             Partition
        --------------------
        |                  |
      Node C             Node D
```

During partition:

### CP System

```
        Client
          |
        Write
          |
        Node A

  Node B unavailable

    Reject request
```

Prioritizes correctness.

---

### AP System

```
            Client
              |
            Write
              |
            Node A

  Node B still serves requests

    Eventually synchronized
```

Prioritizes availability.

---

## Pros
**Helps architects make trade-offs** - No unrealistic expectations.

**Improves scalability** - Works well with distributed architectures.

**Enables fault tolerance** - Systems continue operating during failures.

**Guides database selection** - Helps decide between SQL and NoSQL solutions.

**Essential for cloud systems** - Used extensively in Kubernetes, cloud databases, and distributed storage.

## Cons

**Impossible to achieve all three simultaneously** - One property must be sacrificed during a partition.

**Eventual consistency complexity** - Applications must handle stale data.

**Higher development complexity** - Conflict resolution becomes necessary.

**Increased latency (CP systems)** - Waiting for replicas can slow down writes.

**Operational complexity** - Replication and synchronization add overhead.

---

# Comparison

| Property            | CP                    | AP                                           |
| ------------------- | --------------------- | -------------------------------------------- |
| Consistency         | ✅ Strong              | ❌ Eventual                                   |
| Availability        | ❌ May reject requests | ✅ Always responds                            |
| Partition Tolerance | ✅                     | ✅                                            |
| Stale Data          | No                    | Possible                                     |
| Best For            | Banking, Payments     | Social Media, Analytics                      |
| Examples            | HBase, ZooKeeper      | Cassandra, DynamoDB (default behavior), Riak |

> Note: The labels "CP" and "AP" describe behavior **during a network partition**. Many modern databases let you tune consistency levels or exhibit different behaviors depending on configuration and operation.

---

# Real-world Examples

### Banking System (CP)

```
Account Balance

Node A = $500

Write

Node B unreachable

Reject transaction
```

Why?

Incorrect balances are unacceptable.


### WhatsApp Messaging (Mostly AP)

Messages may arrive slightly later.

Availability is more important.


### Facebook Likes (AP)

Like count:

```
1000
```

Another user:

```
998
```

Eventually:

```
1000
```

Acceptable.

### DNS (AP)

Some DNS servers may return older records briefly.

Eventually all replicas synchronize.

### Distributed Cache (AP)

Redis replicas or distributed caches may briefly return stale values.

Eventually synchronized.

---

# Best Practices

* Accept that network partitions can occur and design for them.
* Choose consistency requirements based on business needs.
* Use eventual consistency only when stale data is acceptable.
* Design applications to handle retries and transient failures.
* Use idempotent operations for distributed writes.
* Replicate data across multiple availability zones or regions.
* Monitor replication lag and partition events.
* Clearly document consistency guarantees for API consumers.

---

# Common Mistakes

**Assuming CA systems exist in distributed environments** - Without partition tolerance, systems are not practically distributed.

**Believing AP means inconsistent forever** - Most AP systems provide **eventual consistency**, not permanent inconsistency.

**Ignoring business requirements** - Not every application needs strong consistency.

**Using eventual consistency for financial data** - This can result in incorrect transactions or balances.

**Confusing consistency with ACID consistency** - CAP consistency means **all replicas observe the same value**. ACID consistency refers to **database integrity constraints**. They are different concepts.

---

# Interview Questions

### 1. What is CAP Theorem?

A distributed system can guarantee at most two of Consistency, Availability, and Partition Tolerance during a network partition.

### 2. Who proposed CAP Theorem?

Eric Brewer proposed it in 2000, and it was later formally proven by Seth Gilbert and Nancy Lynch.

### 3. What is Consistency?

Every read returns the latest write or an error.

### 4. What is Availability?

Every request receives a non-error response.

### 5. What is Partition Tolerance?

The system continues operating despite network failures between nodes.

### 6. Why is Partition Tolerance mandatory?

Because network failures are inevitable in distributed systems.

### 7. What is a CP system?

A system that prioritizes Consistency over Availability during a partition.

### 8. What is an AP system?

A system that prioritizes Availability over strong Consistency during a partition.

### 9. Can a distributed system be CA?

Only if there is **no network partition**. In real-world distributed systems, partitions must be assumed possible, so systems are generally designed as CP or AP during partitions.

### 10. Give examples of CP databases.

* Apache HBase
* Apache ZooKeeper

### 11. Give examples of AP databases.

* Apache Cassandra
* Amazon DynamoDB (with its default eventually consistent reads)
* Riak

### 12. What is eventual consistency?

All replicas eventually converge to the same value if no new updates occur.

### 13. Does CAP apply to monolithic applications?

Primarily no. CAP is relevant to distributed systems where network partitions are possible.

### 14. Is SQL always CP?

No. SQL databases are not inherently CP or AP. Their behavior depends on how they are deployed (e.g., standalone vs. distributed) and their replication and consensus mechanisms.

### 15. Which is more important: CP or AP?

It depends entirely on business requirements.

---

# Key Takeaways

1. **CAP Theorem is about trade-offs in distributed systems, especially during network partitions.**
2. **Partition Tolerance is effectively mandatory in real-world distributed architectures.**
3. **During a partition, systems choose between strong Consistency (CP) and high Availability (AP).**
4. **CP is preferred for correctness-critical domains like banking and payments, while AP fits user-facing systems where temporary stale data is acceptable.**
5. **Modern distributed databases often allow configurable consistency levels, so CAP should be viewed as a framework for making architectural decisions rather than rigid product categories.**
