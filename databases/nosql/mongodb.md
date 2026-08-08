# MongoDB

MongoDB is easy to start with, but designing a backend that continues to perform at **millions of documents, high request volumes, and distributed workloads** requires understanding how the database behaves under the hood.

This document focuses on the **system design concepts that matter in real production systems and technical interviews**.

---

## Core Concepts

### 1. Document Model

MongoDB stores data as **BSON documents** instead of rows and columns.

```json
{
  "_id": "user_101",
  "name": "Alex",
  "email": "alex@example.com",
  "address": {
    "city": "Bangalore",
    "country": "India"
  }
}
```

Documents can contain:

* Nested objects
* Arrays
* Flexible fields
* References to other documents

The biggest mindset shift when moving from relational databases is:

> Model data around **application access patterns**, not only around entities.

---

### 2. Embedding vs Referencing

MongoDB relationships are generally modeled using either **embedded documents** or **references**.

#### Embedding

```json
{
  "_id": "order_1001",
  "customer": {
    "name": "Alex",
    "email": "alex@example.com"
  },
  "items": [
    {
      "product": "Keyboard",
      "quantity": 1
    }
  ]
}
```

Best when:

* Data is frequently read together.
* Child data belongs to one parent.
* The embedded collection will remain reasonably small.

#### Referencing

```json
{
  "_id": "order_1001",
  "userId": "user_101",
  "productIds": ["product_10", "product_20"]
}
```

Best when:

* Data changes independently.
* Multiple documents share the same entity.
* Embedded documents could grow without limits.

---

### 3. Indexing

Indexes reduce the amount of data MongoDB must scan.

Without an index:

```text
Request
   |
   v
Scan thousands/millions of documents
   |
   v
Return result
```

With an index:

```text
Request
   |
   v
Index Lookup
   |
   v
Matching Documents
```

Example:

```javascript
db.users.createIndex({ email: 1 })
```

Compound index:

```javascript
db.orders.createIndex({
  userId: 1,
  createdAt: -1
})
```

The **order of fields matters** in compound indexes.

A useful rule:

> Create indexes based on your most important query patterns.

---

### 4. Replication

MongoDB uses **Replica Sets** for high availability.

```text
                 ┌─────────────┐
                 │   Primary   │
                 │   MongoDB   │
                 └──────┬──────┘
                        │
             Replication│
              ┌─────────┴─────────┐
              │                   │
              v                   v
      ┌──────────────┐    ┌──────────────┐
      │ Secondary 1  │    │ Secondary 2  │
      └──────────────┘    └──────────────┘
```

The **Primary** usually handles writes.

Secondaries continuously replicate data from the primary.

If the primary becomes unavailable, an eligible secondary can be elected as the new primary.

Replication provides:

* High availability
* Automatic failover
* Data redundancy
* Read scaling for suitable workloads

---

### 5. Sharding

Replication helps with **availability**.

Sharding helps with **horizontal scalability**.

Instead of keeping all data on one machine:

```text
                Application
                     |
                     v
                  mongos
                     |
        ┌────────────┼────────────┐
        │            │            │
        v            v            v
     Shard A      Shard B      Shard C
```

Each shard owns a portion of the dataset.

For example:

```text
Shard A → users 1 - 1M
Shard B → users 1M - 2M
Shard C → users 2M - 3M
```

The field used to distribute data is called the **Shard Key**.

Choosing the wrong shard key can create:

* Hot shards
* Uneven data distribution
* Slow queries
* Difficult scaling

A good shard key typically provides:

* High cardinality
* Good distribution
* Query relevance
* Low risk of write hotspots

---

### 6. Transactions

MongoDB supports multi-document ACID transactions.

```javascript
const session = client.startSession();

session.startTransaction();

try {
  await accounts.updateOne(
    { _id: senderId },
    { $inc: { balance: -100 } },
    { session }
  );

  await accounts.updateOne(
    { _id: receiverId },
    { $inc: { balance: 100 } },
    { session }
  );

  await session.commitTransaction();
} catch (error) {
  await session.abortTransaction();
}
```

Transactions are useful when multiple writes **must succeed or fail together**.

However, they should not automatically become the default design.

Good document modeling can often reduce the need for distributed transactions.

---

### 7. Consistency

Distributed systems frequently involve trade-offs between:

* Consistency
* Availability
* Latency
* Durability

MongoDB provides controls such as:

```text
Write Concern
Read Concern
Read Preference
```

For example, requiring acknowledgment from multiple replica-set members increases durability but may also increase write latency.

---

## Architecture

A common production MongoDB backend could look like this:

```text
                           Users
                             |
                             v
                     ┌───────────────┐
                     │ Load Balancer │
                     └───────┬───────┘
                             |
              ┌──────────────┼──────────────┐
              │              │              │
              v              v              v
        ┌──────────┐   ┌──────────┐   ┌──────────┐
        │ API Node │   │ API Node │   │ API Node │
        └────┬─────┘   └────┬─────┘   └────┬─────┘
             │              │              │
             └──────────────┼──────────────┘
                            |
              ┌─────────────┼─────────────┐
              │                           │
              v                           v
        ┌─────────────┐             ┌─────────────┐
        │    Redis    │             │   Message   │
        │    Cache    │             │    Queue    │
        └─────────────┘             └──────┬──────┘
                                          |
                                          v
                                    ┌─────────────┐
                                    │   Workers   │
                                    └──────┬──────┘
                                          
                     ┌─────────────────────────────┐
                     │       MongoDB Cluster       │
                     │                             │
                     │  Primary + Replica Members  │
                     │          or                 │
                     │       Sharded Cluster       │
                     └─────────────────────────────┘
```

### Request Flow

A typical read request:

```text
Client
  ↓
Load Balancer
  ↓
Backend API
  ↓
Check Redis Cache
  ↓
Cache Hit? ── Yes ──> Return response
  |
  No
  ↓
Query MongoDB
  ↓
Store result in cache
  ↓
Return response
```

A write-heavy asynchronous workflow:

```text
Client
  ↓
API
  ↓
MongoDB
  ↓
Message Queue
  ↓
Background Workers
  ↓
Email / Analytics / Notifications / Search Index
```

The goal is to keep the synchronous request path **small, predictable, and fast**.

---

## MongoDB vs PostgreSQL

Neither database is universally better.

The correct choice depends on the system's access patterns and consistency requirements.

| Area               | MongoDB                                         | PostgreSQL                               |
| ------------------ | ----------------------------------------------- | ---------------------------------------- |
| Data model         | Document-oriented                               | Relational                               |
| Schema             | Flexible                                        | Strongly structured                      |
| Nested data        | Natural fit                                     | Often normalized into tables             |
| Joins              | Supported but not the primary modeling strategy | Excellent                                |
| Horizontal scaling | Designed with sharding support                  | Possible, but architecture-dependent     |
| Transactions       | Supported                                       | Extremely mature                         |
| Best fit           | Flexible document-heavy applications            | Relational and transaction-heavy systems |

### Choose MongoDB when

Your application has:

* Document-shaped data
* Rapidly evolving schemas
* Large distributed datasets
* High horizontal-scaling requirements
* Read patterns that benefit from denormalization

### Choose PostgreSQL when

Your application depends heavily on:

* Complex relationships
* Joins
* Strong relational constraints
* Financial-style transactional workflows
* Complex SQL analytics

The database should follow the **workload**, not the trend.

---

# Real-World Example: E-Commerce Backend

Imagine an e-commerce platform with:

```text
10M users
2M products
100M orders
Thousands of requests per second
```

A simplified architecture:

```text
                         Customers
                             |
                             v
                       API Gateway
                             |
                ┌────────────┼────────────┐
                │            │            │
                v            v            v
             User API    Product API   Order API
                │            │            │
                └────────────┼────────────┘
                             |
                 ┌───────────┼───────────┐
                 │                       │
                 v                       v
              Redis                  MongoDB
             Product              Replica Set /
              Cache               Sharded Cluster
                                         |
                                         v
                                  Order Events
                                         |
                                         v
                                   Message Queue
                                         |
                        ┌────────────────┼───────────────┐
                        v                v               v
                    Inventory        Email Worker   Analytics
                     Worker
```

### Product Document

```json
{
  "_id": "product_1201",
  "name": "Mechanical Keyboard",
  "category": "electronics",
  "price": 5999,
  "inventory": 230,
  "rating": 4.7
}
```

Useful indexes might include:

```javascript
db.products.createIndex({ category: 1, price: 1 })

db.products.createIndex({ name: "text" })
```

---

### Order Document

An order may intentionally duplicate some product information:

```json
{
  "_id": "order_9001",
  "userId": "user_101",
  "status": "PAID",
  "items": [
    {
      "productId": "product_1201",
      "name": "Mechanical Keyboard",
      "priceAtPurchase": 5999,
      "quantity": 1
    }
  ],
  "total": 5999,
  "createdAt": "2026-08-01T10:00:00Z"
}
```

Why duplicate `name` and `priceAtPurchase`?

Because historical orders should not change when the current product name or price changes.

This is a good example of **intentional denormalization**.

---

## Best Practices

### Design Schema From Queries

Before designing collections, identify your major access patterns.

For example:

```text
Get user by email
Get latest 20 orders for a user
Find products by category and price
Get an order by orderId
```

Then model documents and indexes around these queries.

---

### Use `explain()` Before Guessing

Inspect query execution:

```javascript
db.orders
  .find({ userId: "user_101" })
  .sort({ createdAt: -1 })
  .explain("executionStats")
```

Watch for excessive document scanning.

A healthy query ideally examines close to the number of documents it returns.

---

### Keep Documents Bounded

Avoid arrays that can grow forever.

Bad:

```json
{
  "userId": "user_101",
  "activityHistory": [
    "... potentially millions of entries ..."
  ]
}
```

Better:

```text
users
user_activity
```

Keep frequently growing events in their own collection.

---

### Avoid Unnecessary Database Calls

Bad:

```text
Get Order
   ↓
Get User
   ↓
Get Product 1
   ↓
Get Product 2
   ↓
Get Product 3
```

This creates unnecessary network round trips.

Consider:

* Embedding
* Batch queries
* Aggregations
* Caching
* Intentional denormalization

---

### Use Pagination Correctly

Offset pagination:

```javascript
.find({})
.skip(100000)
.limit(20)
```

can become expensive for large offsets.

For large datasets, cursor-based pagination is often better:

```javascript
db.orders.find({
  _id: {
    $gt: lastSeenId
  }
}).limit(20)
```

---

### Index Selectively

Indexes improve reads but are not free.

Every additional index increases:

```text
Storage
+
Memory usage
+
Write cost
+
Index maintenance
```

Create indexes for real query patterns rather than indexing every field.

---

### Monitor Production Metrics

Track metrics such as:

```text
Query latency
Connections
CPU
Memory
Disk I/O
Replication lag
Cache hit ratio
Slow queries
Documents examined vs returned
Shard distribution
```

System design does not end when the architecture diagram is finished.

Production behavior matters more than the diagram.

---

## Common Mistakes

### 1. Treating MongoDB Like a Relational Database

Creating many tiny collections and manually joining everything often removes the benefits of the document model.

Design around **how the application reads and writes data**.

---

### 2. Missing Indexes

A query may appear fast with:

```text
1,000 documents
```

but become painful at:

```text
100,000,000 documents
```

Test using realistic data volumes.

---

### 3. Too Many Indexes

Indexes accelerate reads but slow writes.

```text
Insert Document
      |
      ├── Update Index A
      ├── Update Index B
      ├── Update Index C
      └── Update Index D
```

Only maintain indexes that provide measurable value.

---

### 4. Unbounded Arrays

Large growing arrays create oversized documents and increasingly expensive updates.

Use separate collections for unbounded relationships such as:

* Logs
* Comments
* Notifications
* Transactions
* Activity events

---

### 5. Choosing a Bad Shard Key

For example, a monotonically increasing value can send a disproportionate number of writes toward one area of the cluster.

Shard-key selection should be treated as an **architectural decision**, not a configuration detail.

---

### 6. Using Transactions Everywhere

Transactions can simplify correctness, but excessive cross-document transactions increase complexity and cost.

First ask:

> Can the schema make the operation atomic within a single document?

MongoDB guarantees atomicity for operations on a single document.

---

### 7. Scaling Before Measuring

Do not immediately add:

```text
Sharding
+
Redis
+
Kafka
+
Multiple services
```

because a system *might* become large.

Start with the simplest architecture that satisfies the requirements.

Measure the bottleneck.

Then scale the bottleneck.

---

# Interview Questions

### 1. What is the difference between replication and sharding?

**Answer:** Replication creates copies of the same data for high availability and redundancy. Sharding distributes different portions of the dataset across multiple servers for horizontal scalability.

---

### 2. When should you embed documents instead of referencing them?

**Answer:** Embed when related data is usually accessed together, has a clear ownership relationship, and will remain bounded in size.

---

### 3. Why are indexes important in MongoDB?

**Answer:** Indexes allow MongoDB to locate matching documents without scanning the entire collection, significantly reducing query latency at scale.

---

### 4. What makes a good shard key?

**Answer:** A good shard key usually has high cardinality, distributes reads and writes evenly, avoids hotspots, and aligns with important query patterns.

---

### 5. MongoDB is schemaless. Does that mean schema design is unnecessary?

**Answer:** No. MongoDB has a flexible schema, but careful schema design is still essential for performance, scalability, data consistency, and efficient queries.

---

# Key Takeaways

### 1. Design for access patterns

Do not ask only:

```text
"What entities exist?"
```

Ask:

```text
"What are my highest-volume reads and writes?"
```

Your schema should make those operations efficient.

---

### 2. Indexes solve many problems — but create new costs

Indexes can turn a collection scan into a fast lookup, but every index consumes storage, memory, and write capacity.

**Index intentionally.**

---

### 3. Scale based on bottlenecks

A sensible evolution often looks like:

```text
Single MongoDB Deployment
        ↓
Replica Set
        ↓
Caching + Async Processing
        ↓
Read Scaling
        ↓
Sharding when data/write volume demands it
```

Do not build a distributed system merely because distributed systems look impressive.

Build one because the workload requires it.

---

If this document helps you think more clearly about scalable backend architecture, **⭐ star the repo and follow for more system design content.**
