# High-Level Design (HLD)

High-Level Design (HLD) is the process of defining the **overall architecture** of a software system. It focuses on **what components exist, how they communicate, and how the system satisfies functional and non-functional requirements**, without diving into implementation details.

Think of HLD as the **blueprint of a building**—it shows rooms, floors, and connections, but not the electrical wiring inside the walls.

## Goals of HLD

* Define system architecture
* Identify major components/services
* Describe communication between components
* Address scalability, reliability, availability, and security
* Estimate infrastructure requirements
* Enable multiple teams to work independently

## What HLD Includes

### 1. Functional Requirements

What the system should do.

Example:

* User registration
* Login
* Upload files
* Search products

### 2. Non-Functional Requirements

How well the system should perform.

Examples:

* Scalability
* Availability
* Reliability
* Performance
* Latency
* Throughput
* Security
* Fault tolerance
* Maintainability

### 3. System Architecture

Examples:

* Monolith
* Microservices
* Event-driven
* Serverless
* Layered architecture

### 4. Components

Examples:

* API Gateway
* Load Balancer
* Authentication Service
* User Service
* Notification Service
* Cache
* Database
* Message Queue

### 5. Database Design

At HLD level:

* SQL vs NoSQL
* Read replicas
* Sharding
* Partitioning
* Replication

No table-level schema yet.

### 6. API Design

Examples:

```
POST /users

GET /orders/{id}

PUT /cart

DELETE /wishlist
```

Only endpoints—not implementation.

### 7. Data Flow

Example:

```
        Client
          ↓
      API Gateway
          ↓
    Authentication
          ↓
    Order Service
          ↓
      Database
          ↓
      Response
```

### 8. Scaling Strategy

Examples:

* Horizontal Scaling
* Vertical Scaling
* Auto Scaling
* Load Balancing

### 9. Fault Tolerance

Examples:

* Retries
* Circuit Breaker
* Replication
* Failover
* Backup

### 10. Security

Examples:

* JWT
* OAuth
* HTTPS
* Encryption
* RBAC
* API Rate Limiting

---

## Typical HLD Diagram

```text
                    Client
                      |
                Load Balancer
                      |
                  API Gateway
        ______________|_______________
       |              |               |
    User            Order           Search
   Service         Service          Service
       |              |               |
      Redis         Kafka           Redis
       |              |               |
     MySQL        MongoDB         Elasticsearch
```

---

# HLD vs LLD

| Feature / Aspect | High-Level Design (HLD) | Low-Level Design (LLD)  |
| ---------- | ----------------------- | ----------------------- |
| Focus      | Architecture            | Implementation          |
| Level      | High                    | Detailed                |
| Audience   | Architects, Tech Leads  | Developers              |
| Components | Services                | Classes, Methods        |
| Database   | SQL vs NoSQL            | Tables, Columns         |
| APIs       | Endpoints               | Request/Response Models |
| Diagrams   | Architecture            | Class, Sequence, ER     |
| Scale      | Entire System           | Individual Module       |
| Code       | No                      | Yes/Pseudocode          |
| Goal       | System Blueprint        | Coding Blueprint        |
| Scope     | Entire system         | Individual components            |
| Decisions | Architecture          | Design patterns, classes         |
| Detail    | Abstract              | Detailed                         |
| Output    | Architecture diagrams | UML, class diagrams, code design |
| Changes   | Less frequent         | More frequent during development |

---

# Real-world Examples

### 1. URL Shortener (TinyURL/Bitly)

Components:

```
      Client
        ↓
  Load Balancer
        ↓
    API Server
        ↓
    URL Service
        ↓
      Cache
        ↓
     Database
```

Features:

* Generate short URL
* Redirect
* Analytics
* Cache popular URLs

### 2. WhatsApp

Components:

* Chat Service
* Presence Service
* Notification Service
* Media Service
* Message Queue
* Distributed Storage
* Load Balancers

Focus:

* Low latency
* High availability
* Billions of messages

### 3. Netflix

Components:

* API Gateway
* Recommendation Service
* Streaming Service
* User Service
* Billing
* CDN
* Cache

Design Goals:

* Global scalability
* Fault tolerance
* Low latency streaming

### 4. Uber

Components:

* Rider Service
* Driver Service
* Matching Service
* Pricing Service
* Maps
* Payment Service
* Notification Service

Challenges:

* Real-time location updates
* Surge pricing
* High availability

### 5. Amazon

Components:

* Product Catalog
* Inventory
* Search
* Cart
* Payment
* Recommendation
* Order Service

Uses:

* Caching
* Microservices
* Event-driven architecture
* Distributed databases

### 6. YouTube

Components:

* Upload Service
* Encoding Service
* Metadata Service
* CDN
* Recommendation Engine
* Search Service

Challenges:

* Video processing
* Large storage
* Global delivery

---

# Best Practices

### Understand Requirements First

Never start designing before clarifying:

* Functional requirements
* Non-functional requirements
* Constraints
* Scale estimates

### Estimate Scale

Know:

* Daily Active Users
* Requests per second
* Storage
* Bandwidth
* Read/write ratio

### Keep Components Loosely Coupled

Prefer:

```
      API Gateway
          ↓
    Microservices
          ↓
Independent Databases
```

instead of tightly coupled services.

### Design for Failure

Assume:

* Server crashes
* Database failures
* Network issues
* Service outages

Use:

* Retries
* Timeouts
* Circuit breakers
* Failover
* Replication

### Use Caching Wisely

Cache:

* Frequently read data
* Sessions
* Product details
* Search results

Avoid caching highly dynamic or sensitive data without proper invalidation.

### Prefer Stateless Services

Benefits:

* Easier scaling
* Simpler deployments
* Better fault tolerance

### Use Asynchronous Processing

Good candidates:

* Emails
* Notifications
* Image processing
* Video encoding
* Report generation

### Secure Every Layer

Include:

* Authentication
* Authorization
* Encryption
* Rate limiting
* Input validation
* Audit logging

### Monitor Everything

Track:

* Latency
* CPU
* Memory
* Error rates
* Throughput
* Availability
* Business metrics

---

# Common Mistakes

#### 1. Ignoring Non-Functional Requirements

Designing only for features and ignoring scalability, availability, or security.

#### 2. Overengineering

Adding Kafka, Kubernetes, and dozens of microservices for a small application.

#### 3. Choosing the Wrong Database

Using a relational database for highly flexible documents or a NoSQL database where complex transactions are essential.

#### 4. No Caching Strategy

Every request hitting the database leads to poor performance.

#### 5. Single Point of Failure (SPOF)

Examples:

* One database instance
* One server
* One cache node

Always plan redundancy.

#### 6. Synchronous Calls Everywhere

Long dependency chains increase latency and failure propagation.

#### 7. Ignoring Observability

Without logs, metrics, and traces, diagnosing production issues becomes difficult.

#### 8. Not Planning for Growth

Designing only for current traffic instead of expected future scale.

#### 9. Missing Security Considerations

Examples:

* No authentication
* Plain-text passwords
* Unencrypted communication
* Lack of rate limiting

#### 10. Skipping Capacity Estimation

Without estimating users, traffic, and storage, infrastructure choices are often inaccurate.

---

# Key Takeaways

* High-Level Design defines the **overall architecture** of a system rather than implementation details.
* Start with **functional and non-functional requirements** before selecting technologies or patterns.
* Break the system into **independent, loosely coupled components** with clear responsibilities.
* Consider core architectural concerns early: **scalability, availability, reliability, security, and maintainability**.
* Select storage, caching, and communication mechanisms based on workload characteristics (read/write ratio, consistency, latency).
* Eliminate **single points of failure** using redundancy, replication, and failover strategies.
* Use **asynchronous messaging** for long-running or non-critical workflows to improve responsiveness and resilience.
* Build in **observability** (logging, metrics, tracing, alerting) from the beginning.
* Avoid overengineering—choose an architecture that fits the current requirements while leaving room for future growth.
* A strong HLD communicates the system clearly enough that teams can proceed to detailed design (LLD) and implementation with confidence.
