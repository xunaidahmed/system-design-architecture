# MySQL Database Design & System Design Patterns

A practical guide to designing **scalable, reliable, and high-performance MySQL databases** for SaaS platforms, marketplaces, FinTech applications, ERP systems, APIs, and enterprise applications.

---

# Table of Contents

1. [Relational Database Design](#1-relational-database-design)
2. [Normalization](#2-normalization)
3. [Indexing Strategy](#3-indexing-strategy)
4. [Composite Index Pattern](#4-composite-index-pattern)
5. [Transaction Pattern](#5-transaction-pattern)
6. [Audit Log Pattern](#6-audit-log-pattern)
7. [Soft Delete Pattern](#7-soft-delete-pattern)
8. [Read/Write Separation](#8-readwrite-separation)
9. [Partitioning](#9-partitioning)
10. [Replication & Sharding](#10-replication--sharding)
11. [Complete Scalable MySQL Architecture](#complete-scalable-mysql-architecture)

---

# 1. Relational Database Design

## Overview

MySQL is a relational database system where information is organized into:

```text
Database
   │
   ├── Tables
   │    │
   │    ├── Columns
   │    └── Rows
   │
   ├── Relationships
   │
   ├── Indexes
   │
   └── Constraints
```

A typical SaaS application:

```text
MySQL
│
├── users
├── roles
├── permissions
├── products
├── orders
├── order_items
├── payments
├── invoices
└── audit_logs
```

---

# 2. Normalization

## Overview

Normalization reduces duplicate data and improves data consistency.

Common levels:

```text
1NF
 ↓
2NF
 ↓
3NF
```

For most application databases, **3NF** is a useful starting point, followed by selective denormalization where performance requires it.

---

## Bad Design

```text
orders

id
customer_name
customer_email
customer_phone
product_name
product_price
```

The customer information may be duplicated across thousands of orders.

---

## Better Design

```text
users
├── id
├── name
├── email
└── phone

orders
├── id
├── user_id
└── total

products
├── id
├── name
└── price

order_items
├── id
├── order_id
├── product_id
├── quantity
└── price
```

### Relationship

```text
User
 │
 └──────< Orders
              │
              └──────< Order Items >────── Product
```

---

# 3. Indexing Strategy

## Overview

Indexes allow MySQL to locate rows efficiently without scanning the entire table.

Without an index:

```text
10 Million Rows
       │
       ▼
Full Table Scan
       │
       ▼
Slow Query
```

With an index:

```text
10 Million Rows
       │
       ▼
Index
       │
       ▼
Matching Rows
```

---

## Example

```sql
CREATE INDEX idx_users_email
ON users(email);
```

Query:

```sql
SELECT *
FROM users
WHERE email = 'user@example.com';
```

---

## Common Index Types

```text
PRIMARY KEY
UNIQUE INDEX
INDEX
COMPOSITE INDEX
FULLTEXT INDEX
```

---

## Important Rule

Do not create indexes on every column.

Indexes improve reads but add:

* Storage
* INSERT cost
* UPDATE cost
* DELETE cost
* Maintenance overhead

---

# 4. Composite Index Pattern

A composite index contains multiple columns.

Example query:

```sql
SELECT *
FROM orders
WHERE user_id = 10
AND status = 'paid'
ORDER BY created_at DESC;
```

Recommended index:

```sql
CREATE INDEX idx_orders_user_status_created
ON orders(user_id, status, created_at);
```

### Structure

```text
Index

user_id
   ↓
status
   ↓
created_at
```

---

## Leftmost Prefix Rule

For:

```sql
INDEX(user_id, status, created_at)
```

MySQL can efficiently use combinations beginning from:

```text
user_id
user_id + status
user_id + status + created_at
```

Index design should be based on actual query patterns, not simply the number of columns.

---

# 5. Transaction Pattern

## Overview

Transactions ensure that a group of database operations succeeds or fails together.

### Example

Bank transfer:

```text
Account A
   │
   ├── -100
   │
   ▼
Account B
   │
   └── +100
```

Both operations must succeed.

---

## MySQL Example

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

---

## Laravel Example

```php
DB::transaction(function () {

    $sender->decrement('balance', 100);

    $receiver->increment('balance', 100);

});
```

---

## ACID

MySQL transactions provide the ACID model:

```text
A = Atomicity
C = Consistency
I = Isolation
D = Durability
```

---

# 6. Audit Log Pattern

## Overview

An audit log records important changes made to the system.

Useful for:

* FinTech
* Admin panels
* Enterprise systems
* Security
* Compliance
* Debugging

---

## Database

```sql
CREATE TABLE audit_logs (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT UNSIGNED NULL,
    action VARCHAR(100) NOT NULL,
    entity_type VARCHAR(100) NULL,
    entity_id BIGINT UNSIGNED NULL,
    old_values JSON NULL,
    new_values JSON NULL,
    ip_address VARCHAR(45) NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## Example

```text
Admin
  │
  ▼
Update User
  │
  ▼
Audit Log
  │
  ├── User ID
  ├── Old Value
  ├── New Value
  ├── IP Address
  └── Timestamp
```

---

# 7. Soft Delete Pattern

## Overview

Instead of permanently deleting records, mark them as deleted.

Example:

```sql
ALTER TABLE users
ADD deleted_at TIMESTAMP NULL;
```

Active records:

```sql
SELECT *
FROM users
WHERE deleted_at IS NULL;
```

Deleted records:

```sql
SELECT *
FROM users
WHERE deleted_at IS NOT NULL;
```

---

## Laravel

```php
use Illuminate\Database\Eloquent\SoftDeletes;

class User extends Model
{
    use SoftDeletes;
}
```

---

## Benefits

* Data recovery
* Auditability
* Safer deletion
* Historical records

---

# 8. Read/Write Separation

## Overview

When database traffic grows, separate read and write workloads.

```text
                    Application
                         │
               ┌─────────┴─────────┐
               │                   │
             WRITE                READ
               │                   │
               ▼                   ▼
          Primary DB          Read Replica
               │                   │
               │                   ├── Replica 1
               │                   ├── Replica 2
               │                   └── Replica 3
               │
               └──── Replication ────►
```

---

## Write Operations

```text
INSERT
UPDATE
DELETE
```

Go to the primary database.

## Read Operations

```text
SELECT
```

Can be served by read replicas.

---

## Laravel Concept

```php
DB::connection('mysql')->insert(...);

DB::connection('mysql_read')->select(...);
```

---

## Benefits

* Better read scalability
* Reduced primary DB load
* Better performance for read-heavy systems

---

# 9. Partitioning

## Overview

Partitioning divides a large table into smaller logical partitions while keeping it as one table from the application's perspective.

Example:

```text
orders

2024
 ├── Partition

2025
 ├── Partition

2026
 ├── Partition
```

---

## RANGE Partitioning

```sql
CREATE TABLE orders (
    id BIGINT NOT NULL,
    user_id BIGINT NOT NULL,
    total DECIMAL(12,2),
    created_at DATE NOT NULL
)
PARTITION BY RANGE (YEAR(created_at)) (

    PARTITION p2024 VALUES LESS THAN (2025),

    PARTITION p2025 VALUES LESS THAN (2026),

    PARTITION p2026 VALUES LESS THAN (2027),

    PARTITION pmax VALUES LESS THAN MAXVALUE
);
```

---

## Useful For

* Very large tables
* Logs
* Transactions
* Events
* Analytics
* Time-series data

Partitioning should be introduced based on workload and query patterns; it is not automatically faster for every large table.

---

# 10. Replication & Sharding

## Replication

Replication copies data from a primary server to replicas.

```text
                 Primary
                    │
          ┌─────────┼─────────┐
          │         │         │
          ▼         ▼         ▼
       Replica 1 Replica 2 Replica 3
```

Useful for:

* Read scaling
* High availability architectures
* Disaster recovery

---

# Sharding

## Overview

Sharding distributes data across multiple database servers.

Example:

```text
                  Application
                       │
              Sharding Router
                       │
       ┌───────────────┼───────────────┐
       │               │               │
       ▼               ▼               ▼
   MySQL Shard 1   MySQL Shard 2   MySQL Shard 3
   Users 1-1M      Users 1M-2M     Users 2M-3M
```

---

## Hash-Based Sharding

Example:

```text
shard = user_id % 4
```

```text
User 101 → Shard 1
User 102 → Shard 2
User 103 → Shard 3
User 104 → Shard 0
```

---

## Geographic Sharding

```text
UAE Users
   │
   ▼
UAE Database

Europe Users
   │
   ▼
Europe Database

US Users
   │
   ▼
US Database
```

This can be useful when data residency, latency, or workload isolation requirements justify it.

---

# Complete Scalable MySQL Architecture

A larger production architecture can combine several techniques:

```text
                         USERS
                           │
                           ▼
                    LOAD BALANCER
                           │
                           ▼
                       API SERVERS
                           │
              ┌────────────┼────────────┐
              │            │            │
              ▼            ▼            ▼
           Laravel      Laravel      Laravel
           Server 1     Server 2     Server 3
              │            │            │
              └────────────┼────────────┘
                           │
                    DATABASE ROUTER
                           │
                 ┌─────────┴─────────┐
                 │                   │
               WRITE                READ
                 │                   │
                 ▼                   ▼
              PRIMARY            REPLICAS
                 │             ┌─────┼─────┐
                 │             ▼     ▼     ▼
                 │           Read1 Read2 Read3
                 │
                 └──────── Replication ────────►
```

---

# Large Database Architecture

For very large systems:

```text
                         APPLICATION
                              │
                              ▼
                       DATABASE ROUTER
                              │
                ┌─────────────┼─────────────┐
                │             │             │
                ▼             ▼             ▼
             Shard 1       Shard 2       Shard 3
                │             │             │
          ┌─────┴─────┐ ┌─────┴─────┐ ┌─────┴─────┐
          ▼           ▼ ▼           ▼ ▼           ▼
       Primary     Replica       Primary       Replica
```

---

# Recommended MySQL Database Structure

A scalable SaaS application might contain:

```text
database
│
├── users
├── roles
├── permissions
├── user_roles
│
├── organizations
├── organization_users
│
├── products
├── categories
│
├── orders
├── order_items
│
├── payments
├── transactions
│
├── invoices
├── subscriptions
│
├── notifications
├── jobs
│
├── audit_logs
└── system_logs
```

---

# Example Relationships

```text
Organization
     │
     ├──────< Users
     │
     ├──────< Products
     │
     └──────< Orders
                    │
                    └──────< Order Items
                                  │
                                  └────── Product

Order
 │
 ├──── Payment
 │
 └──── Invoice
```

---

# Database Design Rules

## 1. Use Primary Keys

```sql
id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY
```

## 2. Use Foreign Keys Where Appropriate

```sql
FOREIGN KEY (user_id)
REFERENCES users(id)
```

## 3. Use Proper Data Types

Do not use:

```sql
VARCHAR(5000)
```

when:

```sql
VARCHAR(255)
```

is sufficient.

Use:

```text
INT / BIGINT
DECIMAL
DATE / DATETIME / TIMESTAMP
BOOLEAN
JSON
TEXT
```

according to the data and query requirements.

---

# Money Should Use DECIMAL

Avoid:

```sql
FLOAT
DOUBLE
```

for financial amounts when exact decimal arithmetic is required.

Use:

```sql
DECIMAL(15,2)
```

Example:

```sql
amount DECIMAL(15,2) NOT NULL
```

---

# Use EXPLAIN

Before optimizing a slow query:

```sql
EXPLAIN
SELECT *
FROM orders
WHERE user_id = 100
AND status = 'paid';
```

For deeper analysis in modern MySQL versions:

```sql
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE user_id = 100
AND status = 'paid';
```

Look for:

```text
access type
possible keys
key
rows
filtered
execution cost
```

---

# Avoid SELECT *

Instead of:

```sql
SELECT *
FROM users;
```

Prefer:

```sql
SELECT id, name, email
FROM users;
```

This reduces unnecessary data transfer and makes query intent clearer.

---

# Pagination

Avoid large OFFSET pagination for very large datasets.

Basic:

```sql
SELECT *
FROM orders
ORDER BY id DESC
LIMIT 20 OFFSET 100000;
```

For large datasets, consider keyset/cursor pagination:

```sql
SELECT *
FROM orders
WHERE id < 100000
ORDER BY id DESC
LIMIT 20;
```

This is often more efficient for deep pagination when the ordering column is indexed.

---

# Caching Layer

MySQL should not necessarily handle every repeated read.

A common architecture is:

```text
Application
    │
    ▼
   Redis
    │
    ├── Cache Hit ──────► Response
    │
    └── Cache Miss
             │
             ▼
           MySQL
             │
             ▼
           Redis
             │
             ▼
          Response
```

Use caching for suitable read-heavy or expensive data.

---

# Database Performance Checklist

```text
[ ] Proper primary keys
[ ] Proper foreign keys
[ ] Correct data types
[ ] Proper indexes
[ ] Composite indexes where required
[ ] Avoid unnecessary SELECT *
[ ] Analyze slow queries
[ ] Use EXPLAIN
[ ] Use transactions where required
[ ] Use connection pooling where applicable
[ ] Use caching where appropriate
[ ] Read replicas for read-heavy workloads
[ ] Partition very large tables when justified
[ ] Archive old data when appropriate
[ ] Monitor slow queries
[ ] Monitor CPU / RAM / IOPS
[ ] Backup database
[ ] Test restore process
```

---

# Database Scaling Roadmap

Do not immediately jump to sharding.

A practical progression is:

```text
Stage 1
Single MySQL Server
       │
       ▼
Stage 2
Proper Indexing
       │
       ▼
Stage 3
Query Optimization
       │
       ▼
Stage 4
Redis / Caching
       │
       ▼
Stage 5
Read Replicas
       │
       ▼
Stage 6
Partitioning
       │
       ▼
Stage 7
Database Optimization / Archiving
       │
       ▼
Stage 8
Sharding
```

---

# Final MySQL Architecture

```text
                         CLIENTS
                            │
                            ▼
                       LOAD BALANCER
                            │
                            ▼
                       APPLICATION
                            │
                ┌───────────┴───────────┐
                │                       │
                ▼                       ▼
              Redis                   MySQL
            Cache Layer                │
                                ┌───────┴───────┐
                                │               │
                              WRITE            READ
                                │               │
                                ▼               ▼
                             PRIMARY         REPLICAS
                                │           ┌────┼────┐
                                │           ▼    ▼    ▼
                                └──────►   R1   R2   R3
```

For extremely large workloads:

```text
                         APPLICATION
                              │
                              ▼
                       SHARDING ROUTER
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
           SHARD 1         SHARD 2         SHARD 3
              │               │               │
           Primary          Primary          Primary
              │               │               │
           Replica          Replica          Replica
```

---

# Key Principles

### 1. Start Simple

Use a well-designed relational database before introducing distributed complexity.

### 2. Optimize Before Scaling

Find the actual bottleneck before adding replicas or shards.

### 3. Index Based on Queries

Indexes should support real query patterns.

### 4. Protect Data Integrity

Use constraints and transactions where appropriate.

### 5. Separate Reads and Writes When Needed

Read replicas can help with read-heavy workloads.

### 6. Partition Large Data Carefully

Partitioning should match access patterns and operational needs.

### 7. Shard Only When Necessary

Sharding adds significant application and operational complexity.

### 8. Always Have Backups

A backup is only useful if the restore process has been tested.

---

# Quick Reference

| Requirement              | Pattern / Technique        |
| ------------------------ | -------------------------- |
| Reduce duplicate data    | Normalization              |
| Faster lookups           | Indexing                   |
| Multi-column filtering   | Composite Index            |
| Atomic operations        | Transactions               |
| Track changes            | Audit Logs                 |
| Recover deleted records  | Soft Delete                |
| Read scalability         | Replication                |
| Large time-based tables  | Partitioning               |
| Massive datasets         | Sharding                   |
| Repeated expensive reads | Redis / Cache              |
| Deep pagination          | Cursor / Keyset Pagination |

---

# Conclusion

A high-quality MySQL architecture is not about adding the maximum number of database technologies.

The objective is to build a database that is:

* **Correct**
* **Consistent**
* **Performant**
* **Secure**
* **Scalable**
* **Recoverable**
* **Easy to maintain**

A practical architecture usually evolves from:

```text
Good Schema
     ↓
Normalization
     ↓
Indexes
     ↓
Query Optimization
     ↓
Transactions
     ↓
Caching
     ↓
Replication
     ↓
Partitioning
     ↓
Sharding
```

Use each technique when the application's actual requirements justify its complexity.
