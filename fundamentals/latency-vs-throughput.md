# Latency vs Throughput

**Latency** and **Throughput** are two of the most important performance metrics in system design. They measure different aspects of a system's performance and often have a trade-off relationship.

* **Latency** measures **how long it takes** for a single request to be processed.
* **Throughput** measures **how many requests** a system can process over a period of time.

A system can have:

* Low latency but low throughput
* High throughput but high latency
* Low latency and high throughput (ideal, but difficult and expensive)

Understanding the difference is critical for designing scalable, responsive, and efficient distributed systems.

---

## Why and When Should You Use It?

Understanding latency and throughput helps you optimize systems based on business requirements.

Use **Latency** as the primary metric when:

* Users expect immediate responses
* Building real-time applications
* Interactive web applications
* Online gaming
* Financial trading
* Video calls
* Search engines

Use **Throughput** as the primary metric when:

* Processing massive amounts of data
* Batch processing
* Data pipelines
* ETL jobs
* Log processing
* Video encoding
* Email delivery
* Analytics platforms

Sometimes both are equally important.

Examples:

| Application       | Priority                      |
| ----------------- | ----------------------------- |
| Google Search     | Low Latency                   |
| Online Banking    | Low Latency                   |
| Stock Trading     | Ultra-low Latency             |
| Netflix Streaming | Low Latency + High Throughput |
| Apache Kafka      | High Throughput               |
| Hadoop            | High Throughput               |
| Payment Gateway   | Low Latency                   |
| Data Warehouse    | High Throughput               |

---

# Core Concepts

## 1. Latency

Latency is the **time taken to complete one request**.

Formula:

```text
Latency = Response Time = End Time − Start Time
```

Example:

```text
User Request

↓

Server Processing (20 ms)

↓

Database Query (30 ms)

↓

Response (10 ms)

Total Latency = 60 ms
```

Lower latency means faster responses.

Example:

| Latency | User Experience  |
| ------- | ---------------- |
| 20 ms   | Excellent        |
| 100 ms  | Good             |
| 500 ms  | Noticeable delay |
| 2 sec   | Poor             |

---

## 2. Throughput

Throughput is the **number of requests processed per unit time**.

Formula:

```text
Throughput = Requests Completed / Second
```

Example:

```text
Server

1 second

↓

Processes 15,000 requests

Throughput = 15,000 Requests/sec
```

Higher throughput means the system can handle more work.

---

## 3. Bandwidth vs Throughput

Bandwidth is the **maximum possible capacity**, while throughput is the **actual achieved capacity**.

Example:

```text
Network Capacity = 1 Gbps

Actual Data Transfer = 650 Mbps

Bandwidth = 1 Gbps

Throughput = 650 Mbps
```

---

## 4. Response Time

Latency is a major part of response time.

```text
Response Time =           Network Delay
                                +
                           Queue Time
                                +
                          Processing Time
                                +
                          Database Time
                                +
                          Serialization
                                +
                           Transmission
```

---

## 5. Queueing Delay

When requests arrive faster than they can be processed, they wait in a queue.

```text
  Incoming Requests

      ↓↓↓↓↓↓↓↓↓

        Queue

         ↓↓↓

        Server
```

As queue length increases:

* Latency increases
* Throughput eventually plateaus
* Users experience delays

---

## 6. Concurrency

Concurrency allows multiple requests to be processed simultaneously.

Example:

Without concurrency:

```text
Req1

Req2

Req3

Req4
```

With concurrency:

```text
Req1 ──┐

Req2 ──┤

Req3 ──┤

Req4 ──┘
```

Concurrency generally increases throughput, though poor implementation may increase latency due to contention.

---

## 7. Parallelism

Parallelism executes tasks on multiple CPU cores or machines at the same time.

```text
CPU1 → Req1

CPU2 → Req2

CPU3 → Req3

CPU4 → Req4
```

Benefits:

* Higher throughput
* Potentially lower latency for independent workloads

---

## 8. Little's Law

A key performance relationship:

```text
L = λ × W
```

Where:

* **L** = Average number of requests in the system
* **λ (Lambda)** = Throughput (requests/second)
* **W** = Average latency (seconds)

Example:

```text
Throughput = 200 req/sec

Latency = 0.1 sec

L = 200 × 0.1 = 20 requests
```

---

# Architecture

Example architecture balancing latency and throughput:

```text
                   Users
                     │
              Global Load Balancer
                     │
          ┌──────────┴──────────┐
          │                     │
     App Server 1          App Server 2
          │                     │
          └──────────┬──────────┘
                     │
                 Cache (Redis)
                     │
          ┌──────────┴──────────┐
          │                     │
     Database Cluster      Message Queue
          │                     │
          └──────────┬──────────┘
                     │
              Background Workers
```

How components help:

* **Load Balancer** → Distributes traffic, improving throughput.
* **Cache** → Reduces database access, lowering latency.
* **Database Replicas** → Improve read throughput.
* **Message Queue** → Smooths traffic spikes and improves overall throughput.
* **Background Workers** → Offload long-running tasks to keep user-facing latency low.

---

## Pros

#### Optimizing for Low Latency

* Better user experience
* Faster response times
* Ideal for interactive applications
* Reduces perceived wait time
* Improves customer satisfaction

#### Optimizing for High Throughput

* Handles more users
* Better resource utilization
* Higher scalability
* Efficient batch processing
* Lower infrastructure cost per request

## Cons

#### Prioritizing Low Latency

* Higher infrastructure costs
* May require over-provisioning
* More complex caching strategies
* Can reduce overall throughput if resources are dedicated to minimizing delay

#### Prioritizing High Throughput

* Higher response times
* Queueing delays
* Poor experience for interactive users
* Risk of bottlenecks under bursty traffic

---

# Comparison

| Feature      | Latency                     | Throughput                      |
| ------------ | --------------------------- | ------------------------------- |
| Measures     | Time per request            | Requests per unit time          |
| Unit         | ms, µs, seconds             | Requests/sec, Transactions/sec  |
| Goal         | Faster responses            | More completed work             |
| Focus        | Individual request          | Overall system capacity         |
| Better Value | Lower                       | Higher                          |
| Affected By  | Network, CPU, I/O, queueing | Hardware, parallelism, scaling  |
| User Impact  | Directly noticeable         | Mostly visible during high load |

---

### Analogy
Imagine a supermarket.

##### Latency
* Time a single customer spends from entering the checkout line to leaving.

##### Throughput

* Number of customers served in one hour.

Adding more checkout counters increases throughput, but if each cashier is slow, latency remains high.

---

# Real-world Examples

## Google Search

* Primary goal: Ultra-low latency
* Typical response: *<200 ms*
* Uses caching, indexing, and globally distributed data centers.

---

## Netflix

* Startup latency should be minimal.
* Streaming requires sustained high throughput to deliver video without buffering.
* Uses CDNs, adaptive bitrate streaming, and caching.

---

## Amazon

* Product search prioritizes latency.
* Order processing and inventory synchronization prioritize throughput.

---

## Apache Kafka

* Designed for extremely high throughput.
* Can process millions of messages per second with acceptable latency.
* Used for event streaming and log aggregation.

---

## Payment Gateways

* Authorization requests require low latency.
* Daily settlement and reconciliation jobs prioritize throughput.

---

## Hadoop/Spark

* Batch analytics systems optimize throughput over response time.
* Jobs may run for minutes or hours while processing petabytes of data.

---

# Best Practices

* Clearly define whether latency or throughput is the primary business goal.
* Use caching to reduce latency.
* Scale horizontally to improve throughput.
* Employ asynchronous processing for long-running tasks.
* Use message queues to absorb traffic spikes.
* Optimize database queries and indexing.
* Reduce unnecessary network hops.
* Apply load balancing across healthy instances.
* Monitor P50, P95, and P99 latency rather than only averages.
* Track throughput under normal and peak load.
* Perform load, stress, and endurance testing before production.
* Use autoscaling to maintain throughput during traffic bursts.

---

# Common Mistakes

* Assuming high throughput automatically means low latency.
* Measuring only average latency while ignoring tail latency (P95/P99).
* Ignoring queueing delays during peak traffic.
* Optimizing prematurely without identifying bottlenecks.
* Using synchronous processing for tasks that could be asynchronous.
* Overloading a single database instead of scaling reads and writes.
* Focusing only on CPU utilization while neglecting disk or network bottlenecks.
* Not testing the system under realistic production loads.

---

# Interview Questions

#### 1. What is the difference between latency and throughput?

**Answer:**
Latency measures the time taken to process a single request, whereas throughput measures the number of requests processed per unit time.

---

#### 2. Can a system have low latency but low throughput?

**Answer:**
Yes. A system may respond very quickly to a few requests but lack the capacity to handle many concurrent requests.

---

#### 3. Can increasing throughput increase latency?

**Answer:**
Yes. As the system approaches its capacity, requests begin to queue, increasing response times even though throughput may continue to rise until saturation.

---

#### 4. How does caching affect latency and throughput?

**Answer:**
Caching reduces response time by avoiding repeated expensive operations, lowering latency. It also reduces backend load, enabling the system to handle more requests, thereby improving throughput.

---

#### 5. What is Little's Law?

**Answer:**
Little's Law states:

```text
L = λ × W
```

It relates the average number of requests in a system (**L**) to throughput (**λ**) and average latency (**W**), helping engineers estimate system capacity and queue sizes.

---

#### 6. Why is P99 latency more important than average latency?

**Answer:**
Average latency can hide slow outliers. P99 latency measures the response time below which 99% of requests complete, providing a better indicator of worst-case user experience.

---

#### 7. How can you improve throughput?

**Answer:**

* Horizontal scaling
* Load balancing
* Parallel processing
* Asynchronous messaging
* Efficient resource utilization
* Database sharding and replication
* Batch processing

---

#### 8. How can you reduce latency?

**Answer:**

* Add caching
* Optimize algorithms
* Reduce network hops
* Tune database queries
* Use CDNs
* Keep data closer to users
* Minimize serialization/deserialization overhead

---

#### 9. Is higher concurrency always better?

**Answer:**
No. Beyond a certain point, excessive concurrency causes resource contention, context switching, and queueing, which can increase latency and even reduce throughput.

---

#### 10. How do load balancers help with latency and throughput?

**Answer:**
Load balancers distribute requests across multiple servers, preventing overload on individual instances. This increases throughput and can reduce latency by routing traffic to healthy, less-busy servers.

---

# Key Takeaways

* **Latency** measures **how fast** a request is completed; **Throughput** measures **how much work** is completed over time.
* Low latency is crucial for interactive, real-time systems, while high throughput is essential for batch and large-scale processing.
* Improving one metric may negatively impact the other, making performance optimization a balancing act.
* Queueing, concurrency, caching, load balancing, and parallelism all influence both latency and throughput.
* Monitor both average and percentile latencies (P95/P99) alongside throughput to gain a complete picture of system performance.
* The best system design aligns performance optimization with business priorities rather than maximizing a single metric.
