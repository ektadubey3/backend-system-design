# Fault Tolerance

Fault Tolerance is the ability of a system to continue operating correctly even when one or more of its components fail. Instead of allowing a single failure to bring down the entire application, a fault-tolerant system detects failures, isolates them, and recovers automatically while maintaining service availability.

It is one of the core principles of distributed system design and is closely related to **High Availability (HA)**, **Reliability**, and **Resilience**.

### Why and When Should You Use It?

Without fault tolerance:

```
Server Failure
      │
      ▼
Application Down ❌
```

With fault tolerance:

```
Server Failure
      │
      ▼
Traffic automatically shifts
      │
      ▼
Application Continues Running ✅
```

Use Fault Tolerance when:

* Building mission-critical applications (banking, healthcare, aviation)
* High availability is required (99.9%+ uptime)
* System downtime results in financial loss
* Services are distributed across multiple servers or regions
* Hardware failures are expected
* Network failures are common
* Cloud-native or microservices architecture is used
* Applications process continuous user traffic

Examples:

* Online banking
* Payment gateways
* Netflix
* Amazon
* Google Search
* Kubernetes clusters


---

# Core Concepts

## 1. Failure

Anything that prevents a component from functioning correctly.

Examples:

* Server crash
* Disk failure
* Database outage
* Network partition
* Power outage
* Software bug

---

## 2. Redundancy

Having duplicate components so another one can take over.

Examples:

* Multiple servers
* Multiple databases
* Multiple network paths

Example:

```
        Client
           │
    Load Balancer
    ┌──────┴────┐
Server A      Server B
        
```

If Server A fails:

```
Traffic → Server B
```

---

## 3. Failover

Automatic switching to a backup component.

Example:

```
Primary Database
       │
       ▼
    Failure
       │
       ▼
Secondary Database becomes Primary
```

---

## 4. Replication

Copying data across multiple machines.

Types:

### Synchronous Replication

```
Write
 │
 ├──DB1
 └──DB2
```

✅ No data loss
❌ Higher latency

### Asynchronous Replication

```
Write
 │
 ▼
DB1
 │
 ▼ later
DB2
```

✅ Faster
❌ Possible data loss

---

## 5. Health Checks

The system continuously verifies whether components are healthy.

Example:

```
Every 10 sec

Ping Server

Healthy?
  Yes → Continue
  No → Remove from Load Balancer
```

---

## 6. Circuit Breaker

Stops repeatedly calling a failing service.

Without Circuit Breaker:

```
     App
      │
      ▼
Failed Service
      │
    Retry
      │
    Retry
      │
    Retry
```

With Circuit Breaker:

```
     App
      │
      ▼
  Circuit Open
      │
Return fallback
```

Popular libraries:

* Hystrix (legacy)
* Resilience4j
* Polly (.NET)

---

## 7. Retry Mechanism

Temporary failures often resolve automatically.

Example:

```
Attempt 1 ❌

Wait 1 sec

Attempt 2 ❌

Wait 2 sec

Attempt 3 ✅
```

Usually combined with exponential backoff.

---

## 8. Graceful Degradation

Instead of complete failure, reduce functionality.

Example:

Netflix

Recommendations unavailable

↓

Streaming still works

---

## 9. Bulkhead Pattern

Prevent one failing component from affecting others.

Example:

```
Service

├── Payment Pool
├── Notification Pool
└── Search Pool
```

Payment overload should not crash Search.

---

## 10. Timeout

Never wait forever.

Example:

```
Call Service

Timeout = 2 sec

No response

↓

Abort Request
```

---

# Architecture

Example of a Fault-Tolerant Architecture:

```
                    Users
                      │
                DNS / CDN
                      │
             Load Balancer (HA)
             ┌────────┴─────────┐
             │                  │
       App Server 1        App Server 2
             │                  │
             └────────┬─────────┘
                      │
                Message Queue
                      │
          ┌───────────┴───────────┐
          │                       │
     Worker 1                Worker 2
          │                       │
          └───────────┬───────────┘
                      │
               Primary Database
                      │
             Replication (Sync/Async)
                      │
              Replica / Standby DB
                      │
               Backup & Recovery
```

Failure scenarios:

* App Server fails → Load balancer redirects traffic.
* Worker crashes → Queue retries the task.
* Database fails → Replica becomes primary.
* Entire availability zone fails → Traffic shifts to another zone.

---

# Characteristics

| Pros                               | Cons                               |
| ---------------------------------- | ---------------------------------- |
| High availability                  | Higher infrastructure cost         |
| Improved reliability               | Increased architectural complexity |
| Better customer experience         | More operational overhead          |
| Automatic recovery                 | Data consistency challenges        |
| Reduced downtime                   | Harder debugging                   |
| Supports zero-downtime deployments | Additional monitoring required     |
| Better scalability                 | Requires redundancy                |

---

# Comparison

| Feature          | Reliability                      | Availability                   | Fault Tolerance                                   |
| ---------------- | -------------------------------- | ------------------------------ | ------------------------------------------------- |
| **Focus**        | Correct and consistent operation | Continuous accessibility       | Continue operating during failures                |
| **Goal**         | Minimize failures                | Minimize downtime              | Handle component failures gracefully              |
| **Measured By**  | MTBF, MTTR, Failure Rate         | Uptime Percentage              | Recovery Time, Failover Success                   |
| **Example**      | Payment processed exactly once   | Payment service remains online | Backup server takes over after failure            |
| **Relationship** | Quality attribute                | Quality attribute              | Technique to improve reliability and availability |

---

# Real-world Examples

## Netflix

* Multi-region deployment
* Chaos Engineering
* Automatic failover
* Circuit breakers
* Graceful degradation

---

## Amazon

* Multiple Availability Zones
* Auto Scaling
* Load balancing
* Database replication
* Redundant networking

---

## Google Search

* Thousands of replicated servers
* Global load balancing
* Automatic node replacement
* Distributed storage
* Data replication

---

## Kubernetes

* Restarts failed pods
* ReplicaSets
* Self-healing
* Node replacement
* Health probes

---

## Banking Systems

* Active-active databases
* Transaction replication
* Backup data centers
* Automatic failover
* Zero or near-zero downtime

---

# Best Practices

* Design for failure from the beginning.
* Eliminate single points of failure (SPOFs).
* Use redundant infrastructure across Availability Zones or regions.
* Replicate critical data with an appropriate consistency model.
* Implement health checks and automatic failover.
* Use load balancers to distribute traffic.
* Apply circuit breakers to isolate failing dependencies.
* Configure retries with exponential backoff and jitter.
* Set sensible timeout values for all remote calls.
* Make operations idempotent where retries are possible.
* Use message queues for asynchronous processing.
* Monitor system health with metrics, logs, and alerts.
* Regularly test failover using chaos engineering or game days.
* Maintain backups and periodically verify restore procedures.

---

# Common Mistakes

* Having a single database with no replica.
* Relying on one server (Single Point of Failure).
* Infinite retries without backoff.
* Missing timeout configurations.
* Ignoring health checks.
* Treating backups as a replacement for fault tolerance.
* Not testing failover scenarios.
* Using synchronous replication everywhere, increasing latency unnecessarily.
* Assuming cloud infrastructure is automatically fault tolerant without proper design.
* Failing to monitor replicas and standby systems.

---

# Interview Questions

### 1. What is Fault Tolerance?

**Answer:**
Fault Tolerance is the capability of a system to continue functioning correctly even when one or more components fail by using redundancy, failover, replication, and recovery mechanisms.

---

### 2. How is Fault Tolerance different from High Availability?

**Answer:**
Fault Tolerance aims for uninterrupted operation despite failures, while High Availability focuses on minimizing downtime, often allowing brief service interruptions during failover.

---

### 3. What is a Single Point of Failure (SPOF)?

**Answer:**
A SPOF is any component whose failure causes the entire system to stop working. Fault-tolerant systems eliminate or minimize SPOFs through redundancy.

---

### 4. Why is replication important?

**Answer:**
Replication keeps multiple copies of data or services, allowing another instance to take over if the primary fails, improving availability and reducing the risk of data loss.

---

### 5. What is failover?

**Answer:**
Failover is the automatic transfer of operations from a failed component to a healthy backup with minimal disruption.

---

### 6. Explain active-active vs active-passive.

**Answer:**

* **Active-Active:** Multiple instances handle traffic simultaneously. If one fails, others continue serving requests.
* **Active-Passive:** One instance is active while another remains on standby and takes over only after a failure.

---

### 7. What is graceful degradation?

**Answer:**
Graceful degradation allows a system to continue providing core functionality while temporarily disabling non-essential features during failures.

---

### 8. Why are retries alone not enough?

**Answer:**
Retries can worsen overload if a dependency is already struggling. They should be combined with timeouts, exponential backoff, jitter, and circuit breakers.

---

### 9. What role does a load balancer play?

**Answer:**
A load balancer distributes requests across healthy instances, performs health checks, and removes unhealthy nodes from service, improving both scalability and fault tolerance.

---

### 10. How do you test fault tolerance?

**Answer:**
By intentionally introducing failures (e.g., shutting down servers, simulating network partitions, or injecting latency) and verifying that the system continues to operate correctly. Chaos engineering tools and controlled failover drills are commonly used.

---

# Key Takeaways

* Fault Tolerance enables systems to continue operating despite component failures.
* Redundancy is the foundation of fault-tolerant design.
* Failover, replication, retries, timeouts, circuit breakers, and health checks work together to improve resilience.
* Eliminating Single Points of Failure is essential for highly available systems.
* Fault Tolerance is a design strategy that complements High Availability, Reliability, and Disaster Recovery.
* Testing failure scenarios regularly is just as important as implementing recovery mechanisms.
* Modern cloud-native platforms such as Kubernetes and major cloud providers include many fault-tolerance building blocks, but applications must still be designed to use them effectively.
