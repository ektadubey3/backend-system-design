# Functional vs Non-Functional Requirements

Every software system is built to solve a problem. Before designing the architecture, we must clearly understand **what the system should do** and **how well it should perform**. These expectations are captured as **Functional Requirements (FRs)** and **Non-Functional Requirements (NFRs)**.

Functional requirements define the features and business logic of the application, while non-functional requirements define the quality attributes that determine how reliable, scalable, secure, and efficient the system should be.

Understanding this distinction is one of the first and most important steps in system design.

## Why Are Requirements Important?

Well-defined requirements help engineers:

* Design the right architecture
* Estimate infrastructure needs
* Choose appropriate technologies
* Identify scalability challenges early
* Avoid costly redesigns
* Align engineering with business goals

Without clear requirements, even a technically sound architecture may fail to meet user expectations.

---

## Functional Requirements

### Definition

Functional requirements describe **what the system must do**. They specify the features, services, and behaviors that users or other systems expect.

They answer questions like:

* What actions can users perform?
* What business rules should the system follow?
* What APIs should exist?
* What data should be stored?


### Characteristics

* Feature-specific
* Business-driven
* User-focused
* Testable
* Usually documented as user stories or use cases


### Examples

#### Social Media Platform

* User can register.
* User can log in.
* User can create posts.
* User can like posts.
* User can comment on posts.
* User can follow other users.
* User can search profiles.


#### E-Commerce Website

* Browse products
* Add items to cart
* Place orders
* Make payments
* Track orders
* Cancel orders
* Write product reviews


#### Banking Application

* Transfer money
* Check account balance
* View transaction history
* Pay utility bills
* Generate statements

---

## Questions to Identify Functional Requirements

* What features are required?
* Who are the users?
* What actions can users perform?
* What business rules exist?
* What APIs are required?

---

## Non-Functional Requirements

### Definition

Non-functional requirements describe **how well the system should perform** rather than what it does.

They define quality attributes that influence architecture, infrastructure, and operational behavior.

### Characteristics

* Architecture-driven
* Performance-oriented
* Often measurable
* Affect the entire system
* Critical for large-scale applications

#### Common Non-Functional Requirements

#### Scalability

Ability to handle increasing users and traffic without significant performance degradation.

Example:

* Support 10 million active users
* Handle 100,000 requests per second

#### Availability

Percentage of time the system remains operational.

Examples:

* 99%
* 99.9%
* 99.99%
* 99.999% ("Five Nines")

#### Reliability

Ability to consistently produce correct results without failures.

Example:

A payment should never be processed twice.


#### Performance

Measures system responsiveness.

Examples:

* API response time < 200 ms
* Database query < 50 ms
* Page loads within 2 seconds


#### Latency

The time taken to process a request.

Lower latency improves user experience, especially for real-time systems.


#### Throughput

The number of requests processed per unit of time.

Example:

* 50,000 requests per second (RPS)

#### Security

Protects data and services from unauthorized access.

Examples:

* Authentication
* Authorization
* Encryption
* Secure APIs
* Data privacy

#### Fault Tolerance

Ability to continue operating even when components fail.

Examples:

* Database replication
* Automatic failover
* Multi-region deployment

#### Durability

Ensures that committed data is never lost, even during failures.

Example:

Once a payment succeeds, the transaction record must persist.

#### Maintainability

Ease of updating, debugging, and extending the system.

Good maintainability reduces development and operational costs.

#### Observability

Ability to understand system behavior using:

* Logs
* Metrics
* Traces
* Monitoring dashboards

#### Consistency

Ensures users receive accurate and synchronized data across distributed systems.

---

# Comparison

| Feature    | Functional Requirements     | Non-Functional Requirements                    |
| ---------- | --------------------------- | ---------------------------------------------- |
| Purpose    | Define what the system does | Define how the system performs                 |
| Focus      | Features                    | Quality attributes                             |
| Driven By  | Business needs              | System architecture                            |
| Examples   | Login, Search, Payment      | Scalability, Security, Availability            |
| Scope      | Specific functionality      | Entire application                             |
| Validation | Feature testing             | Performance, security, and reliability testing |

---

# Real-World Example: Instagram

## Functional Requirements

* User registration
* User login
* Upload photos
* Follow users
* Like posts
* Comment on posts
* Search users
* View feed
* Send direct messages
* Receive notifications


## Non-Functional Requirements

* Support millions of concurrent users
* Feed loads in under 300 ms
* Highly available (99.99% uptime)
* Secure user authentication
* Store billions of images reliably
* Scale horizontally during traffic spikes
* Deliver notifications with minimal delay

---

# How Requirements Influence Architecture

Requirements directly shape architectural decisions.

| Requirement         | Architectural Decision   |
| ------------------- | ------------------------ |
| Millions of users   | Horizontal scaling       |
| Low latency         | Caching (Redis), CDN     |
| High availability   | Replication and failover |
| Large file uploads  | Object storage           |
| High read traffic   | Read replicas            |
| Real-time messaging | WebSockets               |
| Event processing    | Kafka or RabbitMQ        |
| Fault tolerance     | Redundancy and retries   |
| Fast search         | Elasticsearch            |

---

# Best Practices

* Gather functional requirements before designing the system.
* Define measurable non-functional requirements.
* Prioritize requirements based on business impact.
* Identify trade-offs between competing requirements.
* Validate assumptions with stakeholders.
* Revisit requirements as the product evolves.

---

# Common Mistakes

* Ignoring scalability until production.
* Treating non-functional requirements as optional.
* Designing features without understanding business goals.
* Defining vague requirements such as "the system should be fast."
* Overengineering for unrealistic future scale.
* Failing to quantify performance expectations.

---

# Interview Questions

#### 1. What is the difference between functional and non-functional requirements?

**Answer:** Functional requirements define what the system should do (features), while non-functional requirements define how well the system should perform (quality attributes like scalability, security, and availability).

#### 2. How do non-functional requirements influence system architecture?

**Answer:** They determine architectural decisions such as caching, load balancing, database replication, sharding, monitoring, and security to meet performance and reliability goals.

#### 3. Give functional and non-functional requirements for a URL Shortener.

**Answer:**
**Functional:** Create short URLs, redirect to original URLs, custom aliases, analytics.
**Non-Functional:** High availability, low latency, scalability, fault tolerance, and secure access.

#### 4. Which non-functional requirements are most important for a payment system, and why?

**Answer:** Security, consistency, reliability, availability, and durability are critical to protect transactions, prevent data loss, and ensure correct payment processing.

#### 5. How do you gather and prioritize requirements before starting a system design?

**Answer:** First identify business goals and functional requirements, then define measurable non-functional requirements, prioritize them based on business impact, and design the architecture around those priorities.

---

## Key Takeaways

- Functional requirements define **what** a system should do.
- Non-functional requirements define **how well** the system should perform.
- Both are essential for designing scalable and reliable systems.
