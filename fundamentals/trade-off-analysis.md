# Trade-off Analysis

Trade-off analysis is the process of evaluating different architectural choices by balancing competing quality attributes such as performance, scalability, availability, consistency, cost, maintainability, security, and complexity.

In system design, there is rarely a "best" solution. Every design decision improves one aspect while sacrificing another. A good system designer understands these trade-offs and chooses the solution that best satisfies business requirements.

## Why Trade-off Analysis Matters

Every distributed system has constraints:

* Limited budget
* Limited hardware
* Network latency
* CAP theorem limitations
* Team expertise
* Time to market

Instead of optimizing everything, engineers optimize what matters most.

Example:

If building WhatsApp:

Priority:

* Availability
* Low latency

Less priority:

* Strong consistency

Result:

* Messages may arrive slightly out of order instead of making users wait.

---

# Common System Design Trade-offs

| Trade-off                      | Option A             | Option B                |
| ------------------------------ | -------------------- | ----------------------- |
| Consistency vs Availability    | Strong consistency   | High availability       |
| Latency vs Accuracy            | Fast response        | Accurate computation    |
| Cost vs Performance            | Cheap infrastructure | High-end infrastructure |
| Simplicity vs Flexibility      | Easy maintenance     | More configurable       |
| Read vs Write Optimization     | Faster reads         | Faster writes           |
| SQL vs NoSQL                   | ACID                 | Scalability             |
| Monolith vs Microservices      | Simplicity           | Independent scaling     |
| Vertical vs Horizontal Scaling | Bigger servers       | More servers            |
| Cache Freshness vs Speed       | Fresh data           | Fast access             |
| Compression vs CPU             | Less bandwidth       | Lower CPU               |


## 1. Consistency vs Availability (CAP)

### Strong Consistency

Pros

* Latest data
* Easier reasoning
* Financial applications

Cons

* Higher latency
* Lower availability during partition

Example

Bank transfer.

Cannot show stale balance.

### High Availability

Pros

* Always responds
* Better user experience

Cons

* Temporary stale data

Example

Instagram likes.

Seeing 100 instead of 101 likes is acceptable.

## 2. Latency vs Accuracy

Example:
Recommendation engine

Fast

```
Cached recommendations
```

Accurate

```
Real-time ML inference
```

Trade-off

* Cached → 20ms
* Real-time → 500ms

Most companies choose cached recommendations.


## 3. Read Optimization vs Write Optimization

Read-heavy systems

Examples

* YouTube
* Netflix
* News websites

Design

* CDN
* Redis
* Replication
* Materialized views

Write-heavy systems

Examples

* IoT
* Logging
* Analytics

Design

* Kafka
* Cassandra
* Append-only storage


## 4. SQL vs NoSQL

SQL

Advantages

* Transactions
* ACID
* Joins
* Constraints

Disadvantages

* Harder horizontal scaling

Best for

* Banking
* ERP
* Payments


NoSQL

Advantages

* Massive scalability
* Flexible schema
* High throughput

Disadvantages

* Eventual consistency
* Limited joins

Best for

* Social media
* Gaming
* Analytics


## 5. Monolith vs Microservices

## Monolith

Pros

* Easy deployment
* Simple debugging
* Less infrastructure

Cons

* Hard to scale independently
* Large codebase

Suitable for

* Startups
* MVPs


## Microservices

Pros

* Independent deployment
* Team autonomy
* Independent scaling

Cons

* Network failures
* Distributed transactions
* Observability complexity

Suitable for

* Large organizations


## 6. Vertical vs Horizontal Scaling

Vertical Scaling

```
4 CPU
  ↓
16 CPU
```

Pros

* Simple

Cons

* Hardware limits
* Expensive


Horizontal Scaling

```
Server 1
Server 2
Server 3
Server 4
```

Pros

* Unlimited scaling
* Fault tolerance

Cons

* Distributed complexity


## 7. Cache Freshness vs Speed

Without Cache

```
  User
   ↓
Database
```

Latency

```
100 ms
```


With Cache

```
   User
    ↓
  Redis
    ↓
 Database
```

Latency

```
5 ms
```

Trade-off

* Data may be stale
* Faster response


## 8. Normalization vs Denormalization

Normalization

Pros

* Less redundancy
* Easier updates

Cons

* More joins


Denormalization

Pros

* Faster reads
* Simpler queries

Cons

* Duplicate data
* Harder consistency


## 9. Security vs Convenience

High Security

* MFA
* Frequent login
* Encryption
* Device verification

Pros

* Better protection

Cons

* Worse UX


Convenience

* Remember login
* Social login
* Auto-fill

Pros

* Better user experience

Cons

* More security risks


## 10. Durability vs Performance

High Durability

Example

```
  Write
    ↓
   Disk
    ↓
Acknowledge
```

Safer

Slower


  High Performance

  ```
      Write
        ↓
      Memory
        ↓
    Acknowledge
        ↓
    Disk later
```

Faster

Risk of data loss

Example

Logging systems.


## 11. Compression vs CPU

Compressed

Pros

* Less storage
* Lower bandwidth

Cons

* More CPU


Uncompressed

Pros

* Faster processing

Cons

* More storage


## 12. Cost vs Reliability

Single Server

Pros

* Cheap

Cons

* Single point of failure


Multiple Regions

Pros

* Highly reliable

Cons

* Expensive

---

# Examples

### Problem

Design a URL Shortener.

Requirements

* 100M URLs/day
* Fast redirects
* Highly available


### Option 1

Only SQL

Advantages

* ACID
* Simple

Problems

* Hard to scale


### Option 2

NoSQL + Cache

Advantages

* High throughput
* Horizontal scaling
* Fast lookup

Problems

* Eventual consistency
* Cache invalidation

Decision

Choose NoSQL + Redis because read latency is the top priority.

---

# Trade-off Analysis Framework

For every design decision, evaluate:

| Question                     | Example                 |
| ---------------------------- | ----------------------- |
| What problem are we solving? | Reduce latency          |
| What do we gain?             | Faster response         |
| What do we lose?             | Data freshness          |
| Can we mitigate it?          | TTL, cache invalidation |
| Is it acceptable?            | Yes for social media    |

---

# How to Answer Trade-offs in Interviews

A clear structure helps communicate your reasoning:

1. **State the requirement**

   * "The system is read-heavy, so low latency is important."

2. **Present the options**

   * "We can use either a relational database or a NoSQL database."

3. **Compare the trade-offs**

   * Discuss performance, scalability, consistency, operational complexity, and cost.

4. **Justify your decision**

   * Explain why one option best aligns with the stated requirements.

5. **Mention the downside**

   * Acknowledge the drawbacks of your choice.

6. **Propose mitigations**

   * Describe techniques to reduce those drawbacks (e.g., caching, retries, replication, monitoring).

**Example:**

> "Since this is a read-heavy application with global traffic, I'd use a NoSQL database with Redis caching. This provides low-latency reads and horizontal scalability. The trade-off is eventual consistency and additional operational complexity. To mitigate these issues, I'd use cache invalidation, replication, monitoring, and idempotent writes."

---

# Best Practices

* Start with business requirements before evaluating technologies.
* Optimize for the most important quality attributes, not every attribute.
* Be explicit about both benefits and drawbacks of each decision.
* Consider operational complexity and total cost of ownership, not just technical performance.
* Document assumptions, constraints, and risks.
* Revisit trade-offs as scale, traffic, or business priorities change.

---

# Key Takeaways

* Every architectural decision involves trade-offs; there is no universally optimal design.
* The "right" choice depends on the system's functional and non-functional requirements.
* Common trade-offs include consistency vs. availability, latency vs. accuracy, SQL vs. NoSQL, monolith vs. microservices, and cost vs. reliability.
* Strong interview answers clearly explain the **decision**, **trade-offs**, **impact**, and **mitigation strategy**, demonstrating sound engineering judgment rather than memorization.
