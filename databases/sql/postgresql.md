# PostgreSQL

PostgreSQL is one of the most capable relational databases available for modern backend systems. It combines strong transactional guarantees with advanced indexing, JSON support, full-text search, geospatial extensions, replication, and powerful SQL features.

```text
Backend Application
        |
        v
Connection Pool
        |
        v
PostgreSQL
        |
        ├── Tables
        ├── Indexes
        ├── Transactions
        ├── Views
        ├── Replication
        └── Write-Ahead Log
```

PostgreSQL is commonly used for:

* E-commerce platforms
* Financial systems
* SaaS applications
* Content-management systems
* Analytics platforms
* Geospatial applications
* Authentication systems
* Inventory management
* Booking systems
* Event-driven architectures
* Microservices
* AI application metadata
* Multi-tenant platforms

---

# Why PostgreSQL?

PostgreSQL is a strong default database for backend systems because it provides:

* ACID transactions
* Strong consistency
* Rich SQL support
* Relational constraints
* Advanced indexes
* JSON and JSONB
* Full-text search
* Window functions
* Common Table Expressions
* Stored procedures
* Triggers
* Extensions
* Replication
* Partitioning
* Mature operational tooling

A typical backend flow looks like this:

```text
Client
   |
   v
API Gateway
   |
   v
Application Service
   |
   v
Connection Pool
   |
   v
PostgreSQL Primary
   |
   ├── Read Replica
   ├── Backup Storage
   └── Monitoring System
```

PostgreSQL is especially useful when:

* Data correctness is important.
* Relationships between entities matter.
* Multiple records must be updated atomically.
* Complex queries are required.
* The schema evolves over time.
* Strong constraints should protect data.
* Developers want to avoid implementing database guarantees in application code.

---

# Core Concepts

## 1. Database

A PostgreSQL server can host multiple databases.

```text
PostgreSQL Server
    |
    ├── users_database
    ├── orders_database
    └── analytics_database
```

Databases are logically isolated from one another.

A connection is established to a specific database.

```bash
psql -h localhost -p 5432 -U app_user -d ecommerce
```

---

## 2. Schema

A schema is a namespace inside a database.

```text
Database: ecommerce
    |
    ├── public
    ├── billing
    ├── inventory
    └── audit
```

Schemas help organize objects such as:

* Tables
* Views
* Functions
* Sequences
* Types

Example:

```sql
CREATE SCHEMA billing;

CREATE TABLE billing.invoices (
    id BIGSERIAL PRIMARY KEY,
    customer_id BIGINT NOT NULL,
    amount NUMERIC(12, 2) NOT NULL,
    status VARCHAR(30) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

Schemas can also provide access-control boundaries.

---

## 3. Tables

Tables store structured records.

```sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    email TEXT NOT NULL UNIQUE,
    full_name TEXT NOT NULL,
    status TEXT NOT NULL DEFAULT 'active',
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

A table consists of:

* Columns
* Rows
* Data types
* Constraints
* Indexes

```text
users

┌────┬──────────────────────┬─────────────┬──────────┐
│ id │ email                │ full_name   │ status   │
├────┼──────────────────────┼─────────────┼──────────┤
│ 1  │ alex@example.com     │ Alex Smith  │ active   │
│ 2  │ priya@example.com    │ Priya Rao   │ active   │
└────┴──────────────────────┴─────────────┴──────────┘
```

---

## 4. Rows and Columns

A row represents one entity instance.

A column represents one attribute of that entity.

```text
Row:
A single user

Columns:
id, email, name, status, created_at
```

Good relational design keeps each column focused on one meaning.

Poor design:

```sql
contact_details TEXT
```

Better design:

```sql
email TEXT,
phone_number TEXT,
country_code TEXT
```

Structured columns are easier to:

* Validate
* Query
* Index
* Sort
* Aggregate

---

## 5. Data Types

PostgreSQL provides many built-in data types.

| Category       | Common Types                               |
| -------------- | ------------------------------------------ |
| Integers       | `SMALLINT`, `INTEGER`, `BIGINT`            |
| Decimal values | `NUMERIC`, `DECIMAL`                       |
| Floating point | `REAL`, `DOUBLE PRECISION`                 |
| Text           | `TEXT`, `VARCHAR`, `CHAR`                  |
| Boolean        | `BOOLEAN`                                  |
| Date and time  | `DATE`, `TIME`, `TIMESTAMP`, `TIMESTAMPTZ` |
| Identifiers    | `UUID`                                     |
| Documents      | `JSON`, `JSONB`                            |
| Network        | `INET`, `CIDR`, `MACADDR`                  |
| Arrays         | `INTEGER[]`, `TEXT[]`                      |
| Range values   | `INT4RANGE`, `DATERANGE`, `TSRANGE`        |

Example:

```sql
CREATE TABLE products (
    id UUID PRIMARY KEY,
    name TEXT NOT NULL,
    price NUMERIC(12, 2) NOT NULL,
    available BOOLEAN NOT NULL DEFAULT TRUE,
    tags TEXT[],
    metadata JSONB,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

Choose data types based on meaning, not only current values.

For money, prefer:

```sql
NUMERIC(12, 2)
```

instead of:

```sql
DOUBLE PRECISION
```

Floating-point types may introduce precision issues.

---

## 6. Primary Key

A primary key uniquely identifies each row.

```sql
CREATE TABLE customers (
    id BIGSERIAL PRIMARY KEY,
    email TEXT NOT NULL
);
```

A primary key provides:

* Uniqueness
* Non-null enforcement
* Efficient row lookup
* A stable reference for relationships

Common primary-key choices include:

### Auto-Incrementing Integer

```sql
id BIGSERIAL PRIMARY KEY
```

Advantages:

* Small index size
* Efficient insertion
* Easy debugging

Disadvantages:

* Predictable
* Harder to generate outside the database
* Can expose approximate record counts

### UUID

```sql
id UUID PRIMARY KEY
```

Advantages:

* Globally unique
* Can be generated by multiple services
* Useful in distributed systems

Disadvantages:

* Larger indexes
* Random UUIDs may increase index fragmentation
* Harder to read manually

Time-ordered UUID variants can improve index locality.

---

## 7. Foreign Key

A foreign key enforces a relationship between tables.

```sql
CREATE TABLE orders (
    id BIGSERIAL PRIMARY KEY,
    customer_id BIGINT NOT NULL,
    total_amount NUMERIC(12, 2) NOT NULL,

    CONSTRAINT fk_orders_customer
        FOREIGN KEY (customer_id)
        REFERENCES customers(id)
);
```

Relationship:

```text
customers
    |
    | One customer
    |
    v
orders
Many orders
```

Foreign keys protect against invalid references.

Without a foreign key:

```text
order.customer_id = 999

Customer 999 does not exist
```

With a foreign key, PostgreSQL rejects the invalid write.

---

## 8. Constraints

Constraints enforce data rules inside the database.

### NOT NULL

```sql
email TEXT NOT NULL
```

### UNIQUE

```sql
email TEXT UNIQUE
```

### CHECK

```sql
price NUMERIC(12, 2) CHECK (price >= 0)
```

### FOREIGN KEY

```sql
FOREIGN KEY (user_id) REFERENCES users(id)
```

### PRIMARY KEY

```sql
id BIGINT PRIMARY KEY
```

Example:

```sql
CREATE TABLE bank_accounts (
    id BIGSERIAL PRIMARY KEY,
    account_number TEXT NOT NULL UNIQUE,
    balance NUMERIC(18, 2) NOT NULL CHECK (balance >= 0),
    status TEXT NOT NULL CHECK (
        status IN ('active', 'blocked', 'closed')
    )
);
```

Constraints prevent invalid data from entering the system, regardless of which application writes to the database.

---

## 9. Sequences

Sequences generate unique numeric values.

```sql
CREATE SEQUENCE order_number_sequence;
```

```sql
SELECT nextval('order_number_sequence');
```

`SERIAL` and `BIGSERIAL` historically use sequences behind the scenes.

Modern PostgreSQL also supports identity columns:

```sql
CREATE TABLE orders (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY
);
```

Identity columns are generally clearer for new schema designs.

---

## 10. CRUD Operations

CRUD stands for:

* Create
* Read
* Update
* Delete

### Create

```sql
INSERT INTO users (email, full_name)
VALUES ('alex@example.com', 'Alex Smith')
RETURNING id;
```

### Read

```sql
SELECT id, email, full_name
FROM users
WHERE email = 'alex@example.com';
```

### Update

```sql
UPDATE users
SET full_name = 'Alexander Smith',
    updated_at = NOW()
WHERE id = 1;
```

### Delete

```sql
DELETE FROM users
WHERE id = 1;
```

Use `RETURNING` to retrieve affected rows without issuing another query.

```sql
UPDATE orders
SET status = 'paid'
WHERE id = 1001
RETURNING id, status, updated_at;
```

---

## 11. Transactions

A transaction groups multiple operations into one logical unit.

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1;

UPDATE accounts
SET balance = balance + 100
WHERE id = 2;

COMMIT;
```

Either both operations succeed or neither should be applied.

```text
Debit Account A
       +
Credit Account B
       =
One Transaction
```

On failure:

```sql
ROLLBACK;
```

Transactions are essential for:

* Payments
* Order placement
* Inventory updates
* Account transfers
* Subscription changes
* Multi-table writes

---

## 12. ACID Properties

PostgreSQL transactions follow ACID principles.

### Atomicity

All operations succeed together or fail together.

```text
Create Order
Reserve Inventory
Create Payment Record

All succeed or all roll back
```

### Consistency

Transactions move the database from one valid state to another valid state.

Constraints help enforce consistency.

### Isolation

Concurrent transactions should not interfere in unsafe ways.

### Durability

After a committed transaction, data should survive crashes when the system is configured correctly.

---

## 13. Isolation Levels

PostgreSQL supports several transaction isolation levels.

| Isolation Level | Dirty Reads | Non-Repeatable Reads |                Serialization Anomalies |
| --------------- | ----------: | -------------------: | -------------------------------------: |
| Read Committed  |   Prevented |             Possible |                               Possible |
| Repeatable Read |   Prevented |            Prevented |            Some transactions may abort |
| Serializable    |   Prevented |            Prevented | Prevented through serialization checks |

PostgreSQL treats `READ UNCOMMITTED` as `READ COMMITTED`.

### Read Committed

Default isolation level.

Each SQL statement sees a snapshot of committed data at the beginning of that statement.

```sql
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

### Repeatable Read

All statements in the transaction see a stable snapshot.

```sql
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

### Serializable

Provides the strongest isolation.

```sql
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

Serializable transactions may fail and require application retries.

---

## 14. Multi-Version Concurrency Control

PostgreSQL uses **Multi-Version Concurrency Control**, commonly called MVCC.

Instead of immediately overwriting a row, PostgreSQL maintains row versions.

```text
Original Row Version
        |
        v
Updated Row Version
```

Different transactions can see different versions based on their snapshots.

Benefits:

* Readers usually do not block writers.
* Writers usually do not block readers.
* Queries receive transactionally consistent views.
* Concurrency is improved.

Conceptual example:

```text
Transaction A sees:
balance = 100

Transaction B updates:
balance = 80

Transaction A may continue seeing:
balance = 100

Transaction B commits:
Future transactions see 80
```

MVCC is one of PostgreSQL's most important architectural concepts.

---

## 15. Dead Tuples

Because PostgreSQL creates new row versions, old versions may remain temporarily.

These old versions are called dead tuples.

```text
Before Update:
Row Version 1

After Update:
Row Version 1 — Dead
Row Version 2 — Active
```

Dead tuples consume storage until they are cleaned up.

---

## 16. VACUUM

`VACUUM` reclaims storage occupied by dead tuples and updates visibility information.

```sql
VACUUM users;
```

Production systems normally rely on autovacuum.

```text
Updates and Deletes
        |
        v
Dead Tuples
        |
        v
Autovacuum
        |
        v
Reusable Space
```

`VACUUM` generally makes space reusable inside the table file.

It does not always return storage directly to the operating system.

---

## 17. Autovacuum

Autovacuum automatically runs maintenance tasks.

It helps:

* Clean dead tuples
* Prevent transaction-ID wraparound
* Update planner statistics
* Maintain table health

Disabling autovacuum globally is dangerous.

A high-write table may require custom autovacuum settings.

Example:

```sql
ALTER TABLE events SET (
    autovacuum_vacuum_scale_factor = 0.02,
    autovacuum_analyze_scale_factor = 0.01
);
```

Autovacuum should be tuned based on table size and write patterns.

---

## 18. ANALYZE

`ANALYZE` collects statistics used by the query planner.

```sql
ANALYZE orders;
```

Statistics help PostgreSQL estimate:

* Number of matching rows
* Value distribution
* Null frequency
* Column correlation
* Distinct values

Poor or outdated statistics can cause inefficient query plans.

---

## 19. Query Planner

PostgreSQL evaluates multiple execution strategies before running a query.

Possible operations include:

* Sequential scan
* Index scan
* Index-only scan
* Bitmap heap scan
* Nested-loop join
* Hash join
* Merge join
* Sort
* Aggregate

The planner chooses a plan based on estimated cost.

```text
SQL Query
   |
   v
Parser
   |
   v
Planner
   |
   v
Execution Plan
   |
   v
Executor
```

---

## 20. EXPLAIN

`EXPLAIN` shows the planned query execution.

```sql
EXPLAIN
SELECT *
FROM orders
WHERE customer_id = 100;
```

`EXPLAIN ANALYZE` executes the query and reports actual timing and row counts.

```sql
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE customer_id = 100;
```

Example output may include:

```text
Index Scan using idx_orders_customer_id on orders
  Index Cond: (customer_id = 100)
```

Use `EXPLAIN ANALYZE` carefully on expensive data-changing queries because the statement is actually executed.

---

## 21. Indexes

Indexes speed up data retrieval at the cost of:

* Additional storage
* Slower inserts
* Slower updates
* Slower deletes
* Maintenance overhead

Example:

```sql
CREATE INDEX idx_orders_customer_id
ON orders(customer_id);
```

Query:

```sql
SELECT *
FROM orders
WHERE customer_id = 100;
```

Without an index:

```text
Scan many rows
```

With an appropriate index:

```text
Navigate directly to matching rows
```

---

## 22. B-Tree Index

B-tree is PostgreSQL's default index type.

```sql
CREATE INDEX idx_users_email
ON users(email);
```

B-tree indexes are useful for:

* Equality
* Range queries
* Sorting
* Prefix ordering

Examples:

```sql
WHERE email = 'alex@example.com'
```

```sql
WHERE created_at >= NOW() - INTERVAL '7 days'
```

```sql
ORDER BY created_at DESC
```

---

## 23. Composite Index

A composite index contains multiple columns.

```sql
CREATE INDEX idx_orders_customer_status
ON orders(customer_id, status);
```

Useful query:

```sql
SELECT *
FROM orders
WHERE customer_id = 100
  AND status = 'pending';
```

Column order matters.

An index on:

```text
(customer_id, status)
```

can efficiently support:

```sql
WHERE customer_id = 100
```

and:

```sql
WHERE customer_id = 100
  AND status = 'pending'
```

It may not efficiently support a query using only:

```sql
WHERE status = 'pending'
```

Design composite indexes around actual query patterns.

---

## 24. Unique Index

A unique index prevents duplicate values.

```sql
CREATE UNIQUE INDEX idx_users_email_unique
ON users(email);
```

This can also be created using a constraint:

```sql
ALTER TABLE users
ADD CONSTRAINT users_email_unique UNIQUE (email);
```

For business rules, constraints usually communicate intent more clearly.

---

## 25. Partial Index

A partial index contains only rows matching a condition.

```sql
CREATE INDEX idx_pending_orders
ON orders(created_at)
WHERE status = 'pending';
```

This is useful when queries frequently target a small subset of a large table.

```sql
SELECT *
FROM orders
WHERE status = 'pending'
ORDER BY created_at;
```

Advantages:

* Smaller index
* Faster maintenance
* Better cache efficiency

---

## 26. Expression Index

An expression index stores the result of an expression.

```sql
CREATE UNIQUE INDEX idx_users_lower_email
ON users(LOWER(email));
```

This supports case-insensitive uniqueness.

```sql
SELECT *
FROM users
WHERE LOWER(email) = LOWER('Alex@Example.com');
```

---

## 27. GIN Index

GIN stands for **Generalized Inverted Index**.

It is commonly used for:

* JSONB
* Arrays
* Full-text search

Example:

```sql
CREATE INDEX idx_products_metadata
ON products
USING GIN(metadata);
```

Query:

```sql
SELECT *
FROM products
WHERE metadata @> '{"brand": "Acme"}';
```

---

## 28. GiST Index

GiST stands for **Generalized Search Tree**.

It is useful for:

* Geometric data
* Range types
* Nearest-neighbor queries
* Full-text search
* PostGIS operations

Example:

```sql
CREATE INDEX idx_reservations_period
ON reservations
USING GIST(reservation_period);
```

---

## 29. BRIN Index

BRIN stands for **Block Range Index**.

It stores summaries for ranges of physical table blocks.

BRIN is useful for very large tables where values correlate with insertion order.

Example:

```sql
CREATE INDEX idx_logs_created_at_brin
ON logs
USING BRIN(created_at);
```

Good candidates include:

* Logs
* Events
* Time-series data
* Append-only tables

BRIN indexes are much smaller than B-tree indexes but are less precise.

---

## 30. Covering Index

A covering index contains the columns required to answer a query.

```sql
CREATE INDEX idx_orders_customer_covering
ON orders(customer_id)
INCLUDE (status, total_amount, created_at);
```

Query:

```sql
SELECT status, total_amount, created_at
FROM orders
WHERE customer_id = 100;
```

PostgreSQL may use an index-only scan when visibility information allows it.

---

## 31. Joins

Joins combine related data from multiple tables.

### INNER JOIN

Returns matching rows.

```sql
SELECT
    orders.id,
    customers.email
FROM orders
INNER JOIN customers
    ON customers.id = orders.customer_id;
```

### LEFT JOIN

Returns all rows from the left table, including those without a match.

```sql
SELECT
    customers.id,
    orders.id AS order_id
FROM customers
LEFT JOIN orders
    ON orders.customer_id = customers.id;
```

### FULL OUTER JOIN

Returns matching and unmatched rows from both sides.

```sql
SELECT *
FROM table_a
FULL OUTER JOIN table_b
    ON table_a.id = table_b.id;
```

---

## 32. Normalization

Normalization organizes data to reduce duplication and inconsistency.

Poor design:

```text
orders

order_id
customer_name
customer_email
customer_address
product_name
product_price
```

Better design:

```text
customers
products
orders
order_items
```

Architecture:

```text
customers
    |
    v
orders
    |
    v
order_items
    |
    v
products
```

Normalization improves:

* Data consistency
* Update safety
* Storage efficiency
* Constraint enforcement

---

## 33. Denormalization

Denormalization intentionally duplicates some data to improve read performance or preserve historical values.

Example:

```sql
CREATE TABLE order_items (
    id BIGSERIAL PRIMARY KEY,
    order_id BIGINT NOT NULL,
    product_id BIGINT NOT NULL,
    product_name_snapshot TEXT NOT NULL,
    unit_price_snapshot NUMERIC(12, 2) NOT NULL
);
```

The snapshot ensures an old order still displays the original product name and price, even if the product changes later.

Denormalization should be deliberate and documented.

---

## 34. JSON and JSONB

PostgreSQL supports both `JSON` and `JSONB`.

### JSON

Stores the original JSON text.

### JSONB

Stores a parsed binary representation.

```sql
CREATE TABLE events (
    id BIGSERIAL PRIMARY KEY,
    event_type TEXT NOT NULL,
    payload JSONB NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

Insert:

```sql
INSERT INTO events (event_type, payload)
VALUES (
    'user_registered',
    '{
        "userId": 101,
        "source": "mobile",
        "campaign": "summer"
    }'
);
```

Query:

```sql
SELECT *
FROM events
WHERE payload->>'source' = 'mobile';
```

Containment query:

```sql
SELECT *
FROM events
WHERE payload @> '{"source": "mobile"}';
```

JSONB is useful for flexible attributes, but it should not automatically replace relational modeling.

---

## 35. Arrays

PostgreSQL supports array columns.

```sql
CREATE TABLE articles (
    id BIGSERIAL PRIMARY KEY,
    title TEXT NOT NULL,
    tags TEXT[] NOT NULL DEFAULT '{}'
);
```

Query:

```sql
SELECT *
FROM articles
WHERE 'postgresql' = ANY(tags);
```

Arrays are useful for small, bounded lists.

For complex relationships or large collections, use a separate table.

---

## 36. Views

A view stores a reusable query definition.

```sql
CREATE VIEW active_customers AS
SELECT id, email, full_name
FROM customers
WHERE status = 'active';
```

Query:

```sql
SELECT *
FROM active_customers;
```

Views can simplify complex queries and control data access.

---

## 37. Materialized Views

A materialized view stores query results physically.

```sql
CREATE MATERIALIZED VIEW daily_sales AS
SELECT
    DATE(created_at) AS sale_date,
    SUM(total_amount) AS revenue
FROM orders
WHERE status = 'paid'
GROUP BY DATE(created_at);
```

Refresh:

```sql
REFRESH MATERIALIZED VIEW daily_sales;
```

Materialized views are useful for expensive analytical queries that do not require real-time freshness.

---

## 38. Common Table Expressions

A Common Table Expression, or CTE, defines a temporary named result within a query.

```sql
WITH customer_totals AS (
    SELECT
        customer_id,
        SUM(total_amount) AS total_spent
    FROM orders
    WHERE status = 'paid'
    GROUP BY customer_id
)
SELECT *
FROM customer_totals
WHERE total_spent > 10000;
```

CTEs improve readability for complex queries.

Recursive CTEs can process hierarchical data.

```sql
WITH RECURSIVE category_tree AS (
    SELECT id, parent_id, name, 1 AS depth
    FROM categories
    WHERE parent_id IS NULL

    UNION ALL

    SELECT c.id, c.parent_id, c.name, ct.depth + 1
    FROM categories c
    JOIN category_tree ct
        ON c.parent_id = ct.id
)
SELECT *
FROM category_tree;
```

---

## 39. Window Functions

Window functions calculate values across related rows without collapsing them into one row per group.

Example:

```sql
SELECT
    customer_id,
    id AS order_id,
    total_amount,
    ROW_NUMBER() OVER (
        PARTITION BY customer_id
        ORDER BY created_at DESC
    ) AS order_rank
FROM orders;
```

Common window functions include:

* `ROW_NUMBER`
* `RANK`
* `DENSE_RANK`
* `LAG`
* `LEAD`
* `SUM`
* `AVG`

Example running total:

```sql
SELECT
    created_at,
    total_amount,
    SUM(total_amount) OVER (
        ORDER BY created_at
    ) AS running_revenue
FROM orders;
```

---

## 40. Upsert

An upsert inserts a row or updates it when a conflict occurs.

```sql
INSERT INTO user_settings (
    user_id,
    theme
)
VALUES (
    101,
    'dark'
)
ON CONFLICT (user_id)
DO UPDATE SET
    theme = EXCLUDED.theme;
```

Upserts are useful for:

* Idempotent writes
* Synchronization
* Configuration updates
* Counters
* Import jobs

---

## 41. Row-Level Locking

PostgreSQL supports locking selected rows.

```sql
BEGIN;

SELECT *
FROM inventory
WHERE product_id = 100
FOR UPDATE;

UPDATE inventory
SET available_quantity = available_quantity - 1
WHERE product_id = 100;

COMMIT;
```

`FOR UPDATE` prevents another transaction from modifying the selected row until the transaction ends.

Other lock modes include:

* `FOR NO KEY UPDATE`
* `FOR SHARE`
* `FOR KEY SHARE`

---

## 42. Optimistic Locking

Optimistic locking detects concurrent changes using a version number.

```sql
CREATE TABLE documents (
    id BIGSERIAL PRIMARY KEY,
    content TEXT NOT NULL,
    version INTEGER NOT NULL DEFAULT 1
);
```

Update:

```sql
UPDATE documents
SET content = 'Updated content',
    version = version + 1
WHERE id = 10
  AND version = 3;
```

If zero rows are updated, another transaction changed the record.

Optimistic locking works well when conflicts are uncommon.

---

## 43. Pessimistic Locking

Pessimistic locking reserves the row before modification.

```sql
SELECT *
FROM seats
WHERE id = 500
FOR UPDATE;
```

This is useful when:

* Conflicts are likely.
* The operation must serialize access.
* Double allocation must be prevented.

Long transactions should be avoided because they keep locks active.

---

## 44. Deadlocks

A deadlock occurs when transactions wait on one another.

```text
Transaction A locks Row 1
Transaction B locks Row 2

Transaction A waits for Row 2
Transaction B waits for Row 1
```

PostgreSQL detects the deadlock and aborts one transaction.

Prevention techniques:

* Lock rows in a consistent order.
* Keep transactions short.
* Avoid unnecessary locks.
* Retry transactions that fail from deadlocks.

---

## 45. Write-Ahead Log

PostgreSQL records changes in the Write-Ahead Log, or WAL, before modified data pages are written to permanent table storage.

```text
Transaction
    |
    v
Write-Ahead Log
    |
    v
Commit
    |
    v
Data Pages Written Later
```

WAL supports:

* Crash recovery
* Replication
* Point-in-time recovery
* Durability

The principle is:

```text
Log the change before modifying the data file.
```

---

## 46. Checkpoints

A checkpoint ensures modified data pages are written to disk and establishes a recovery point.

```text
WAL Records
    |
    v
Dirty Memory Pages
    |
    v
Checkpoint
    |
    v
Persistent Data Files
```

Very frequent checkpoints can increase I/O.

Very infrequent checkpoints can increase recovery time and WAL usage.

Checkpoint configuration should be tuned based on workload and infrastructure.

---

## 47. Replication

Replication copies database changes to other PostgreSQL instances.

```text
Primary
   |
   | WAL Stream
   v
Replica
```

Replication improves:

* Read scalability
* Availability
* Disaster recovery
* Backup flexibility

---

## 48. Streaming Replication

Streaming replication sends WAL records from the primary to replicas.

```text
Applications
      |
      v
Primary PostgreSQL
      |
      | WAL Stream
      v
Read Replica
```

The replica continuously replays changes.

This is commonly used for physical replication.

---

## 49. Synchronous Replication

With synchronous replication, the primary may wait for confirmation from a replica before confirming a commit.

```text
Client
   |
   v
Primary
   |
   | Send WAL
   v
Synchronous Replica
   |
   | Confirm
   v
Primary Confirms Commit
```

Advantages:

* Lower risk of committed-data loss

Disadvantages:

* Higher write latency
* Reduced availability if the required replica is unavailable

---

## 50. Asynchronous Replication

With asynchronous replication, the primary confirms commits without waiting for replicas.

```text
Client
   |
   v
Primary Confirms Commit
   |
   v
WAL Reaches Replica Later
```

Advantages:

* Lower write latency
* Better primary availability

Disadvantages:

* Replica lag
* Possible loss of recent committed transactions during failover

---

## 51. Replication Lag

Replication lag is the delay between a primary commit and replica replay.

```text
Primary State:
Order 100 is paid

Replica State:
Order 100 is still pending
```

This can cause stale reads.

Applications must decide whether a request can tolerate eventual consistency.

Examples:

```text
Product catalog:
Replica read may be acceptable

Payment confirmation:
Read from primary
```

---

## 52. Read Replicas

Read replicas can handle read-only workloads.

```text
                  ┌── Read Replica A
Primary ──────────┼── Read Replica B
                  └── Read Replica C
```

Suitable workloads include:

* Product browsing
* Reporting
* Search support
* Analytics queries
* Administrative dashboards

Read replicas do not automatically solve all scaling problems.

The application must handle:

* Replica lag
* Read routing
* Failover
* Connection management
* Read-after-write consistency

---

## 53. Logical Replication

Logical replication publishes changes at the table level.

```text
Publisher
    |
    | INSERT / UPDATE / DELETE
    v
Subscriber
```

It can be useful for:

* Selective table replication
* Data migration
* Integration pipelines
* Version upgrades
* Event processing

Example:

```sql
CREATE PUBLICATION ecommerce_publication
FOR TABLE orders, customers;
```

Subscriber:

```sql
CREATE SUBSCRIPTION ecommerce_subscription
CONNECTION 'host=source-db dbname=ecommerce user=replicator'
PUBLICATION ecommerce_publication;
```

---

## 54. High Availability

A highly available PostgreSQL deployment commonly includes:

* One primary
* One or more replicas
* Automated failure detection
* Leader promotion
* Connection routing
* Backups
* Monitoring

```text
                      Applications
                           |
                           v
                 Database Proxy or Router
                           |
              ┌────────────┴────────────┐
              │                         │
        Primary PostgreSQL        Replica PostgreSQL
              │                         │
              └──────── WAL ────────────┘
```

During primary failure:

```text
Old Primary:
Unavailable

Replica:
Promoted to Primary
```

Failover automation must avoid split-brain scenarios where two nodes accept writes independently.

---

## 55. Connection Pooling

PostgreSQL uses a process-oriented connection model.

Each database connection consumes resources.

```text
1,000 Application Requests
          |
          v
Connection Pool
          |
          v
50 Database Connections
```

Connection pooling reduces:

* Connection setup overhead
* Memory consumption
* Backend process count
* Authentication overhead

Common deployment pattern:

```text
Application Instances
        |
        v
Connection Pooler
        |
        v
PostgreSQL
```

Pool sizing should be based on database capacity, not application request count.

---

## 56. Connection Pooling Modes

External poolers may support different modes.

### Session Pooling

One backend connection remains assigned for the entire client session.

### Transaction Pooling

A backend connection is assigned only during a transaction.

### Statement Pooling

A backend connection is assigned for one statement.

Transaction pooling improves capacity but may not support session-dependent features such as:

* Temporary tables
* Session-level prepared statements
* Session variables
* Some advisory lock patterns

---

## 57. Partitioning

Partitioning divides one logical table into smaller physical tables.

```text
orders
  |
  ├── orders_2025
  ├── orders_2026
  └── orders_2027
```

Common partitioning strategies:

* Range
* List
* Hash

### Range Partitioning

```sql
CREATE TABLE events (
    id BIGINT GENERATED ALWAYS AS IDENTITY,
    created_at TIMESTAMPTZ NOT NULL,
    payload JSONB NOT NULL
) PARTITION BY RANGE (created_at);
```

```sql
CREATE TABLE events_2026_07
PARTITION OF events
FOR VALUES FROM ('2026-07-01') TO ('2026-08-01');
```

Partitioning is useful for:

* Very large tables
* Time-series data
* Retention management
* Partition pruning
* Faster bulk deletion

Partitioning should not be added before the workload requires it.

---

## 58. Partition Pruning

PostgreSQL can skip partitions that cannot contain matching rows.

Query:

```sql
SELECT *
FROM events
WHERE created_at >= '2026-07-01'
  AND created_at < '2026-08-01';
```

Execution:

```text
Scan:
events_2026_07

Skip:
events_2026_06
events_2026_08
```

Pruning reduces the amount of data scanned.

---

## 59. Table Bloat

Table bloat occurs when dead tuples and unused space grow significantly.

Potential causes include:

* Heavy updates
* Heavy deletes
* Long-running transactions
* Insufficient autovacuum
* Poor fill-factor choices

Effects:

* Larger tables
* Larger indexes
* More disk I/O
* Slower scans
* Longer maintenance operations

Possible tools include:

* `VACUUM`
* `VACUUM FULL`
* Online table-rewrite extensions
* Reindexing
* Better autovacuum tuning

`VACUUM FULL` requires stronger locks and should be planned carefully.

---

## 60. Backups

Replication is not a backup.

A replica may copy:

* Accidental deletes
* Corrupt writes
* Dropped tables
* Application bugs

A production backup strategy should include:

* Regular base backups
* WAL archiving
* Off-site storage
* Encryption
* Retention policies
* Restore testing

```text
Primary Database
       |
       ├── Replica
       |
       └── Backup Storage
              |
              └── Point-in-Time Recovery
```

---

## 61. Point-in-Time Recovery

Point-in-time recovery restores a base backup and replays WAL until a selected time.

Example:

```text
12:00 — Base backup
14:25 — Table accidentally deleted
14:30 — Incident detected

Restore to:
14:24:59
```

Point-in-time recovery helps recover from logical mistakes.

---

## 62. Extensions

PostgreSQL supports extensions that add new capabilities.

Examples include:

* PostGIS for geospatial data
* `pg_trgm` for text similarity
* `citext` for case-insensitive text
* `hstore` for key-value storage
* `uuid-ossp` for UUID generation
* `pgcrypto` for cryptographic functions

Example:

```sql
CREATE EXTENSION IF NOT EXISTS pg_trgm;
```

Extensions should be reviewed for:

* Operational support
* Upgrade compatibility
* Security
* Managed-database availability

---

## 63. Full-Text Search

PostgreSQL includes full-text-search features.

```sql
SELECT *
FROM articles
WHERE to_tsvector('english', title || ' ' || body)
      @@ plainto_tsquery('english', 'database indexing');
```

A stored search vector can be indexed using GIN.

```sql
CREATE INDEX idx_articles_search
ON articles
USING GIN (
    to_tsvector('english', title || ' ' || body)
);
```

PostgreSQL search is useful for moderate search requirements.

Dedicated search engines may be better for:

* Complex ranking
* Typo tolerance
* Large-scale faceting
* Advanced linguistic analysis
* Search-specific analytics

---

# Architecture

## Basic PostgreSQL Architecture

```text
                        Client Applications
                                 |
                                 v
                         Connection Pooler
                                 |
                                 v
                    ┌────────────────────────┐
                    │ PostgreSQL Server      │
                    │                        │
                    │ ┌────────────────────┐ │
                    │ │ Backend Processes  │ │
                    │ └─────────┬──────────┘ │
                    │           │            │
                    │ ┌─────────▼──────────┐ │
                    │ │ Shared Buffers     │ │
                    │ └─────────┬──────────┘ │
                    │           │            │
                    │ ┌─────────▼──────────┐ │
                    │ │ WAL Buffers        │ │
                    │ └─────────┬──────────┘ │
                    │           │            │
                    │ ┌─────────▼──────────┐ │
                    │ │ Background Workers │ │
                    │ │ Checkpointer       │ │
                    │ │ WAL Writer         │ │
                    │ │ Autovacuum         │ │
                    │ └─────────┬──────────┘ │
                    └───────────┼────────────┘
                                │
                 ┌──────────────┴──────────────┐
                 │                             │
          Table and Index Files           WAL Files
```

---

## Query Execution Flow

```text
SQL Query
   |
   v
Parser
   |
   v
Analyzer
   |
   v
Rewriter
   |
   v
Query Planner
   |
   v
Execution Plan
   |
   v
Executor
   |
   v
Rows Returned
```

### Parser

Checks SQL syntax and creates an internal representation.

### Analyzer

Resolves tables, columns, data types, and functions.

### Rewriter

Applies rules and view definitions.

### Planner

Evaluates possible execution strategies.

### Executor

Runs the selected plan.

---

## Production Architecture

```text
                            Clients
                               |
                               v
                         API Gateway
                               |
              ┌────────────────┴────────────────┐
              │                                 │
       Application Service A             Application Service B
              │                                 │
              └────────────────┬────────────────┘
                               │
                         Connection Pool
                               |
                               v
                     Database Proxy / Router
                               |
             ┌─────────────────┴──────────────────┐
             │                                    │
       PostgreSQL Primary                   Read Replica
             │                                    │
             │ WAL Replication                    │
             └────────────────────────────────────┘
             |
       ┌─────┴───────────────┐
       │                     │
 Backup Storage         Monitoring System
       │
       v
Point-in-Time Recovery
```

---

## Multi-Region Architecture

```text
                         Global Application
                                |
             ┌──────────────────┴──────────────────┐
             │                                     │
         Region A                              Region B
             │                                     │
     Application Cluster                    Application Cluster
             │                                     │
             v                                     v
      PostgreSQL Primary  ───── Replication ───> Read Replica
             |
             v
       Backup Storage
```

Cross-region replication is usually asynchronous because network latency makes synchronous writes expensive.

Trade-offs include:

* Lower disaster-recovery time
* Replica lag
* Potential recent-data loss
* Failover complexity
* Data-residency requirements

---

## Read and Write Routing

```text
                    Application
                         |
             ┌───────────┴───────────┐
             │                       │
          Writes                   Reads
             │                       │
             v                       v
          Primary              Read Replica
```

Not all reads should go to replicas.

Read from the primary when:

* Read-after-write consistency is required.
* The latest payment state is required.
* A transaction includes both reads and writes.
* Replica lag is unacceptable.

Read from replicas when:

* Slightly stale data is acceptable.
* The workload is reporting-heavy.
* The query does not affect business correctness.

---

# Comparisons

## PostgreSQL vs MySQL

| Category          | PostgreSQL                    | MySQL                                      |
| ----------------- | ----------------------------- | ------------------------------------------ |
| SQL capabilities  | Very advanced                 | Strong and widely used                     |
| Complex queries   | Excellent                     | Good                                       |
| JSON support      | Strong JSONB support          | Strong JSON support                        |
| Extensions        | Rich extension ecosystem      | More limited                               |
| Geospatial        | Excellent with PostGIS        | Supported                                  |
| Full-text search  | Built in                      | Built in                                   |
| Concurrency model | MVCC                          | MVCC through common storage engines        |
| Common use        | Complex transactional systems | Web applications and transactional systems |
| Learning curve    | Moderate                      | Often considered simpler initially         |

Both are strong relational databases.

PostgreSQL is often preferred when advanced SQL, extensibility, complex data relationships, or strict standards support are important.

---

## PostgreSQL vs MongoDB

| Category            | PostgreSQL                                | MongoDB                        |
| ------------------- | ----------------------------------------- | ------------------------------ |
| Data model          | Relational                                | Document                       |
| Schema              | Explicit, with optional JSONB flexibility | Flexible document schema       |
| Transactions        | Mature ACID transactions                  | Supported                      |
| Joins               | Powerful SQL joins                        | Aggregation and lookup support |
| Constraints         | Strong                                    | More application-driven        |
| Query language      | SQL                                       | Document-query API             |
| Best fit            | Relational and transactional systems      | Document-centered systems      |
| Flexible attributes | JSONB                                     | Native documents               |

PostgreSQL with JSONB can support flexible fields while preserving relational constraints for core entities.

MongoDB may be a better fit when the natural unit of storage is a self-contained document with limited cross-document relationships.

---

## PostgreSQL vs Redis

| Category     | PostgreSQL                         | Redis                               |
| ------------ | ---------------------------------- | ----------------------------------- |
| Primary role | System of record                   | In-memory data platform             |
| Storage      | Disk-backed relational data        | Primarily memory-oriented           |
| Query model  | SQL                                | Command and data-structure based    |
| Transactions | Full relational transactions       | Limited transaction semantics       |
| Latency      | Low milliseconds in many workloads | Often sub-millisecond               |
| Data size    | Large persistent datasets          | Limited by memory and configuration |
| Best use     | Durable business data              | Cache, counters, locks, sessions    |

A common architecture uses both:

```text
Application
    |
    ├── Redis Cache
    |
    └── PostgreSQL Source of Truth
```

---

## PostgreSQL vs Elasticsearch

| Category         | PostgreSQL                       | Elasticsearch                                       |
| ---------------- | -------------------------------- | --------------------------------------------------- |
| Primary role     | Transactional database           | Search and analytics engine                         |
| Consistency      | Strong transactional guarantees  | Search-oriented distributed consistency             |
| Full-text search | Good                             | Advanced                                            |
| Joins            | Strong                           | Limited                                             |
| Aggregations     | Strong SQL                       | Powerful distributed search aggregations            |
| Source of truth  | Yes                              | Usually not recommended as the only source of truth |
| Best use         | Transactions and relational data | Search, logs, observability                         |

Common architecture:

```text
PostgreSQL
    |
    | Change Events
    v
Message Broker
    |
    v
Search Index
```

PostgreSQL remains the source of truth while Elasticsearch serves search traffic.

---

## PostgreSQL vs Cassandra

| Category                 | PostgreSQL                    | Cassandra                                          |
| ------------------------ | ----------------------------- | -------------------------------------------------- |
| Data model               | Relational                    | Wide-column                                        |
| Consistency              | Strong by default             | Tunable                                            |
| Transactions             | Rich ACID transactions        | Limited across partitions                          |
| Query flexibility        | High                          | Query-driven schema                                |
| Horizontal write scaling | More difficult                | Strong                                             |
| Joins                    | Supported                     | Not supported                                      |
| Best use                 | Complex transactional systems | Massive distributed writes and predictable queries |

Choose Cassandra when:

* Massive write throughput is required.
* Multi-region availability is critical.
* Queries are predictable.
* Denormalized data models are acceptable.

Choose PostgreSQL when:

* Relationships matter.
* Transactions span multiple rows.
* Flexible SQL queries are important.
* Strong constraints are required.

---

## PostgreSQL vs SQLite

| Category           | PostgreSQL                 | SQLite                                 |
| ------------------ | -------------------------- | -------------------------------------- |
| Deployment         | Database server            | Embedded file                          |
| Concurrent writers | Strong support             | Limited compared with server databases |
| Network access     | Native                     | Not designed as a remote server        |
| Administration     | Required                   | Minimal                                |
| Best use           | Backend production systems | Embedded apps, local tools, testing    |

SQLite is excellent for local or embedded use.

PostgreSQL is better for multi-user backend services.

---

## Primary-Replica vs Sharding

| Category           | Primary-Replica               | Sharding                                   |
| ------------------ | ----------------------------- | ------------------------------------------ |
| Main goal          | Read scaling and availability | Data and write scaling                     |
| Write location     | Usually one primary           | Multiple shard leaders                     |
| Complexity         | Moderate                      | High                                       |
| Cross-node queries | Usually straightforward       | Difficult                                  |
| Transactions       | Full on primary               | Cross-shard transactions are complex       |
| Failure handling   | Replica promotion             | Per-shard failover                         |
| Best use           | Most growing systems          | Very large workloads exceeding one primary |

Prefer vertical scaling, query optimization, caching, replicas, and partitioning before introducing application-level sharding.

---

# Real-World Example: E-Commerce Order Platform

Consider an e-commerce platform with these requirements:

* Customer accounts
* Product catalog
* Inventory management
* Shopping carts
* Orders
* Payments
* Read replicas
* Reliable transaction processing
* Reporting
* Auditing

---

## Data Model

```text
customers
    |
    v
orders
    |
    v
order_items
    |
    v
products
    |
    v
inventory
```

Tables:

```sql
CREATE TABLE customers (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email TEXT NOT NULL UNIQUE,
    full_name TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE products (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    sku TEXT NOT NULL UNIQUE,
    name TEXT NOT NULL,
    current_price NUMERIC(12, 2) NOT NULL CHECK (current_price >= 0),
    active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE inventory (
    product_id BIGINT PRIMARY KEY
        REFERENCES products(id),
    available_quantity INTEGER NOT NULL
        CHECK (available_quantity >= 0),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE orders (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_id BIGINT NOT NULL
        REFERENCES customers(id),
    status TEXT NOT NULL CHECK (
        status IN (
            'pending',
            'confirmed',
            'paid',
            'shipped',
            'cancelled'
        )
    ),
    total_amount NUMERIC(12, 2) NOT NULL
        CHECK (total_amount >= 0),
    idempotency_key TEXT NOT NULL UNIQUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE order_items (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    order_id BIGINT NOT NULL
        REFERENCES orders(id)
        ON DELETE CASCADE,
    product_id BIGINT NOT NULL
        REFERENCES products(id),
    quantity INTEGER NOT NULL
        CHECK (quantity > 0),
    unit_price NUMERIC(12, 2) NOT NULL
        CHECK (unit_price >= 0),
    UNIQUE (order_id, product_id)
);
```

---

## Architecture

```text
                         Customer
                            |
                            v
                       API Gateway
                            |
                            v
                     Order Service
                            |
                     Connection Pool
                            |
                            v
                 PostgreSQL Primary
                    /             \
                   /               \
          Read Replica          Backup Storage
               |                     |
               v                     v
        Reporting Service      Point-in-Time Recovery
```

Supporting systems:

```text
PostgreSQL
    |
    | Outbox Events
    v
Message Broker
    |
    ├── Payment Service
    ├── Email Service
    ├── Shipping Service
    └── Analytics Pipeline
```

---

## Place Order Transaction

The system must:

1. Validate the product.
2. Lock inventory.
3. Check available stock.
4. Create the order.
5. Create order items.
6. Reduce inventory.
7. Store an event for downstream processing.
8. Commit atomically.

```sql
BEGIN;

SELECT available_quantity
FROM inventory
WHERE product_id = 101
FOR UPDATE;

UPDATE inventory
SET available_quantity = available_quantity - 2,
    updated_at = NOW()
WHERE product_id = 101
  AND available_quantity >= 2;

INSERT INTO orders (
    customer_id,
    status,
    total_amount,
    idempotency_key
)
VALUES (
    5001,
    'confirmed',
    199.98,
    'checkout-request-abc123'
)
RETURNING id;
```

Suppose the returned order ID is `9001`.

```sql
INSERT INTO order_items (
    order_id,
    product_id,
    quantity,
    unit_price
)
VALUES (
    9001,
    101,
    2,
    99.99
);

INSERT INTO outbox_events (
    aggregate_type,
    aggregate_id,
    event_type,
    payload
)
VALUES (
    'order',
    '9001',
    'order_confirmed',
    '{
        "orderId": 9001,
        "customerId": 5001,
        "totalAmount": 199.98
    }'
);

COMMIT;
```

If any step fails:

```sql
ROLLBACK;
```

This ensures the order and inventory remain consistent.

---

## Preventing Overselling

Two customers may attempt to purchase the final item simultaneously.

Without locking:

```text
Customer A reads quantity = 1
Customer B reads quantity = 1

Customer A purchases
Customer B purchases

Result:
Oversold
```

With an atomic update:

```sql
UPDATE inventory
SET available_quantity = available_quantity - 1
WHERE product_id = 101
  AND available_quantity >= 1
RETURNING available_quantity;
```

Only one transaction can successfully reduce the quantity from `1` to `0`.

The other transaction updates zero rows and should return an out-of-stock response.

---

## Idempotent Order Creation

Clients may retry after a timeout.

```text
Client sends checkout request
        |
        v
Server commits order
        |
        v
Response is lost
        |
        v
Client retries
```

Without idempotency:

```text
Two orders may be created
```

With a unique idempotency key:

```sql
idempotency_key TEXT NOT NULL UNIQUE
```

The retry can return the original order instead of creating a duplicate.

```sql
SELECT *
FROM orders
WHERE idempotency_key = 'checkout-request-abc123';
```

---

## Transactional Outbox Pattern

Publishing directly to a message broker after committing creates a failure window.

```text
Database Commit
      |
      v
Application Crashes
      |
      v
Event Never Published
```

The outbox pattern writes the event in the same database transaction.

```text
Order Insert
     +
Outbox Event Insert
     +
One PostgreSQL Transaction
```

A separate worker publishes pending events.

```text
Outbox Table
    |
    v
Publisher Worker
    |
    v
Message Broker
```

Example table:

```sql
CREATE TABLE outbox_events (
    id BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    aggregate_type TEXT NOT NULL,
    aggregate_id TEXT NOT NULL,
    event_type TEXT NOT NULL,
    payload JSONB NOT NULL,
    published_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

Index:

```sql
CREATE INDEX idx_outbox_unpublished
ON outbox_events(created_at)
WHERE published_at IS NULL;
```

Worker query:

```sql
SELECT *
FROM outbox_events
WHERE published_at IS NULL
ORDER BY created_at
FOR UPDATE SKIP LOCKED
LIMIT 100;
```

`SKIP LOCKED` allows multiple workers to process different rows concurrently.

---

## Read Replica Usage

Read replicas can serve:

* Product browsing
* Order-history reporting
* Business dashboards
* Revenue reports

Sensitive reads should remain on the primary.

Example:

```text
Immediately after checkout:

GET /orders/9001
        |
        v
Primary Database
```

A replica might not contain the new order yet because of replication lag.

---

## Useful Indexes

```sql
CREATE INDEX idx_orders_customer_created
ON orders(customer_id, created_at DESC);

CREATE INDEX idx_orders_status_created
ON orders(status, created_at);

CREATE INDEX idx_order_items_order_id
ON order_items(order_id);

CREATE INDEX idx_products_active
ON products(id)
WHERE active = TRUE;
```

These indexes support common application queries.

---

## Reporting Query

```sql
SELECT
    DATE_TRUNC('day', created_at) AS order_day,
    COUNT(*) AS order_count,
    SUM(total_amount) AS revenue
FROM orders
WHERE status IN ('paid', 'shipped')
  AND created_at >= NOW() - INTERVAL '30 days'
GROUP BY DATE_TRUNC('day', created_at)
ORDER BY order_day;
```

For expensive reporting, consider:

* Read replicas
* Materialized views
* Analytical data warehouses
* Incremental aggregation

---

# Best Practices

## 1. Model Data Around Business Rules

Use the database schema to express important rules.

```sql
email TEXT NOT NULL UNIQUE
```

```sql
quantity INTEGER NOT NULL CHECK (quantity > 0)
```

```sql
customer_id BIGINT NOT NULL REFERENCES customers(id)
```

Application validation improves user experience.

Database constraints protect the source of truth.

Use both.

---

## 2. Use Explicit Constraints

Do not rely only on application code for:

* Required values
* Uniqueness
* Valid ranges
* Relationships
* Allowed statuses

Multiple services, scripts, and administrators may write to the same database.

Constraints protect data from every writer.

---

## 3. Use Transactions for Related Writes

Operations that must succeed together belong in one transaction.

Examples:

* Order and order items
* Account debit and credit
* Inventory reservation and order creation
* Subscription update and billing record

Keep transactions short to reduce locking and contention.

---

## 4. Add Indexes Based on Query Patterns

Do not index every column.

Start with actual access patterns:

```sql
SELECT *
FROM orders
WHERE customer_id = ?
ORDER BY created_at DESC;
```

Suitable index:

```sql
CREATE INDEX idx_orders_customer_created
ON orders(customer_id, created_at DESC);
```

Every index should have a clear purpose.

---

## 5. Review Query Plans

Use:

```sql
EXPLAIN
```

and:

```sql
EXPLAIN ANALYZE
```

Look for:

* Unexpected sequential scans
* Bad row estimates
* Expensive sorts
* Repeated nested loops
* Large temporary files
* Rows removed by filters
* Indexes that are not used

Do not optimize queries only by intuition.

---

## 6. Avoid `SELECT *` in Production Queries

Poor:

```sql
SELECT *
FROM users;
```

Better:

```sql
SELECT id, email, full_name, status
FROM users;
```

Selecting only required columns:

* Reduces network transfer
* Reduces memory usage
* Improves API stability
* May enable index-only scans

---

## 7. Use Pagination Carefully

Offset pagination is simple:

```sql
SELECT id, created_at
FROM orders
ORDER BY created_at DESC
LIMIT 20 OFFSET 10000;
```

Large offsets become expensive because rows still need to be skipped.

Prefer keyset pagination for large datasets:

```sql
SELECT id, created_at
FROM orders
WHERE (created_at, id) < (
    '2026-07-30 10:00:00+00',
    50000
)
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

Keyset pagination provides:

* Better performance
* Stable ordering
* Less duplication during concurrent inserts

---

## 8. Use Connection Pooling

Do not open an unlimited number of database connections.

Configure:

* Maximum pool size
* Acquisition timeout
* Idle timeout
* Connection lifetime
* Health checks

Example architecture:

```text
50 Application Instances
           |
           v
      Connection Pool
           |
           v
  100 Database Connections
```

More connections do not always mean more throughput.

Excessive connections can increase context switching and memory usage.

---

## 9. Set Statement Timeouts

Prevent runaway queries.

```sql
SET statement_timeout = '5s';
```

Other useful timeouts include:

* `lock_timeout`
* `idle_in_transaction_session_timeout`
* Connection acquisition timeout
* Application request timeout

Timeouts should align across the request path.

---

## 10. Avoid Long-Running Transactions

Long transactions can:

* Hold locks
* Delay vacuum cleanup
* Increase table bloat
* Increase replication lag
* Consume connection capacity

Poor pattern:

```text
Begin Transaction
Call External Payment API
Wait 10 Seconds
Update Database
Commit
```

Better pattern:

* Keep external network calls outside database transactions.
* Use state machines.
* Use idempotency.
* Use transactional outbox events.

---

## 11. Keep Autovacuum Enabled

Autovacuum is essential for PostgreSQL health.

Monitor:

* Dead tuples
* Vacuum frequency
* Analyze frequency
* Long-running transactions
* Wraparound risk
* Vacuum duration

Tune large or high-write tables individually.

---

## 12. Design for Read-After-Write Consistency

Do not route a user to a lagging replica immediately after a write when the latest state is required.

Possible strategies:

* Read from primary after writes.
* Use session stickiness.
* Track replication positions.
* Wait until the replica catches up.
* Tolerate stale data only for selected endpoints.

---

## 13. Use Appropriate Primary Keys

Prefer stable, meaningless identifiers.

Avoid using mutable business values such as email addresses as primary keys.

Poor:

```sql
email TEXT PRIMARY KEY
```

Better:

```sql
id BIGINT PRIMARY KEY,
email TEXT NOT NULL UNIQUE
```

Business values can change.

Primary keys should remain stable.

---

## 14. Use `TIMESTAMPTZ` for Global Systems

Prefer:

```sql
created_at TIMESTAMPTZ
```

for timestamps representing a real moment in time.

Store timestamps consistently and convert them for presentation.

Avoid storing ambiguous local timestamps for cross-region applications.

---

## 15. Store Money Precisely

Use:

```sql
NUMERIC(12, 2)
```

or integer minor units:

```sql
amount_in_cents BIGINT
```

Avoid floating-point values for financial calculations.

---

## 16. Plan Schema Migrations

Production migrations should consider:

* Table locks
* Long-running rewrites
* Index build time
* Backward compatibility
* Rollback
* Application deployment order

Safer migration pattern:

```text
1. Add nullable column
2. Deploy code that writes both formats
3. Backfill existing rows
4. Deploy code that reads new format
5. Add constraint
6. Remove old column later
```

Use concurrent index creation when appropriate:

```sql
CREATE INDEX CONCURRENTLY idx_orders_customer_id
ON orders(customer_id);
```

Concurrent creation reduces blocking but requires additional time and resources.

---

## 17. Use Backward-Compatible Deployments

Do not deploy application code that immediately depends on a destructive schema change.

Prefer expand-and-contract migrations.

```text
Expand:
Add new schema

Migrate:
Backfill and update applications

Contract:
Remove old schema later
```

This supports rolling deployments and safer rollback.

---

## 18. Separate Online and Analytical Workloads

Large analytical queries can consume:

* CPU
* Memory
* Disk I/O
* Temporary storage
* Connection capacity

Consider:

* Read replicas
* Materialized views
* Data warehouses
* Incremental pipelines
* Query timeouts

Protect transactional workloads from reporting spikes.

---

## 19. Monitor Database Health

Track at least:

* Query latency
* Queries per second
* Active connections
* Waiting connections
* Lock waits
* Deadlocks
* Cache hit ratio
* Disk usage
* WAL generation
* Replication lag
* Autovacuum progress
* Dead tuples
* Checkpoint frequency
* Long-running queries
* Transaction rate

Observability is essential for diagnosing database incidents.

---

## 20. Back Up and Test Restores

A backup is useful only when it can be restored.

Regularly test:

* Full restore
* Point-in-time recovery
* Backup encryption
* Backup retention
* Cross-region recovery
* Recovery-time objectives
* Recovery-point objectives

Document the recovery process.

---

## 21. Use Least-Privilege Database Roles

Create separate roles for:

* Application runtime
* Read-only analytics
* Migrations
* Administration
* Replication
* Backup

Example:

```sql
CREATE ROLE app_runtime LOGIN PASSWORD 'secure-secret';

GRANT CONNECT ON DATABASE ecommerce TO app_runtime;
GRANT USAGE ON SCHEMA public TO app_runtime;
GRANT SELECT, INSERT, UPDATE, DELETE
ON ALL TABLES IN SCHEMA public
TO app_runtime;
```

The application should not run as a database superuser.

---

## 22. Encrypt Connections

Use TLS between:

* Application and database
* Pooler and database
* Replication nodes
* Administrative clients

Also protect:

* Database credentials
* Backup files
* Encryption keys
* Connection strings

---

## 23. Use Prepared Statements Carefully

Prepared statements can:

* Reduce parse overhead
* Improve security
* Prevent SQL injection when parameterized correctly

Example:

```sql
SELECT id, email
FROM users
WHERE id = $1;
```

Always use parameters for user-controlled input.

Never concatenate raw input into SQL.

---

## 24. Use Advisory Locks Only for Suitable Problems

PostgreSQL advisory locks can coordinate application tasks.

Example:

```sql
SELECT pg_try_advisory_lock(12345);
```

Suitable uses:

* Preventing duplicate scheduled jobs
* Serializing rare administrative operations
* Coordinating workers

They should not replace correctly modeled row locks or constraints.

---

## 25. Archive or Partition Very Large Tables

For append-heavy data such as:

* Audit records
* Events
* Logs
* Notifications

Use:

* Time-based partitioning
* Retention policies
* Archive storage
* Bulk partition deletion

Dropping an old partition can be far cheaper than deleting billions of individual rows.

---

# Common Mistakes

## 1. Creating Too Many Indexes

Every index increases write cost and storage.

A table with many indexes may become slow to:

* Insert
* Update
* Delete
* Vacuum
* Replicate

Create indexes for real query patterns and remove unused ones carefully.

---

## 2. Missing Indexes on Foreign Keys

PostgreSQL does not automatically create an index on every foreign-key column.

Example:

```sql
orders.customer_id
```

Without an index, joins and parent-row deletes may become expensive.

Add an index when access patterns require it:

```sql
CREATE INDEX idx_orders_customer_id
ON orders(customer_id);
```

---

## 3. Treating JSONB as a Replacement for Schema Design

Poor:

```sql
CREATE TABLE users (
    data JSONB
);
```

This makes it harder to enforce:

* Required fields
* Uniqueness
* Foreign keys
* Data types
* Efficient queries

Use relational columns for core business data.

Use JSONB for genuinely flexible attributes.

---

## 4. Keeping Transactions Open During Network Calls

Do not hold a transaction open while calling:

* Payment gateways
* Email providers
* Other microservices
* Object storage
* External APIs

This increases lock time and connection usage.

Use asynchronous workflows and state transitions instead.

---

## 5. Assuming Replicas Are Immediately Consistent

A read replica may lag behind the primary.

Reading immediately after a write may return stale data.

Route consistency-sensitive reads to the primary.

---

## 6. Using Offset Pagination for Huge Datasets

Large offsets become progressively slower.

Use keyset pagination for high-volume endpoints.

---

## 7. Ignoring Query Plans

A query that works with 1,000 rows may fail badly with 100 million rows.

Inspect execution plans before and after data growth.

---

## 8. Running Without Connection Limits

Unlimited application connections can overwhelm PostgreSQL.

Use bounded pools and reject or queue excess demand.

---

## 9. Disabling Autovacuum

Disabling autovacuum can cause:

* Severe table bloat
* Poor query performance
* Transaction-ID wraparound risk
* Operational outages

Tune it instead of disabling it.

---

## 10. Using Floating Point for Money

Floating-point values can produce precision errors.

Use `NUMERIC` or integer minor units.

---

## 11. Using `VARCHAR(255)` Everywhere

The number `255` is often inherited from old conventions without business meaning.

Prefer:

```sql
TEXT
```

unless there is a real domain limit.

Use a check constraint when the limit matters:

```sql
name TEXT CHECK (LENGTH(name) <= 100)
```

---

## 12. Deleting Large Amounts of Data in One Transaction

A massive delete can create:

* Long locks
* Large WAL volume
* Replica lag
* Table bloat
* Long rollback time

Delete in batches or use partitioning.

---

## 13. Ignoring Null Semantics

`NULL` means unknown or absent.

This query does not match nulls:

```sql
WHERE status != 'active'
```

Use:

```sql
WHERE status IS DISTINCT FROM 'active'
```

or explicitly handle null:

```sql
WHERE status != 'active'
   OR status IS NULL
```

---

## 14. Using Application Checks Instead of Unique Constraints

Unsafe pattern:

```text
1. Query whether email exists
2. No row found
3. Insert email
```

Two concurrent requests may both pass the check.

Use a unique constraint:

```sql
email TEXT NOT NULL UNIQUE
```

Then handle the constraint violation safely.

---

## 15. Running Destructive Migrations Without Planning

Operations such as:

* Dropping columns
* Changing data types
* Adding expensive defaults
* Rewriting large tables
* Building indexes

may block production traffic.

Test migrations with production-like data and use staged deployment patterns.

---

## 16. Assuming Replication Replaces Backups

Replication copies mistakes.

If a table is dropped on the primary, the drop is also replicated.

Maintain independent backups and test restores.

---

## 17. Storing Comma-Separated Values

Poor:

```text
tags = "postgresql,database,backend"
```

This is hard to validate, query, and index.

Use:

* Arrays for small bounded lists
* Join tables for relationships
* JSONB for flexible structured data

---

## 18. Selecting Rows Without Deterministic Ordering

Poor pagination query:

```sql
SELECT *
FROM orders
LIMIT 20;
```

The database does not guarantee a stable order.

Use:

```sql
SELECT *
FROM orders
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

Always include a deterministic tie-breaker.

---

## 19. Using the Database as an Unlimited Job Queue

PostgreSQL can support moderate queue workloads using:

```sql
FOR UPDATE SKIP LOCKED
```

However, extreme event throughput, long retention, and complex consumer semantics may be better served by a dedicated message broker.

Choose based on workload rather than fashion.

---

## 20. Sharding Too Early

Sharding introduces:

* Cross-shard query complexity
* Distributed transactions
* Rebalancing
* Routing logic
* Operational overhead
* More difficult migrations

First consider:

* Query optimization
* Better indexes
* Vertical scaling
* Caching
* Read replicas
* Partitioning
* Archiving

---

# Interview Questions

## 1. What is MVCC in PostgreSQL?

MVCC keeps multiple versions of rows so transactions can read consistent snapshots while other transactions update data. It improves concurrency because readers usually do not block writers.

---

## 2. What is the purpose of the Write-Ahead Log?

The Write-Ahead Log records changes before data pages are written. It supports durability, crash recovery, replication, and point-in-time recovery.

---

## 3. What is the difference between an index and a constraint?

An index improves data-access performance, while a constraint enforces a business or integrity rule. Some constraints, such as primary keys and unique constraints, use indexes internally.

---

## 4. Why can reading from a PostgreSQL replica return stale data?

Most replicas use asynchronous replication. A commit may be confirmed on the primary before the replica receives and replays the corresponding WAL records.

---

## 5. When should you use partitioning?

Use partitioning when very large tables have clear partition keys, commonly dates or tenant ranges, and the workload benefits from partition pruning, easier retention, or smaller maintenance units.

---

# Key Takeaways

1. **PostgreSQL is a strong default database for backend systems because it combines ACID transactions, relational constraints, advanced SQL, JSONB, powerful indexing, and a mature operational ecosystem.**

2. **Production performance depends more on good schema design, correct indexes, short transactions, connection pooling, autovacuum health, and query-plan analysis than on adding unnecessary infrastructure.**

3. **Scale PostgreSQL progressively: optimize queries first, then add caching, replicas, partitioning, and stronger hardware before accepting the complexity of application-level sharding.**
