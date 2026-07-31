# MySQL

MySQL is one of the world’s most widely used relational database management systems. It powers web applications, SaaS products, e-commerce platforms, financial services, content-management systems, and large-scale distributed architectures.

```text
Clients
   |
   v
Backend Services
   |
   v
Connection Pool
   |
   v
MySQL
   |
   ├── Tables
   ├── Indexes
   ├── Transactions
   ├── Binary Logs
   ├── Replicas
   └── Backups
```

MySQL is a strong choice when a system requires:

* Structured relational data
* ACID transactions
* Strong data integrity
* Efficient indexed queries
* Mature operational tooling
* Read replication
* High availability
* Predictable query patterns
* Broad framework and cloud support

---

# Core Concepts

## 1. Relational Database

MySQL stores data in tables made of rows and columns.

```text
users

┌────┬─────────────────────┬──────────────┬──────────┐
│ id │ email               │ name         │ status   │
├────┼─────────────────────┼──────────────┼──────────┤
│ 1  │ alex@example.com    │ Alex Smith   │ active   │
│ 2  │ priya@example.com   │ Priya Rao    │ active   │
└────┴─────────────────────┴──────────────┴──────────┘
```

A relational model represents connections between entities.

```text
customers
    |
    | One customer
    v
orders
    |
    | One order
    v
order_items
```

Relationships are commonly enforced using foreign keys.

---

## 2. Database and Schema

In MySQL, the terms **database** and **schema** are effectively interchangeable.

```sql
CREATE DATABASE ecommerce;

USE ecommerce;
```

A database contains objects such as:

* Tables
* Views
* Indexes
* Triggers
* Stored procedures
* Functions
* Events

Example:

```text
ecommerce
    |
    ├── customers
    ├── products
    ├── inventory
    ├── orders
    ├── order_items
    └── payments
```

---

## 3. Tables

Tables store application data.

```sql
CREATE TABLE users (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    full_name VARCHAR(150) NOT NULL,
    status ENUM('active', 'blocked', 'deleted') NOT NULL DEFAULT 'active',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
        ON UPDATE CURRENT_TIMESTAMP
);
```

A well-designed table should contain:

* A stable primary key
* Appropriate data types
* Required constraints
* Useful indexes
* Clear ownership of each attribute

---

## 4. Rows and Columns

A row represents one record.

A column represents one attribute.

```text
Entity:
Customer

Attributes:
id
email
full_name
phone_number
created_at
```

Avoid storing multiple unrelated values in one column.

Poor design:

```sql
contact_details TEXT
```

Better design:

```sql
email VARCHAR(255),
phone_number VARCHAR(30),
country_code CHAR(2)
```

Structured columns are easier to:

* Validate
* Search
* Index
* Sort
* Aggregate
* Join

---

## 5. Data Types

MySQL provides several data-type categories.

| Category             | Common Types                               |
| -------------------- | ------------------------------------------ |
| Integer              | `TINYINT`, `SMALLINT`, `INT`, `BIGINT`     |
| Decimal              | `DECIMAL`, `NUMERIC`                       |
| Floating point       | `FLOAT`, `DOUBLE`                          |
| Text                 | `CHAR`, `VARCHAR`, `TEXT`, `LONGTEXT`      |
| Boolean-like values  | `BOOLEAN`, commonly stored as `TINYINT(1)` |
| Date and time        | `DATE`, `TIME`, `DATETIME`, `TIMESTAMP`    |
| Binary               | `BINARY`, `VARBINARY`, `BLOB`              |
| Structured documents | `JSON`                                     |
| Enumerated values    | `ENUM`, `SET`                              |
| Spatial              | `POINT`, `GEOMETRY`, `POLYGON`             |

Example:

```sql
CREATE TABLE products (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    sku VARCHAR(100) NOT NULL UNIQUE,
    name VARCHAR(255) NOT NULL,
    price DECIMAL(12, 2) NOT NULL,
    available BOOLEAN NOT NULL DEFAULT TRUE,
    metadata JSON,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,

    CHECK (price >= 0)
);
```

Choose data types according to business meaning.

For monetary values, prefer:

```sql
price DECIMAL(12, 2)
```

instead of:

```sql
price DOUBLE
```

Floating-point values may introduce precision errors.

---

## 6. Storage Engines

MySQL supports pluggable storage engines.

The storage engine controls how table data is:

* Stored
* Indexed
* Locked
* Recovered
* Transacted

View available engines:

```sql
SHOW ENGINES;
```

### InnoDB

InnoDB is the standard storage engine for most production applications.

It provides:

* ACID transactions
* Row-level locking
* Foreign keys
* Crash recovery
* Multi-Version Concurrency Control
* Clustered indexes
* Buffer-pool caching

```sql
CREATE TABLE accounts (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    balance DECIMAL(18, 2) NOT NULL
) ENGINE = InnoDB;
```

### MyISAM

MyISAM is an older engine that does not provide full transactional behavior or foreign-key enforcement.

It is generally not appropriate for modern transactional systems.

### MEMORY

The MEMORY engine stores data primarily in memory.

It may be useful for temporary or specialized workloads, but data is not durable across server restarts.

For most backend systems:

```text
Use InnoDB unless there is a carefully evaluated reason not to.
```

---

## 7. Primary Keys

A primary key uniquely identifies each row.

```sql
CREATE TABLE customers (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(255) NOT NULL
);
```

A primary key provides:

* Uniqueness
* Non-null enforcement
* Fast row access
* A stable relationship target

### Auto-Incrementing Primary Key

```sql
id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY
```

Advantages:

* Compact
* Efficient
* Naturally ordered
* Good index locality

Disadvantages:

* Predictable
* Harder to generate independently across services
* May reveal approximate record volume

### UUID Primary Key

```sql
id BINARY(16) PRIMARY KEY
```

UUIDs support distributed ID generation but are larger and can create random index writes.

For UUID-based systems, compact binary storage and time-ordered UUIDs can improve performance.

---

## 8. InnoDB Clustered Index

In InnoDB, table rows are physically organized around the primary key.

```text
Primary Key B-Tree
        |
        ├── Row 1
        ├── Row 2
        └── Row 3
```

This is called a **clustered index**.

Secondary indexes store:

```text
Secondary Key
      +
Primary Key Value
```

Lookup through a secondary index may require:

1. Search the secondary index.
2. Retrieve the primary-key value.
3. Search the clustered primary-key index.
4. Return the row.

Primary-key design therefore affects:

* Table storage
* Secondary-index size
* Insert performance
* Page fragmentation
* Cache efficiency

Prefer primary keys that are:

* Small
* Stable
* Unique
* Mostly sequential when possible

---

## 9. Foreign Keys

Foreign keys enforce relationships between tables.

```sql
CREATE TABLE orders (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    customer_id BIGINT UNSIGNED NOT NULL,
    total_amount DECIMAL(12, 2) NOT NULL,

    CONSTRAINT fk_orders_customer
        FOREIGN KEY (customer_id)
        REFERENCES customers(id)
);
```

Relationship:

```text
customers
    |
    | 1
    |
    | N
    v
orders
```

Without a foreign key, an order could reference a customer that does not exist.

Foreign keys help protect the database from invalid relationships.

---

## 10. Constraints

Constraints enforce data rules.

### NOT NULL

```sql
email VARCHAR(255) NOT NULL
```

### UNIQUE

```sql
email VARCHAR(255) UNIQUE
```

### PRIMARY KEY

```sql
id BIGINT PRIMARY KEY
```

### FOREIGN KEY

```sql
FOREIGN KEY (customer_id) REFERENCES customers(id)
```

### CHECK

```sql
CHECK (quantity > 0)
```

Example:

```sql
CREATE TABLE bank_accounts (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    account_number VARCHAR(100) NOT NULL UNIQUE,
    balance DECIMAL(18, 2) NOT NULL,
    status VARCHAR(20) NOT NULL,

    CHECK (balance >= 0),
    CHECK (status IN ('active', 'blocked', 'closed'))
);
```

Application validation improves usability.

Database constraints protect correctness from every writer.

---

## 11. CRUD Operations

CRUD stands for:

* Create
* Read
* Update
* Delete

### Create

```sql
INSERT INTO users (
    email,
    full_name
)
VALUES (
    'alex@example.com',
    'Alex Smith'
);
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
SET full_name = 'Alexander Smith'
WHERE id = 1;
```

### Delete

```sql
DELETE FROM users
WHERE id = 1;
```

Always use a restrictive condition for update and delete operations.

Dangerous:

```sql
DELETE FROM users;
```

Safer:

```sql
DELETE FROM users
WHERE id = 1;
```

---

## 12. Transactions

A transaction groups multiple statements into one logical unit.

```sql
START TRANSACTION;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1;

UPDATE accounts
SET balance = balance + 100
WHERE id = 2;

COMMIT;
```

If something fails:

```sql
ROLLBACK;
```

Transaction flow:

```text
Begin
  |
  v
Debit Account A
  |
  v
Credit Account B
  |
  ├── Success → Commit
  |
  └── Failure → Rollback
```

Transactions are essential for:

* Money transfers
* Order creation
* Inventory reservation
* Subscription updates
* Payment state changes
* Multi-table writes

---

## 13. ACID Properties

MySQL with InnoDB supports ACID transactions.

### Atomicity

All operations in a transaction succeed or fail together.

### Consistency

Transactions preserve database constraints and business invariants.

### Isolation

Concurrent transactions should not interfere in unsafe ways.

### Durability

Committed changes survive crashes when persistence settings and storage behave correctly.

---

## 14. Transaction Isolation Levels

MySQL supports four standard isolation levels.

| Isolation Level  | Dirty Reads | Non-Repeatable Reads |                      Phantom Reads |
| ---------------- | ----------: | -------------------: | ---------------------------------: |
| Read Uncommitted |    Possible |             Possible |                           Possible |
| Read Committed   |   Prevented |             Possible |                           Possible |
| Repeatable Read  |   Prevented |            Prevented | Controlled through InnoDB behavior |
| Serializable     |   Prevented |            Prevented | Prevented through stronger locking |

InnoDB commonly uses `REPEATABLE READ` by default.

Set isolation level:

```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

START TRANSACTION;
```

The correct isolation level depends on:

* Required correctness
* Contention
* Transaction duration
* Retry handling
* Read patterns

Stronger isolation can reduce concurrency.

---

## 15. Multi-Version Concurrency Control

InnoDB uses Multi-Version Concurrency Control, or MVCC.

MVCC allows transactions to read consistent versions of data while other transactions are changing it.

```text
Transaction A reads Row Version 1

Transaction B updates the row
        |
        v
Creates Row Version 2

Transaction A may continue seeing Version 1
```

Benefits include:

* Readers often do not block writers.
* Writers often do not block consistent readers.
* Transactions receive stable snapshots.
* Concurrency improves.

Undo logs help InnoDB reconstruct earlier row versions.

---

## 16. Undo Logs

Undo logs store information needed to:

* Roll back transactions
* Provide earlier row versions
* Support MVCC

Conceptually:

```text
Original:
balance = 100

Update:
balance = 80

Undo Information:
restore balance = 100
```

Long-running transactions can prevent old undo information from being cleaned up efficiently.

This can increase storage pressure and degrade performance.

---

## 17. Redo Logs

Redo logs record changes required to recover committed transactions after a crash.

```text
Transaction Change
       |
       v
Redo Log
       |
       v
Commit
       |
       v
Data Page Written Later
```

Redo logging allows MySQL to confirm commits without immediately flushing every modified table page.

After a crash, InnoDB replays redo information to restore durable changes.

---

## 18. Binary Log

The binary log, commonly called the **binlog**, records database changes.

It supports:

* Replication
* Point-in-time recovery
* Change data capture
* Auditing workflows
* Data integration

```text
Primary MySQL
     |
     | Binary Log Events
     v
Replica MySQL
```

The redo log and binary log serve different purposes.

| Log        | Main Purpose                     |
| ---------- | -------------------------------- |
| Redo log   | InnoDB crash recovery            |
| Undo log   | Rollback and MVCC                |
| Binary log | Replication and logical recovery |

---

## 19. Doublewrite Buffer

InnoDB uses a doublewrite mechanism to protect against partial page writes.

Conceptually:

```text
Modified Page
     |
     v
Doublewrite Area
     |
     v
Final Data File
```

If a crash leaves a partially written page, InnoDB can use the additional copy during recovery.

This supports storage integrity but introduces additional write work.

---

## 20. Buffer Pool

The InnoDB buffer pool caches frequently accessed:

* Table pages
* Index pages
* Modified pages
* Adaptive structures

```text
Application Query
       |
       v
Buffer Pool
       |
       ├── Cache Hit → Return Data
       |
       └── Cache Miss → Read From Disk
```

The buffer pool is one of the most important MySQL performance components.

A high cache-hit ratio reduces storage reads.

The correct size depends on:

* Available memory
* Dataset size
* Other processes
* Workload type
* Managed database restrictions

---

## 21. Query Cache

Older MySQL versions had a global query cache.

It caused contention and was removed from modern MySQL releases.

Do not confuse:

```text
InnoDB Buffer Pool
```

with:

```text
Old MySQL Query Cache
```

Modern application caching commonly uses:

* Redis
* Memcached
* CDN caching
* Application memory
* Materialized result tables

---

## 22. Indexes

Indexes speed up reads at the cost of:

* Additional storage
* Slower inserts
* Slower updates
* Slower deletes
* More memory pressure
* Maintenance overhead

Create an index:

```sql
CREATE INDEX idx_orders_customer_id
ON orders(customer_id);
```

Query:

```sql
SELECT id, total_amount, status
FROM orders
WHERE customer_id = 100;
```

Without an index:

```text
Scan many table rows
```

With an appropriate index:

```text
Navigate to matching keys
```

---

## 23. B-Tree Indexes

InnoDB commonly uses B-tree indexes.

They work well for:

* Equality conditions
* Range filters
* Sorting
* Prefix matching
* Ordered scans

Examples:

```sql
WHERE email = 'alex@example.com'
```

```sql
WHERE created_at >= '2026-01-01'
```

```sql
ORDER BY created_at DESC
```

B-tree indexes generally cannot efficiently optimize a leading wildcard search:

```sql
WHERE name LIKE '%smith'
```

---

## 24. Composite Indexes

A composite index contains multiple columns.

```sql
CREATE INDEX idx_orders_customer_status_created
ON orders(customer_id, status, created_at DESC);
```

It may support:

```sql
WHERE customer_id = 100
```

```sql
WHERE customer_id = 100
  AND status = 'pending'
```

```sql
WHERE customer_id = 100
  AND status = 'pending'
ORDER BY created_at DESC
```

Column order matters.

The index may not be efficient for:

```sql
WHERE status = 'pending'
```

because `customer_id` is the leading column.

---

## 25. Leftmost Prefix Rule

A composite B-tree index is most effective when a query uses columns from the left side of the index definition.

Index:

```text
(customer_id, status, created_at)
```

Efficient prefixes:

```text
customer_id
customer_id + status
customer_id + status + created_at
```

Usually less effective:

```text
status only
created_at only
status + created_at
```

Design indexes around actual query filters and sort order.

---

## 26. Covering Indexes

A covering index contains all columns needed by a query.

```sql
CREATE INDEX idx_orders_customer_covering
ON orders(customer_id, created_at, status, total_amount);
```

Query:

```sql
SELECT created_at, status, total_amount
FROM orders
WHERE customer_id = 100;
```

MySQL may answer the query using only the secondary index without accessing the clustered row.

Execution plans may show:

```text
Using index
```

Covering indexes can improve performance but may become large.

---

## 27. Unique Indexes

A unique index enforces uniqueness.

```sql
CREATE UNIQUE INDEX idx_users_email
ON users(email);
```

Equivalent intent can be expressed inside a table definition:

```sql
email VARCHAR(255) NOT NULL UNIQUE
```

Unique constraints are essential for preventing race conditions.

Unsafe application-only approach:

```text
1. Check whether email exists
2. No row found
3. Insert email
```

Two concurrent requests may both pass the check.

A database uniqueness rule prevents duplicates safely.

---

## 28. Prefix Indexes

MySQL can index a prefix of a long string.

```sql
CREATE INDEX idx_articles_title_prefix
ON articles(title(50));
```

This reduces index size but may reduce selectivity.

Prefix indexes require careful measurement.

They are not appropriate when full-value uniqueness or ordering is required.

---

## 29. Functional Indexes

Modern MySQL versions can index expressions.

Example:

```sql
CREATE INDEX idx_users_lower_email
ON users ((LOWER(email)));
```

Query:

```sql
SELECT id, email
FROM users
WHERE LOWER(email) = 'alex@example.com';
```

Functional indexes can improve searches on normalized or computed values.

---

## 30. Invisible Indexes

An invisible index remains maintained but is ignored by the query optimizer by default.

```sql
ALTER TABLE orders
ALTER INDEX idx_orders_status INVISIBLE;
```

This helps test whether an index can be removed without immediately dropping it.

Restore visibility:

```sql
ALTER TABLE orders
ALTER INDEX idx_orders_status VISIBLE;
```

Invisible indexes are useful for safe index cleanup.

---

## 31. EXPLAIN

`EXPLAIN` shows how MySQL plans to execute a query.

```sql
EXPLAIN
SELECT id, status, total_amount
FROM orders
WHERE customer_id = 100;
```

Important fields commonly include:

* Access type
* Chosen key
* Estimated rows
* Filter percentage
* Extra operations

Common access types include:

| Access Type | Meaning                                     |
| ----------- | ------------------------------------------- |
| `const`     | At most one matching row                    |
| `eq_ref`    | One row per joined row using a unique key   |
| `ref`       | Index lookup with possible multiple matches |
| `range`     | Index range scan                            |
| `index`     | Full index scan                             |
| `ALL`       | Full table scan                             |

`EXPLAIN ANALYZE` provides actual execution information in supported versions.

```sql
EXPLAIN ANALYZE
SELECT id, status
FROM orders
WHERE customer_id = 100;
```

---

## 32. Query Optimizer

The MySQL optimizer chooses an execution plan based on:

* Available indexes
* Table statistics
* Estimated selectivity
* Join order
* Sorting requirements
* Temporary-table cost
* Access methods

```text
SQL Query
   |
   v
Parser
   |
   v
Optimizer
   |
   v
Execution Plan
   |
   v
Storage Engine
```

A valid query can still perform poorly because of:

* Missing indexes
* Incorrect index order
* Outdated statistics
* Non-sargable filters
* Large intermediate results
* Poor join order
* Unnecessary sorting

---

## 33. Sargable Queries

A sargable query allows the database to use an index efficiently.

Poor:

```sql
SELECT *
FROM orders
WHERE YEAR(created_at) = 2026;
```

The function is applied to every row value.

Better:

```sql
SELECT *
FROM orders
WHERE created_at >= '2026-01-01'
  AND created_at < '2027-01-01';
```

Poor:

```sql
WHERE LOWER(email) = 'alex@example.com'
```

Better options:

* Store a normalized email column.
* Use an appropriate case-insensitive collation.
* Create a functional index.

---

## 34. Joins

Joins combine related rows.

### INNER JOIN

Returns matching rows from both tables.

```sql
SELECT
    orders.id,
    customers.email
FROM orders
INNER JOIN customers
    ON customers.id = orders.customer_id;
```

### LEFT JOIN

Returns all rows from the left table.

```sql
SELECT
    customers.id,
    orders.id AS order_id
FROM customers
LEFT JOIN orders
    ON orders.customer_id = customers.id;
```

### CROSS JOIN

Returns combinations of all rows from both sides.

```sql
SELECT *
FROM sizes
CROSS JOIN colors;
```

MySQL does not directly provide a traditional `FULL OUTER JOIN`, but similar behavior can be constructed using unions when needed.

---

## 35. Normalization

Normalization reduces duplicated data and update anomalies.

Poor design:

```text
orders

order_id
customer_name
customer_email
product_name
product_price
quantity
```

Better design:

```text
customers
products
orders
order_items
```

Relationship:

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
* Constraint enforcement
* Update safety
* Storage efficiency

---

## 36. Denormalization

Denormalization intentionally duplicates data for performance or historical correctness.

Example:

```sql
CREATE TABLE order_items (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    order_id BIGINT UNSIGNED NOT NULL,
    product_id BIGINT UNSIGNED NOT NULL,
    product_name_snapshot VARCHAR(255) NOT NULL,
    unit_price_snapshot DECIMAL(12, 2) NOT NULL,
    quantity INT UNSIGNED NOT NULL
);
```

The snapshot preserves the original purchase information even when the product changes later.

Denormalization should be:

* Intentional
* Documented
* Consistently maintained
* Supported by reconciliation processes

---

## 37. JSON Data Type

MySQL supports native JSON storage and functions.

```sql
CREATE TABLE events (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    event_type VARCHAR(100) NOT NULL,
    payload JSON NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

Insert:

```sql
INSERT INTO events (
    event_type,
    payload
)
VALUES (
    'user_registered',
    JSON_OBJECT(
        'userId', 101,
        'source', 'mobile',
        'campaign', 'summer'
    )
);
```

Query:

```sql
SELECT
    id,
    JSON_UNQUOTE(JSON_EXTRACT(payload, '$.source')) AS source
FROM events
WHERE JSON_EXTRACT(payload, '$.userId') = 101;
```

Shorter operator syntax:

```sql
SELECT payload->>'$.source'
FROM events;
```

JSON is useful for flexible attributes, but core relational data should usually remain in typed columns.

---

## 38. Generated Columns

Generated columns derive values from expressions.

```sql
CREATE TABLE users (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    profile JSON NOT NULL,
    country_code CHAR(2)
        GENERATED ALWAYS AS (
            JSON_UNQUOTE(
                JSON_EXTRACT(profile, '$.countryCode')
            )
        ) STORED
);
```

The generated column can be indexed:

```sql
CREATE INDEX idx_users_country_code
ON users(country_code);
```

Generated columns are useful for:

* Indexing JSON attributes
* Normalized search values
* Repeated calculations
* Compatibility with older functional-index patterns

---

## 39. Views

A view stores a reusable SQL query.

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

Views can:

* Simplify complex queries
* Restrict exposed columns
* Standardize reporting logic
* Provide compatibility layers

A regular view does not normally store the result itself.

---

## 40. Common Table Expressions

A Common Table Expression defines a named temporary result inside a query.

```sql
WITH customer_totals AS (
    SELECT
        customer_id,
        SUM(total_amount) AS total_spent
    FROM orders
    WHERE status = 'paid'
    GROUP BY customer_id
)
SELECT customer_id, total_spent
FROM customer_totals
WHERE total_spent > 10000;
```

Recursive CTE:

```sql
WITH RECURSIVE category_tree AS (
    SELECT
        id,
        parent_id,
        name,
        1 AS depth
    FROM categories
    WHERE parent_id IS NULL

    UNION ALL

    SELECT
        c.id,
        c.parent_id,
        c.name,
        ct.depth + 1
    FROM categories c
    INNER JOIN category_tree ct
        ON c.parent_id = ct.id
)
SELECT *
FROM category_tree;
```

---

## 41. Window Functions

Window functions perform calculations across related rows without reducing the result to one row per group.

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

Running total:

```sql
SELECT
    created_at,
    total_amount,
    SUM(total_amount) OVER (
        ORDER BY created_at
    ) AS running_total
FROM orders;
```

---

## 42. Upsert

MySQL supports upserts using `ON DUPLICATE KEY UPDATE`.

```sql
INSERT INTO user_settings (
    user_id,
    theme
)
VALUES (
    101,
    'dark'
)
ON DUPLICATE KEY UPDATE
    theme = 'dark';
```

Upserts are useful for:

* Idempotent configuration writes
* Synchronization
* Aggregations
* Imports
* Cache-like tables

The conflict must be detected through a primary or unique key.

---

## 43. Row-Level Locking

InnoDB supports row-level locks.

```sql
START TRANSACTION;

SELECT available_quantity
FROM inventory
WHERE product_id = 100
FOR UPDATE;

UPDATE inventory
SET available_quantity = available_quantity - 1
WHERE product_id = 100;

COMMIT;
```

`FOR UPDATE` locks matching rows for modification.

This is useful for:

* Inventory reservation
* Seat booking
* Account balance updates
* Job claiming
* Stateful workflows

Queries should use indexes so MySQL locks the smallest practical range.

---

## 44. Gap Locks and Next-Key Locks

Under InnoDB’s `REPEATABLE READ` isolation, range queries may acquire locks that cover both existing records and gaps between records.

```text
Existing Keys:

10       20       30

Possible Locked Range:

[10 -------- 20)
```

These locks help prevent phantom changes but can increase contention.

A transaction such as:

```sql
SELECT *
FROM reservations
WHERE seat_number BETWEEN 100 AND 110
FOR UPDATE;
```

may affect inserts into the locked range.

Understanding gap locking is important when diagnosing unexpected blocking.

---

## 45. Optimistic Locking

Optimistic locking detects concurrent updates using a version column.

```sql
CREATE TABLE documents (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    content TEXT NOT NULL,
    version INT UNSIGNED NOT NULL DEFAULT 1
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

If zero rows are affected, another transaction already modified the document.

Optimistic locking is useful when conflicts are uncommon.

---

## 46. Deadlocks

A deadlock occurs when transactions wait for each other.

```text
Transaction A locks Order 1
Transaction B locks Order 2

Transaction A requests Order 2
Transaction B requests Order 1
```

InnoDB detects the deadlock and rolls back one transaction.

Reduce deadlocks by:

* Locking rows in a consistent order
* Keeping transactions short
* Indexing lock queries
* Avoiding unnecessary range locks
* Retrying deadlock victims safely

Deadlocks are not always bugs; they are an expected possibility in concurrent systems.

---

## 47. Character Sets and Collations

A character set defines how text is encoded.

A collation defines how text is compared and sorted.

Prefer full Unicode support:

```sql
CHARACTER SET utf8mb4
```

Example:

```sql
CREATE TABLE messages (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    body TEXT NOT NULL
) CHARACTER SET utf8mb4
  COLLATE utf8mb4_0900_ai_ci;
```

Collation affects:

* Case sensitivity
* Accent sensitivity
* Sorting
* Unique constraints
* Index comparisons

Do not use MySQL’s older `utf8` alias when full Unicode character support is required.

---

## 48. Pagination

### Offset Pagination

```sql
SELECT id, created_at, status
FROM orders
ORDER BY created_at DESC, id DESC
LIMIT 20 OFFSET 10000;
```

Advantages:

* Simple
* Supports arbitrary page numbers

Disadvantages:

* Slower for deep pages
* Can skip or duplicate rows during concurrent changes
* Database still scans or skips earlier rows

### Keyset Pagination

```sql
SELECT id, created_at, status
FROM orders
WHERE
    created_at < '2026-07-30 10:00:00'
    OR (
        created_at = '2026-07-30 10:00:00'
        AND id < 50000
    )
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

Advantages:

* Fast for deep navigation
* Stable with large datasets
* Works efficiently with a matching index

---

## 49. Table Partitioning

Partitioning divides a large logical table into smaller partitions.

```text
events
   |
   ├── events_2026_01
   ├── events_2026_02
   ├── events_2026_03
   └── events_2026_04
```

Common strategies include:

* Range partitioning
* List partitioning
* Hash partitioning
* Key partitioning

Example:

```sql
CREATE TABLE events (
    id BIGINT UNSIGNED NOT NULL,
    created_at DATETIME NOT NULL,
    payload JSON NOT NULL,
    PRIMARY KEY (id, created_at)
)
PARTITION BY RANGE COLUMNS(created_at) (
    PARTITION p2026_01
        VALUES LESS THAN ('2026-02-01'),
    PARTITION p2026_02
        VALUES LESS THAN ('2026-03-01'),
    PARTITION pmax
        VALUES LESS THAN (MAXVALUE)
);
```

Partitioning can help with:

* Retention management
* Large append-only tables
* Partition pruning
* Bulk deletion
* Maintenance isolation

It should not be used as a replacement for correct indexing.

---

## 50. Replication

MySQL replication copies changes from a source server to replica servers.

```text
Primary
   |
   | Binary Log
   v
Replica
```

Replication supports:

* Read scaling
* High availability
* Reporting
* Disaster recovery
* Backup operations
* Geographic distribution

---

## 51. Replication Process

A simplified replication flow:

```text
Primary Transaction
        |
        v
Binary Log
        |
        v
Replica I/O Thread
        |
        v
Relay Log
        |
        v
Replica SQL Thread
        |
        v
Replica Data
```

Modern replication can apply changes in parallel depending on configuration and workload.

---

## 52. Asynchronous Replication

In asynchronous replication, the primary does not wait for replicas before confirming a commit.

```text
Client
   |
   v
Primary Commits
   |
   v
Client Receives Success
   |
   v
Replica Receives Change Later
```

Advantages:

* Low primary write latency
* Primary remains independent of replica speed

Disadvantages:

* Replica lag
* Potential loss of recent committed data during failover

---

## 53. Semi-Synchronous Replication

Semi-synchronous replication waits for confirmation that at least one replica has received the transaction event.

```text
Primary
   |
   | Send Replication Event
   v
Replica Receives Event
   |
   | Acknowledge
   v
Primary Confirms Commit
```

This reduces the risk of losing committed transactions but adds network latency.

Receiving the event does not always mean the replica has fully applied it.

---

## 54. Replication Formats

MySQL supports different binary-log formats.

### Statement-Based Replication

Records SQL statements.

```text
UPDATE inventory
SET quantity = quantity - 1
WHERE product_id = 100;
```

Advantages:

* Compact for some operations

Risks:

* Non-deterministic statements
* Different execution effects

### Row-Based Replication

Records changed row values.

Advantages:

* More deterministic
* Safer for many complex statements

Disadvantages:

* Larger log volume for bulk changes

### Mixed Replication

Chooses statement or row format based on the operation.

Row-based replication is common for production reliability.

---

## 55. Global Transaction Identifiers

Global Transaction Identifiers, or GTIDs, uniquely identify replicated transactions.

Conceptually:

```text
source-id:transaction-number
```

GTIDs simplify:

* Replica promotion
* Failover
* Re-parenting replicas
* Replication recovery
* Determining executed transactions

GTID-based replication is commonly preferred for modern high-availability deployments.

---

## 56. Replication Lag

Replication lag is the delay between a primary commit and replica application.

```text
Primary:
Order 9001 = paid

Replica:
Order 9001 = pending
```

This creates eventual consistency for replica reads.

Suitable replica reads:

* Product catalogs
* Analytics
* Reports
* Historical lists
* Public content

Primary reads may be required for:

* Payment confirmation
* Read-after-write requests
* Inventory decisions
* Security changes
* Transactional workflows

---

## 57. Read Replicas

Read replicas distribute read traffic.

```text
                       ┌── Replica A
Application ──Router───┼── Replica B
                       └── Replica C

Writes ────────────────> Primary
```

Read replicas help when:

* The workload is read-heavy.
* Queries can tolerate replica lag.
* Reporting should not overload the primary.
* Geographic read latency needs improvement.

They do not increase primary write capacity.

---

## 58. High Availability

A highly available MySQL architecture often includes:

* One writable primary
* Multiple replicas
* Failure detection
* Automated promotion
* Proxy or connection router
* Fencing
* Backups
* Monitoring

```text
                    Applications
                         |
                         v
                MySQL Proxy / Router
                         |
             ┌───────────┴───────────┐
             │                       │
          Primary                 Replica
             │                       │
             └──── Replication ──────┘
```

During failure:

```text
Primary Fails
     |
     v
Select Healthy Replica
     |
     v
Promote Replica
     |
     v
Update Traffic Routing
```

Failover systems must prevent split brain.

---

## 59. InnoDB Cluster

InnoDB Cluster combines technologies such as:

* Group Replication
* MySQL Router
* MySQL Shell
* Automated topology management

Conceptual architecture:

```text
                  Applications
                       |
                       v
                  MySQL Router
                       |
         ┌─────────────┼─────────────┐
         │             │             │
      Node A        Node B        Node C
      Primary       Replica       Replica
```

Group Replication coordinates membership and transaction replication.

This can simplify high availability, but applications must still handle:

* Temporary failures
* Transaction retries
* Connection interruption
* Failover latency

---

## 60. Connection Pooling

Opening a new database connection for every request is expensive.

```text
1,000 Requests
      |
      v
Connection Pool
      |
      v
50 MySQL Connections
```

Connection pooling reduces:

* Authentication overhead
* TLS negotiation
* Socket creation
* Database thread churn
* Connection storms

Configure:

* Maximum pool size
* Minimum idle connections
* Acquisition timeout
* Idle timeout
* Connection lifetime
* Health validation

Do not size the pool only from application traffic.

Size it according to database capacity and query duration.

---

## 61. Thread-Per-Connection Model

MySQL commonly associates client connections with server threads.

Too many active connections may increase:

* Memory usage
* Context switching
* Lock contention
* Scheduler overhead
* Failure amplification

More connections do not guarantee more throughput.

A bounded connection pool creates backpressure and protects the database.

---

## 62. Stored Procedures

Stored procedures execute logic inside MySQL.

```sql
DELIMITER //

CREATE PROCEDURE transfer_funds(
    IN source_account BIGINT,
    IN destination_account BIGINT,
    IN transfer_amount DECIMAL(18, 2)
)
BEGIN
    START TRANSACTION;

    UPDATE accounts
    SET balance = balance - transfer_amount
    WHERE id = source_account;

    UPDATE accounts
    SET balance = balance + transfer_amount
    WHERE id = destination_account;

    COMMIT;
END //

DELIMITER ;
```

Stored procedures may reduce network round trips but can create:

* Database coupling
* Harder testing
* Complex deployments
* Vendor-specific logic
* Difficult observability

Use them selectively.

---

## 63. Triggers

Triggers run automatically after or before database events.

```sql
CREATE TRIGGER before_user_update
BEFORE UPDATE ON users
FOR EACH ROW
SET NEW.updated_at = CURRENT_TIMESTAMP;
```

Triggers can support:

* Audit fields
* Validation
* Derived values
* Legacy integration

However, hidden side effects can make debugging difficult.

Prefer explicit application logic unless database-level automation provides clear value.

---

## 64. Events

MySQL Event Scheduler can run scheduled SQL tasks.

```sql
CREATE EVENT delete_expired_sessions
ON SCHEDULE EVERY 1 HOUR
DO
    DELETE FROM sessions
    WHERE expires_at < CURRENT_TIMESTAMP;
```

Scheduled events may be useful for lightweight maintenance.

For complex workflows, use an external job system with:

* Monitoring
* Retries
* Ownership
* Alerting
* Execution history

---

# Architecture

## MySQL Internal Architecture

```text
                         Client Applications
                                  |
                                  v
                       Connection Management
                                  |
                                  v
                      Authentication and SQL Layer
                                  |
                ┌─────────────────┴─────────────────┐
                │                                   │
           Query Parser                        Query Optimizer
                │                                   │
                └─────────────────┬─────────────────┘
                                  |
                                  v
                           Query Executor
                                  |
                                  v
                    Storage Engine Interface
                                  |
                                  v
                             InnoDB
                ┌─────────────────┼─────────────────┐
                │                 │                 │
          Buffer Pool         Redo Log          Undo Log
                │                 │                 │
                └─────────────────┼─────────────────┘
                                  |
                                  v
                        Tablespaces and Indexes
```

---

## Query Execution Flow

```text
SQL Request
    |
    v
Connection Validation
    |
    v
Parser
    |
    v
Semantic Analysis
    |
    v
Optimizer
    |
    v
Execution Plan
    |
    v
Storage Engine
    |
    v
Rows Returned
```

### Parser

Validates SQL syntax and creates an internal representation.

### Optimizer

Chooses:

* Indexes
* Join order
* Access methods
* Sorting strategies
* Temporary-table behavior

### Executor

Runs the chosen plan through the storage-engine interface.

### InnoDB

Retrieves or modifies table and index pages while handling transactions and locks.

---

## Basic Production Architecture

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
                              |
                       Connection Pool
                              |
                              v
                       MySQL Primary
                              |
                  ┌───────────┴───────────┐
                  │                       │
            Read Replica             Backup System
                  │                       │
                  v                       v
          Reporting Traffic       Point-in-Time Recovery
```

---

## Highly Available Architecture

```text
                           Applications
                                |
                                v
                       Proxy / MySQL Router
                                |
             ┌──────────────────┼──────────────────┐
             │                  │                  │
       Primary Node        Replica Node A     Replica Node B
             │                  ▲                  ▲
             └──── Binary Log ──┴──────────────────┘
                                |
                                v
                         Backup Storage
```

Responsibilities:

### Proxy or Router

* Routes writes to the primary
* Routes eligible reads to replicas
* Detects topology changes
* Supports connection failover

### Primary

* Accepts writes
* Produces binary logs
* Maintains transactional consistency

### Replicas

* Apply primary changes
* Serve stale-tolerant reads
* Support failover and reporting

### Backup Storage

* Protects against logical deletion
* Supports point-in-time recovery
* Remains independent from replication

---

## Multi-Region Architecture

```text
                         Global Application
                                 |
                ┌────────────────┴────────────────┐
                │                                 │
            Region A                          Region B
                │                                 │
       Application Cluster              Application Cluster
                │                                 │
                v                                 v
          MySQL Primary ─── Async Replication ─> Replica
                |
                v
          Backup Storage
```

Cross-region replication is commonly asynchronous because synchronous commits across long distances increase write latency.

Trade-offs:

* Lower disaster-recovery time
* Replica lag
* Possible recent-data loss
* Complex failover
* Cross-region networking cost
* Data-residency requirements

---

## Read and Write Routing

```text
                       Application
                            |
               ┌────────────┴────────────┐
               │                         │
             Writes                    Reads
               │                         │
               v                         v
             Primary               Read Replicas
```

Read from the primary when:

* Read-after-write consistency is required.
* A payment result must be current.
* Inventory correctness is critical.
* The read is part of a write transaction.

Read from a replica when:

* Small delays are acceptable.
* The query is analytical.
* The data changes infrequently.
* Reporting must not overload the primary.

---

## Cache-Aside Architecture

```text
                       Application
                            |
                ┌───────────┴───────────┐
                │                       │
                v                       v
              Redis                   MySQL
              Cache               Source of Truth
```

Read flow:

```text
1. Check Redis
2. Cache hit → return value
3. Cache miss → query MySQL
4. Store result in Redis
5. Return value
```

Write flow:

```text
1. Update MySQL
2. Invalidate or update cache
3. Return success
```

Caching improves read scalability but introduces:

* Stale data
* Invalidation complexity
* Hot keys
* Cache stampedes
* Additional operational dependencies

---

## Sharded Architecture

When one primary cannot hold or process the entire workload, data may be divided into shards.

```text
                         Application
                              |
                              v
                         Shard Router
               ┌──────────────┼──────────────┐
               │              │              │
          Shard A         Shard B         Shard C
       Users 1–1M      Users 1M–2M     Users 2M–3M
```

Possible shard keys include:

* Customer ID
* Tenant ID
* Geographic region
* Account ID

Sharding introduces:

* Routing logic
* Cross-shard queries
* Distributed transactions
* Rebalancing
* Uneven shard growth
* More difficult schema changes
* Per-shard failover

Use sharding only after simpler approaches are insufficient.

---

# Comparisons

## MySQL vs PostgreSQL

| Category             | MySQL                                       | PostgreSQL                                     |
| -------------------- | ------------------------------------------- | ---------------------------------------------- |
| Primary model        | Relational                                  | Relational and object-relational               |
| Common strength      | Straightforward transactional web workloads | Advanced SQL and extensibility                 |
| Storage architecture | Pluggable engines, commonly InnoDB          | Integrated storage architecture                |
| JSON                 | Strong native support                       | Strong JSON and JSONB support                  |
| Advanced indexing    | Good                                        | Very extensive                                 |
| Extensions           | More limited                                | Rich extension ecosystem                       |
| Geospatial           | Supported                                   | Especially strong with PostGIS                 |
| Replication          | Mature and widely used                      | Mature physical and logical replication        |
| Default isolation    | Commonly Repeatable Read                    | Commonly Read Committed                        |
| Typical use          | Web, SaaS, commerce, transactional systems  | Complex relational and analytical applications |

Choose based on:

* Team expertise
* Query complexity
* Extension requirements
* Operational environment
* Existing ecosystem
* Scaling strategy

Both are excellent production databases.

---

## MySQL vs MongoDB

| Category       | MySQL                         | MongoDB                                 |
| -------------- | ----------------------------- | --------------------------------------- |
| Data model     | Relational tables             | JSON-like documents                     |
| Schema         | Explicit                      | Flexible                                |
| Relationships  | Foreign keys and joins        | Embedded documents and references       |
| Transactions   | Mature ACID support           | Supported                               |
| Query language | SQL                           | Document-query API                      |
| Constraints    | Strong database constraints   | Often more application-driven           |
| Best fit       | Structured transactional data | Document-centered models                |
| Ad hoc queries | Strong SQL                    | Strong document queries and aggregation |

Choose MySQL when:

* Relationships are important.
* Cross-row transactions are common.
* Constraints must protect data.
* SQL reporting is required.

Choose MongoDB when:

* Data naturally forms self-contained documents.
* Schema flexibility is a primary requirement.
* Cross-document relationships are limited.

---

## MySQL vs Redis

| Category      | MySQL                        | Redis                            |
| ------------- | ---------------------------- | -------------------------------- |
| Main role     | Durable system of record     | In-memory data platform          |
| Storage       | Disk-backed                  | Primarily memory-oriented        |
| Query model   | SQL                          | Commands and data structures     |
| Transactions  | Full relational transactions | Limited transaction semantics    |
| Latency       | Usually milliseconds         | Often sub-millisecond            |
| Relationships | Joins and foreign keys       | Application-managed              |
| Best use      | Business data                | Cache, sessions, counters, locks |

Common architecture:

```text
Application
    |
    ├── Redis for fast temporary access
    |
    └── MySQL for durable records
```

Redis should not automatically replace MySQL as the source of truth.

---

## MySQL vs Elasticsearch

| Category         | MySQL                      | Elasticsearch                     |
| ---------------- | -------------------------- | --------------------------------- |
| Primary role     | Transactional database     | Search and analytics              |
| Data correctness | Strong ACID semantics      | Search-oriented distributed model |
| Full-text search | Available                  | Advanced                          |
| Joins            | Strong relational joins    | Limited                           |
| Aggregations     | SQL aggregation            | Distributed search aggregation    |
| Source of truth  | Yes                        | Usually no                        |
| Best use         | Orders, accounts, payments | Search, logs, text retrieval      |

Common pattern:

```text
MySQL
   |
   | Change Events
   v
Message Broker
   |
   v
Elasticsearch
```

MySQL remains authoritative while Elasticsearch provides search.

---

## MySQL vs SQLite

| Category          | MySQL                | SQLite                            |
| ----------------- | -------------------- | --------------------------------- |
| Deployment        | Client-server        | Embedded file                     |
| Network access    | Native               | Not a network server              |
| Concurrent writes | Strong support       | More limited                      |
| Administration    | Required             | Minimal                           |
| Replication       | Supported            | Not built in                      |
| Best use          | Backend applications | Local apps, tests, embedded tools |

SQLite is excellent for local storage.

MySQL is better for shared, multi-user backend services.

---

## MySQL vs Cassandra

| Category          | MySQL                                  | Cassandra                           |
| ----------------- | -------------------------------------- | ----------------------------------- |
| Data model        | Relational                             | Wide-column                         |
| Consistency       | Strong transactions                    | Tunable consistency                 |
| Joins             | Supported                              | Not supported                       |
| Query flexibility | High                                   | Query-driven schema                 |
| Horizontal writes | Requires sharding or distributed layer | Designed for distributed writes     |
| Transactions      | Rich                                   | Limited across partitions           |
| Best use          | Business transactions                  | Massive distributed write workloads |

Choose Cassandra when:

* Multi-region write availability is critical.
* Queries are highly predictable.
* Denormalized tables are acceptable.
* Huge write throughput is required.

Choose MySQL when:

* Relationships and transactions dominate.
* Strong constraints are needed.
* Flexible SQL queries matter.

---

## Replication vs Sharding

| Category                | Replication                   | Sharding                        |
| ----------------------- | ----------------------------- | ------------------------------- |
| Main purpose            | Availability and read scaling | Data and write scaling          |
| Data placement          | Copies of the same dataset    | Different subsets of data       |
| Write capacity          | Usually one primary           | Multiple shard writers          |
| Read scaling            | Strong                        | Strong within each shard        |
| Complexity              | Moderate                      | High                            |
| Cross-node transactions | Usually unnecessary           | Difficult                       |
| Failover                | Replica promotion             | Per-shard failover              |
| Best use                | Most growing systems          | Workloads exceeding one primary |

Use replication before sharding when the primary limitation is read traffic.

---

## MySQL vs Distributed SQL

| Category               | MySQL                              | Distributed SQL                        |
| ---------------------- | ---------------------------------- | -------------------------------------- |
| Topology               | Usually primary and replicas       | Multiple coordinated nodes             |
| Horizontal writes      | Requires sharding or extensions    | Often built in                         |
| Transactions           | Local database transactions        | Distributed transactions               |
| Operational complexity | Familiar and mature                | Higher coordination complexity         |
| Geographic writes      | Difficult                          | Often supported                        |
| Latency                | Excellent within one region        | Coordination may add latency           |
| Best use               | Conventional transactional systems | Horizontally distributed SQL workloads |

Distributed SQL can simplify some sharding problems, but it introduces new trade-offs involving consensus, latency, and operations.

---

# Real-World Example: E-Commerce Order System

Consider an e-commerce platform with:

* Customers
* Products
* Inventory
* Shopping carts
* Orders
* Payments
* Read replicas
* Transactional event publishing
* Reporting

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

Schema:

```sql
CREATE TABLE customers (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    full_name VARCHAR(150) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE = InnoDB;

CREATE TABLE products (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    sku VARCHAR(100) NOT NULL UNIQUE,
    name VARCHAR(255) NOT NULL,
    current_price DECIMAL(12, 2) NOT NULL,
    active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,

    CHECK (current_price >= 0)
) ENGINE = InnoDB;

CREATE TABLE inventory (
    product_id BIGINT UNSIGNED PRIMARY KEY,
    available_quantity INT UNSIGNED NOT NULL,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
        ON UPDATE CURRENT_TIMESTAMP,

    CONSTRAINT fk_inventory_product
        FOREIGN KEY (product_id)
        REFERENCES products(id)
) ENGINE = InnoDB;

CREATE TABLE orders (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    customer_id BIGINT UNSIGNED NOT NULL,
    status VARCHAR(30) NOT NULL,
    total_amount DECIMAL(12, 2) NOT NULL,
    idempotency_key VARCHAR(100) NOT NULL UNIQUE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
        ON UPDATE CURRENT_TIMESTAMP,

    CONSTRAINT fk_orders_customer
        FOREIGN KEY (customer_id)
        REFERENCES customers(id),

    CHECK (
        status IN (
            'pending',
            'confirmed',
            'paid',
            'shipped',
            'cancelled'
        )
    ),

    CHECK (total_amount >= 0)
) ENGINE = InnoDB;

CREATE TABLE order_items (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    order_id BIGINT UNSIGNED NOT NULL,
    product_id BIGINT UNSIGNED NOT NULL,
    quantity INT UNSIGNED NOT NULL,
    unit_price DECIMAL(12, 2) NOT NULL,

    CONSTRAINT fk_order_items_order
        FOREIGN KEY (order_id)
        REFERENCES orders(id)
        ON DELETE CASCADE,

    CONSTRAINT fk_order_items_product
        FOREIGN KEY (product_id)
        REFERENCES products(id),

    CONSTRAINT uq_order_product
        UNIQUE (order_id, product_id),

    CHECK (quantity > 0),
    CHECK (unit_price >= 0)
) ENGINE = InnoDB;
```

---

## Production Architecture

```text
                             Customer
                                |
                                v
                           API Gateway
                                |
                                v
                          Order Service
                                |
                                v
                         Connection Pool
                                |
                                v
                          MySQL Primary
                        /       |        \
                       /        |         \
              Read Replica   Binlog     Backup Storage
                   |            |             |
                   v            v             v
              Reporting    Event Pipeline   Recovery
```

Supporting event flow:

```text
MySQL Outbox Table
        |
        v
Outbox Publisher
        |
        v
Message Broker
        |
        ├── Payment Service
        ├── Shipping Service
        ├── Email Service
        └── Analytics Service
```

---

## Order Placement Flow

The system must:

1. Validate the request.
2. prevent duplicate checkout.
3. lock or atomically update inventory.
4. create the order.
5. create order items.
6. store an event.
7. commit everything together.

```text
Client
   |
   v
Order Service
   |
   v
Begin Transaction
   |
   ├── Reserve Inventory
   ├── Create Order
   ├── Create Items
   └── Create Outbox Event
   |
   v
Commit
```

---

## Preventing Overselling

Suppose one product has one unit remaining.

Without concurrency protection:

```text
Customer A reads quantity = 1
Customer B reads quantity = 1

Customer A purchases
Customer B purchases

Final result:
Two purchases for one item
```

Use an atomic update:

```sql
UPDATE inventory
SET available_quantity = available_quantity - 1
WHERE product_id = 101
  AND available_quantity >= 1;
```

Application logic:

```text
Affected rows = 1
    |
    v
Inventory reserved

Affected rows = 0
    |
    v
Out of stock
```

This is often simpler than reading the quantity first.

---

## Complete Order Transaction

```sql
START TRANSACTION;

UPDATE inventory
SET available_quantity = available_quantity - 2
WHERE product_id = 101
  AND available_quantity >= 2;
```

The application must verify that one row was affected.

Create the order:

```sql
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
);
```

Retrieve the generated ID from the driver or session:

```sql
SET @order_id = LAST_INSERT_ID();
```

Create the item:

```sql
INSERT INTO order_items (
    order_id,
    product_id,
    quantity,
    unit_price
)
VALUES (
    @order_id,
    101,
    2,
    99.99
);
```

Create an outbox event:

```sql
INSERT INTO outbox_events (
    aggregate_type,
    aggregate_id,
    event_type,
    payload
)
VALUES (
    'order',
    CAST(@order_id AS CHAR),
    'order_confirmed',
    JSON_OBJECT(
        'orderId', @order_id,
        'customerId', 5001,
        'totalAmount', 199.98
    )
);

COMMIT;
```

If any step fails:

```sql
ROLLBACK;
```

---

## Idempotent Checkout

A client may retry because it did not receive a response.

```text
Client Request
      |
      v
Order Committed
      |
      v
Response Lost
      |
      v
Client Retries
```

Without idempotency:

```text
Duplicate order
Duplicate inventory deduction
Duplicate payment attempt
```

Use:

```sql
idempotency_key VARCHAR(100) NOT NULL UNIQUE
```

On retry:

```sql
SELECT
    id,
    status,
    total_amount
FROM orders
WHERE idempotency_key = 'checkout-request-abc123';
```

The service can return the original result.

---

## Transactional Outbox Pattern

Publishing directly after a commit creates a failure window.

```text
Order Transaction Commits
        |
        v
Application Crashes
        |
        v
Event Is Never Published
```

The outbox pattern writes the business data and event in the same MySQL transaction.

```sql
CREATE TABLE outbox_events (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    aggregate_type VARCHAR(100) NOT NULL,
    aggregate_id VARCHAR(100) NOT NULL,
    event_type VARCHAR(100) NOT NULL,
    payload JSON NOT NULL,
    published_at TIMESTAMP NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE = InnoDB;
```

Index pending events:

```sql
CREATE INDEX idx_outbox_pending
ON outbox_events(published_at, created_at);
```

Workers claim rows:

```sql
START TRANSACTION;

SELECT
    id,
    event_type,
    payload
FROM outbox_events
WHERE published_at IS NULL
ORDER BY id
LIMIT 100
FOR UPDATE SKIP LOCKED;
```

Multiple workers can process different rows concurrently.

After successful publication:

```sql
UPDATE outbox_events
SET published_at = CURRENT_TIMESTAMP
WHERE id IN (...);

COMMIT;
```

Consumers should still be idempotent because message delivery may occur more than once.

---

## Read Replica Usage

Suitable replica queries:

```text
Product catalog
Order history
Business reports
Analytics dashboards
Search-support jobs
```

Primary-only query:

```text
Customer completes payment
        |
        v
Immediately requests payment state
        |
        v
Read from primary
```

The replica may still show the previous state.

---

## Recommended Indexes

```sql
CREATE INDEX idx_orders_customer_created
ON orders(customer_id, created_at DESC);

CREATE INDEX idx_orders_status_created
ON orders(status, created_at);

CREATE INDEX idx_order_items_order
ON order_items(order_id);

CREATE INDEX idx_outbox_pending_created
ON outbox_events(published_at, created_at);
```

Each index supports a specific query pattern.

Example:

```sql
SELECT
    id,
    status,
    total_amount,
    created_at
FROM orders
WHERE customer_id = 5001
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

Potential matching index:

```sql
CREATE INDEX idx_orders_customer_created_id
ON orders(customer_id, created_at DESC, id DESC);
```

---

## Reporting Query

```sql
SELECT
    DATE(created_at) AS order_date,
    COUNT(*) AS order_count,
    SUM(total_amount) AS revenue
FROM orders
WHERE status IN ('paid', 'shipped')
  AND created_at >= CURRENT_TIMESTAMP - INTERVAL 30 DAY
GROUP BY DATE(created_at)
ORDER BY order_date;
```

For expensive reports, consider:

* Read replicas
* Summary tables
* Scheduled aggregation
* Data warehouses
* Change-data-capture pipelines

Do not allow analytical workloads to destabilize checkout traffic.

---

# Best Practices

## 1. Use InnoDB for Transactional Workloads

InnoDB provides:

* Transactions
* Row-level locks
* Crash recovery
* Foreign keys
* MVCC

Specify it explicitly when consistency across environments matters:

```sql
CREATE TABLE orders (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY
) ENGINE = InnoDB;
```

---

## 2. Use `utf8mb4`

Configure full Unicode support.

```sql
CREATE DATABASE ecommerce
CHARACTER SET utf8mb4
COLLATE utf8mb4_0900_ai_ci;
```

This supports:

* International characters
* Emoji
* Modern Unicode text

Verify that the application connection also uses the correct character set.

---

## 3. Model Business Rules With Constraints

Use the database to enforce critical invariants.

```sql
email VARCHAR(255) NOT NULL UNIQUE
```

```sql
quantity INT UNSIGNED NOT NULL
```

```sql
FOREIGN KEY (customer_id) REFERENCES customers(id)
```

Application validation alone is vulnerable to race conditions and alternative writers.

---

## 4. Keep Transactions Short

Long transactions can:

* Hold locks
* Increase undo-log growth
* Delay cleanup
* Increase deadlock risk
* Consume connections
* Increase replication lag

Avoid this pattern:

```text
Begin Transaction
      |
      v
Call External Payment API
      |
      v
Wait Several Seconds
      |
      v
Update MySQL
      |
      v
Commit
```

Keep external network calls outside database transactions.

---

## 5. Use Atomic Updates

Avoid separate read-then-write logic when one SQL statement can enforce the condition.

Poor:

```text
Read quantity
Check quantity
Update quantity
```

Better:

```sql
UPDATE inventory
SET available_quantity = available_quantity - 1
WHERE product_id = 100
  AND available_quantity >= 1;
```

Atomic statements reduce race conditions.

---

## 6. Add Indexes for Real Queries

Start with query patterns, not individual columns.

Query:

```sql
SELECT id, status, created_at
FROM orders
WHERE customer_id = ?
ORDER BY created_at DESC
LIMIT 20;
```

Index:

```sql
CREATE INDEX idx_orders_customer_created
ON orders(customer_id, created_at DESC);
```

Every index should have a documented purpose.

---

## 7. Respect Composite Index Order

Design composite indexes according to:

1. Equality filters
2. Range filters
3. Sorting
4. Selected columns where covering is valuable

Example:

```sql
WHERE tenant_id = ?
  AND status = ?
  AND created_at >= ?
ORDER BY created_at DESC
```

Possible index:

```sql
CREATE INDEX idx_orders_tenant_status_created
ON orders(tenant_id, status, created_at DESC);
```

---

## 8. Use `EXPLAIN ANALYZE`

Do not guess how a query runs.

```sql
EXPLAIN ANALYZE
SELECT id, status
FROM orders
WHERE customer_id = 100;
```

Look for:

* Full table scans
* Incorrect row estimates
* Filesorts
* Temporary tables
* Expensive nested loops
* Large numbers of examined rows
* Poor index choices

Test with production-like data volume.

---

## 9. Avoid `SELECT *`

Poor:

```sql
SELECT *
FROM customers;
```

Better:

```sql
SELECT id, email, full_name, status
FROM customers;
```

Selecting fewer columns:

* Reduces network traffic
* Reduces memory use
* May enable covering indexes
* Creates clearer API contracts
* Avoids exposing sensitive columns

---

## 10. Use Keyset Pagination at Scale

Avoid deep offsets:

```sql
LIMIT 20 OFFSET 500000;
```

Prefer a cursor using stable columns:

```sql
WHERE
    created_at < ?
    OR (created_at = ? AND id < ?)
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

Use a matching composite index.

---

## 11. Use Connection Pools

Each service instance should have bounded database connections.

Configure:

* Maximum pool size
* Acquisition timeout
* Idle timeout
* Connection lifetime
* Health checks
* Retry limits

Ensure:

```text
Application Instances × Pool Size
```

does not overwhelm MySQL.

---

## 12. Set Timeouts

Configure limits for:

* Connection establishment
* Connection acquisition
* Query execution
* Lock waiting
* Idle transactions
* Application requests

A database query should not continue indefinitely after the client request has already timed out.

---

## 13. Use Idempotency Keys

Critical operations may be retried.

Examples:

* Checkout
* Payment initiation
* Refund creation
* Subscription changes
* Webhook processing

Store a unique request identifier:

```sql
idempotency_key VARCHAR(100) NOT NULL UNIQUE
```

Return the existing result for duplicate requests.

---

## 14. Separate Read and Write Workloads Carefully

Use replicas for stale-tolerant reads.

Do not route correctness-sensitive reads blindly.

Strategies include:

* Primary reads after writes
* Session stickiness
* Lag-aware routing
* Consistency tokens
* Endpoint-specific routing

---

## 15. Monitor Replication Lag

Track:

* Seconds behind source
* Relay-log growth
* Apply-thread status
* Network delays
* Disk performance
* Long-running replica queries

Replica lag can silently produce incorrect user experiences.

---

## 16. Treat Replication and Backups Separately

Replication supports availability.

Backups support recovery from:

* Accidental deletion
* Corruption
* Bad migrations
* Security incidents
* Application bugs

Use:

* Full backups
* Incremental backups where supported
* Binary-log retention
* Point-in-time recovery
* Off-site storage
* Restore testing

---

## 17. Test Restores Regularly

A backup is not proven until it has been restored.

Test:

* Full database restore
* Point-in-time recovery
* Cross-region recovery
* Credential recovery
* Application compatibility
* Recovery time objective
* Recovery point objective

Document the process as an operational runbook.

---

## 18. Use Least-Privilege Accounts

Create separate database users for:

* Application runtime
* Schema migrations
* Read-only reporting
* Replication
* Backups
* Administration

The application should not connect as `root`.

Example:

```sql
CREATE USER 'app_runtime'@'%'
IDENTIFIED BY 'strong-secret';

GRANT SELECT, INSERT, UPDATE, DELETE
ON ecommerce.*
TO 'app_runtime'@'%';
```

Avoid granting schema-changing privileges to normal runtime accounts.

---

## 19. Encrypt Connections

Use TLS for:

* Application-to-database traffic
* Replication traffic
* Administrative access
* Backup transfers

Also protect:

* Passwords
* Connection strings
* Certificates
* Encryption keys
* Backup files

---

## 20. Use Parameterized Queries

Poor:

```text
"SELECT * FROM users WHERE email = '" + userInput + "'"
```

Better:

```sql
SELECT id, email
FROM users
WHERE email = ?;
```

Parameterized queries reduce SQL injection risk and improve code clarity.

---

## 21. Plan Online Schema Changes

Large schema changes can block traffic or rebuild tables.

Potentially expensive operations include:

* Adding indexes
* Changing column types
* Reordering columns
* Adding constraints
* Modifying primary keys
* Changing character sets

Safer deployment pattern:

```text
1. Add new nullable structure
2. Deploy compatible application code
3. Backfill data in batches
4. Validate new data
5. Switch reads and writes
6. Remove old structure later
```

Use online DDL capabilities and migration tooling where appropriate.

---

## 22. Use Expand-and-Contract Migrations

### Expand

Add new columns, tables, or indexes without removing old ones.

### Migrate

Write both formats and backfill existing data.

### Contract

Remove obsolete structures after all applications stop using them.

```text
Old Application ──┐
                  ├── Compatible Schema
New Application ──┘
```

This supports rolling deployment and rollback.

---

## 23. Avoid Large Deletes in One Transaction

A massive delete can cause:

* Long lock duration
* Huge undo logs
* Large binary logs
* Replica lag
* Extended rollback
* Storage pressure

Use batched deletion:

```sql
DELETE FROM audit_events
WHERE created_at < '2025-01-01'
LIMIT 10000;
```

Repeat safely until complete.

For time-based data, partition deletion may be more efficient.

---

## 24. Separate Transactional and Analytical Workloads

Large reports may consume:

* CPU
* Buffer-pool pages
* Temporary storage
* Disk bandwidth
* Connections
* Lock resources

Use:

* Read replicas
* Summary tables
* Data warehouses
* ETL pipelines
* Materialized aggregates
* Query timeouts

Protect user-facing transactions from analytical spikes.

---

## 25. Monitor Database Health

Track at least:

* Query latency
* Throughput
* Active connections
* Connection errors
* Buffer-pool hit ratio
* Disk latency
* Lock waits
* Deadlocks
* Long-running transactions
* Slow queries
* Temporary-table creation
* Binary-log generation
* Replica lag
* CPU and memory
* Disk capacity

Observability should include query-level and system-level metrics.

---

## 26. Enable the Slow Query Log Carefully

The slow query log helps identify expensive queries.

Use it to detect:

* Long-running statements
* Queries examining too many rows
* Missing indexes
* Frequent expensive operations

Analyze results rather than optimizing only the single slowest query.

A moderately slow query running millions of times may be more important than one rare report.

---

## 27. Use Caching Selectively

Good cache candidates:

* Product details
* Public configuration
* Category trees
* Popular read-heavy objects
* Expensive aggregate results

Poor cache candidates:

* Account balances
* Real-time inventory decisions
* Payment state without consistency controls
* Security permissions that must change immediately

Always define:

* Cache key
* TTL
* Invalidation behavior
* Failure behavior
* Source of truth

---

## 28. Avoid Premature Sharding

Before sharding, consider:

* Query optimization
* Better indexes
* Vertical scaling
* Connection-pool tuning
* Read replicas
* Redis caching
* Archiving
* Partitioning
* Workload separation

Sharding should solve a measured limitation, not an imagined future problem.

---

# Common Mistakes

## 1. Using the Wrong Character Set

Using an incomplete Unicode character set may break emoji and international text.

Prefer:

```text
utf8mb4
```

Verify database, table, column, and connection settings.

---

## 2. Creating Too Many Indexes

Every index increases:

* Insert cost
* Update cost
* Delete cost
* Binary-log volume
* Storage use
* Buffer-pool pressure

Remove unused and redundant indexes after careful testing.

---

## 3. Missing Composite Indexes

Creating separate indexes on two columns does not always replace one well-designed composite index.

Query:

```sql
WHERE customer_id = ?
  AND status = ?
ORDER BY created_at DESC
```

A combined index may be more useful than three isolated indexes.

---

## 4. Ignoring the Leftmost Prefix Rule

An index on:

```text
(customer_id, status)
```

does not necessarily optimize:

```sql
WHERE status = 'pending';
```

Column order must reflect query patterns.

---

## 5. Using Random Large Primary Keys Carelessly

Random primary-key inserts can create:

* Page splits
* Fragmentation
* Larger secondary indexes
* Reduced cache locality

Use compact and ordered identifiers where possible.

---

## 6. Using Floating Point for Money

Poor:

```sql
amount DOUBLE
```

Better:

```sql
amount DECIMAL(18, 2)
```

or:

```sql
amount_in_cents BIGINT
```

Financial calculations require exact values.

---

## 7. Treating JSON as a Replacement for Relational Design

Poor:

```sql
CREATE TABLE orders (
    data JSON
);
```

This weakens:

* Constraints
* Type safety
* Relationships
* Query clarity
* Index design

Use JSON for genuinely flexible fields, not for the entire core model.

---

## 8. Performing Read-Then-Write Without Locking

Unsafe:

```text
Read inventory = 1
Check inventory
Update inventory = 0
```

Two transactions can both read the same value.

Use:

* Atomic conditional updates
* `SELECT ... FOR UPDATE`
* Unique constraints
* Optimistic version checks

---

## 9. Keeping Transactions Open During API Calls

Holding a transaction open while calling another service can:

* Hold locks
* Exhaust the connection pool
* Increase deadlocks
* Grow undo history
* Increase replica lag

Use state machines and asynchronous events instead.

---

## 10. Assuming Replicas Are Immediately Consistent

Replication lag can cause stale results.

Do not route every read to replicas without understanding correctness requirements.

---

## 11. Using Replication as a Backup

Replicas copy destructive changes.

A dropped table is also dropped on the replica.

Maintain independent backups and point-in-time recovery.

---

## 12. Running With Unlimited Connections

Too many connections can cause:

* Memory exhaustion
* Thread contention
* Slow context switching
* Connection storms
* Database instability

Use bounded pools and backpressure.

---

## 13. Using Deep Offset Pagination

This query becomes increasingly expensive:

```sql
LIMIT 20 OFFSET 1000000;
```

Use keyset pagination for large result sets.

---

## 14. Ignoring Collation Behavior

Case-insensitive collations may consider values equivalent.

```text
Alex@example.com
alex@example.com
```

A unique constraint may treat these as duplicates depending on collation.

Choose collations based on business requirements.

---

## 15. Running Destructive Migrations Directly

Changing a large table may cause:

* Long metadata locks
* Table reconstruction
* Deployment outage
* Replica lag
* Disk exhaustion

Test with production-like data and use staged migrations.

---

## 16. Using `SELECT *`

This can:

* Transfer unnecessary data
* Prevent covering-index use
* Expose sensitive columns
* Break clients after schema changes

Select only required columns.

---

## 17. Forgetting Deterministic Ordering

Poor:

```sql
SELECT *
FROM orders
LIMIT 20;
```

Rows are not guaranteed to return in stable order.

Better:

```sql
SELECT id, status, created_at
FROM orders
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

Use a unique tie-breaker.

---

## 18. Not Indexing Foreign-Key Access Paths

Foreign-key columns commonly participate in joins and parent-row operations.

Example:

```sql
orders.customer_id
```

Create a supporting index when it is not already covered:

```sql
CREATE INDEX idx_orders_customer
ON orders(customer_id);
```

---

## 19. Ignoring Deadlocks

Applications should expect deadlock errors in concurrent transactional workloads.

Use bounded retry logic for safe, idempotent transactions.

Do not retry indefinitely.

---

## 20. Sharding Too Early

Sharding creates significant complexity:

* Distributed queries
* Cross-shard transactions
* Data movement
* Hot shards
* Per-shard backups
* Routing failures

Scale vertically and optimize first.

---

# Interview Questions

## 1. What is InnoDB?

InnoDB is MySQL’s primary transactional storage engine. It provides ACID transactions, row-level locking, foreign keys, MVCC, crash recovery, and clustered indexes.

---

## 2. What is the difference between a clustered index and a secondary index in InnoDB?

The clustered index stores the full row organized by primary key. A secondary index stores its indexed columns along with the row’s primary-key value.

---

## 3. Why can a read replica return stale data?

Replication is often asynchronous. The primary may commit a transaction before a replica receives and applies the corresponding binary-log events.

---

## 4. What is the leftmost prefix rule?

A composite B-tree index is most useful when a query filters by the leading columns of that index in order. An index on `(a, b, c)` commonly supports `a`, `a + b`, and `a + b + c`.

---

## 5. How do you prevent overselling inventory in MySQL?

Use a short transaction with an atomic conditional update or a row lock. For example, decrement inventory only when the available quantity is greater than or equal to the requested amount.

---

# Key Takeaways

1. **MySQL with InnoDB is a reliable default for transactional backend systems because it combines relational modeling, ACID transactions, row-level locking, mature replication, and broad ecosystem support.**

2. **Production performance depends on deliberate schema design, compact primary keys, query-driven indexes, short transactions, bounded connection pools, query-plan analysis, and healthy replication.**

3. **Scale MySQL progressively: optimize queries and indexes first, then add caching, read replicas, workload separation, partitioning, and stronger hardware before accepting the complexity of sharding.**
