# Availability

Availability is the ability of a system to remain operational and accessible to users, even in the presence of failures. A highly available system minimizes downtime and ensures that users can continue accessing services without interruption.

Availability is typically measured as a percentage of uptime over a given period. For example, an availability of **99.99%** means the system is operational for 99.99% of the time.

---

## Why and When Should You Use It?

System downtime can lead to revenue loss, poor user experience, and reduced customer trust. High availability ensures that services remain accessible despite hardware failures, software bugs, network outages, or maintenance activities.

High availability should be prioritized when building systems that:

* Serve a large number of users
* Require 24/7 operation
* Process financial transactions
* Provide healthcare or emergency services
* Support critical business operations
* Have strict Service Level Agreements (SLAs)

Examples include banking systems, e-commerce platforms, cloud services, streaming platforms, and messaging applications.

---

# Core Concepts

## Uptime

Uptime represents the amount of time a system remains operational.

Example:

* 99% Availability
* 99.9% Availability
* 99.99% Availability
* 99.999% Availability (Five Nines)

Higher availability generally requires greater architectural complexity and cost.

---

## Downtime

Downtime is the period during which a service is unavailable.

Approximate yearly downtime:

| Availability | Maximum Downtime/Year |
| ------------ | --------------------: |
| 99%          |             3.65 days |
| 99.9%        |            8.76 hours |
| 99.99%       |          52.6 minutes |
| 99.999%      |          5.26 minutes |

---

## Redundancy

Redundancy involves maintaining duplicate components so that if one component fails, another can immediately take over.

Examples:

* Multiple application servers
* Multiple databases
* Backup network connections
* Redundant storage

---

## Failover

Failover is the automatic process of switching from a failed component to a healthy backup.

Types:

* Active-Passive Failover
* Active-Active Failover

---

## Elimination of Single Point of Failure (SPOF)

A Single Point of Failure is any component whose failure can bring down the entire system.

Examples:

* Single database server
* Single load balancer
* Single application server

Highly available systems eliminate SPOFs by introducing redundancy.

---

## Health Checks

Health checks continuously monitor the status of services.

If a service becomes unhealthy, traffic is redirected to healthy instances automatically.

Health checks can be:

* Liveness Checks
* Readiness Checks

---

## Replication

Replication creates multiple copies of data across different servers or regions.

Benefits:

* Higher availability
* Disaster recovery
* Read scalability

---

# Architecture

A simple highly available architecture looks like:

```text
                     Users
                       │
                   DNS / CDN
                       │
                Load Balancer
                ┌──────┴──────┐
                │             │
          App Server 1    App Server 2
                │             │
                └──────┬──────┘
                       │
               Redis Cache Cluster
                       │
                Primary Database
                │             │
           Read Replica   Read Replica
```

---

## Pros

* High system uptime
* Better user experience
* Fault tolerance
* Reduced business impact during failures
* Supports maintenance with minimal downtime
* Improved customer trust

## Cons

* Increased infrastructure cost
* More complex architecture
* Data synchronization challenges
* Additional monitoring requirements
* More operational overhead

---

# Comparison

## Availability vs Reliability

| Feature     | Availability                  | Reliability                                   |
| ----------- | ----------------------------- | --------------------------------------------- |
| Definition  | Ability to remain operational | Ability to perform correctly without failures |
| Focus       | Minimizing downtime           | Preventing failures                           |
| Goal        | Keep the service accessible   | Ensure consistent and correct operation       |
| Measured By | Uptime percentage             | Mean Time Between Failures (MTBF)             |

---

## Active-Active vs Active-Passive Failover

| Feature              | Active-Active             | Active-Passive               |
| -------------------- | ------------------------- | ---------------------------- |
| Active Servers       | Multiple                  | One                          |
| Traffic Handling     | Shared across all servers | Only primary handles traffic |
| Failover Time        | Near instant              | Slight delay                 |
| Resource Utilization | High                      | Lower                        |
| Complexity           | Higher                    | Lower                        |

---

# Real-world Examples

### Netflix

* Deploys services across multiple Availability Zones and regions.
* Automatically redirects traffic if an instance or region becomes unavailable.

### Amazon

* Uses multiple Availability Zones for EC2, RDS, and other cloud services to minimize downtime.

### Google Search

* Replicates search indexes across data centers worldwide to ensure uninterrupted search availability.

### WhatsApp

* Uses replicated backend infrastructure to keep messaging services available even during server failures.

---

# Best Practices

* Eliminate all Single Points of Failure (SPOFs)
* Deploy multiple application instances
* Use load balancers with health checks
* Replicate databases across multiple nodes
* Perform automatic failover whenever possible
* Use distributed caches instead of local caches
* Monitor system health continuously
* Perform regular backup and disaster recovery testing
* Deploy services across multiple Availability Zones or regions
* Design applications to be stateless

---

# Common Mistakes

* Relying on a single database server
* Assuming backups alone provide high availability
* Ignoring health checks
* Not testing failover mechanisms
* Keeping application state on individual servers
* Deploying all services in a single Availability Zone
* Neglecting monitoring and alerting
* Failing to eliminate Single Points of Failure

---

# Interview Questions

#### 1. What does availability mean?

**Answer:**
Availability is the ability of a system to remain operational and accessible to users, even when failures occur. It is commonly measured as the percentage of uptime over a period of time.

---

#### 2. What is the difference between availability and reliability?

**Answer:**
Availability measures whether a system is accessible, whereas reliability measures whether it performs correctly without failures. A system can be highly available but still unreliable if it frequently returns incorrect results.

---

#### 3. What is a Single Point of Failure (SPOF)?

**Answer:**
A Single Point of Failure is a component whose failure causes the entire system to become unavailable. Examples include a single database server or a single application server without redundancy.

---

#### 4. How do you improve system availability?

**Answer:**
Common techniques include:

* Redundancy
* Load balancing
* Database replication
* Automatic failover
* Health checks
* Multi-region deployment
* Distributed caching
* Stateless application design

---

#### 5. What is the difference between Active-Active and Active-Passive architectures?

**Answer:**
In an Active-Active architecture, multiple servers simultaneously handle user traffic, improving both availability and scalability. In an Active-Passive architecture, only the primary server handles traffic while the standby server remains idle until a failure occurs.

---

# Key Takeaways

* Availability ensures a system remains accessible despite hardware, software, or network failures by eliminating single points of failure and introducing redundancy.
* High availability is achieved through techniques such as load balancing, replication, health checks, failover mechanisms, and multi-zone or multi-region deployments.
* Improving availability increases system resilience and user satisfaction but also introduces additional architectural complexity and operational cost.
