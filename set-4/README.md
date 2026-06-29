# Set 4

| S.No. | Question                                                                                                                                   |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| 1.    | [Difference between SQL and NoSQL databases](#question-1-difference-between-sql-and-nosql-databases)                                       |
| 2.    | [When should you use SQL over NoSQL and vice versa?](#question-2-when-should-you-use-sql-over-nosql-and-vice-versa)                        |
| 3.    | [What is database normalization?](#question-3-what-is-database-normalization)                                                              |
| 4.    | [What is denormalization and when is it useful?](#question-4-what-is-denormalization-and-when-is-it-useful)                                |
| 5.    | [What are the different types of NoSQL databases?](#question-5-what-are-the-different-types-of-nosql-databases)                            |
| 6.    | [What is a graph database and when is it used?](#question-6-what-is-a-graph-database-and-when-is-it-used)                                  |
| 7.    | [What is database partitioning?](#question-7-what-is-database-partitioning)                                                                |
| 8.    | [What is consistent hashing in system design?](#question-8-what-is-consistent-hashing-in-system-design)                                    |
| 9.    | [How do you handle schema migrations in distributed databases?](#question-9-how-do-you-handle-schema-migrations-in-distributed-databases)  |
| 10.   | [How do you design a distributed transaction system?](#question-10-how-do-you-design-a-distributed-transaction-system)                     |
| 11.   | [What is REST API vs GraphQL vs gRPC?](#question-11-what-is-rest-api-vs-graphql-vs-grpc)                                                   |
| 12.   | [What is WebSocket and when to use it?](#question-12-what-is-websocket-and-when-to-use-it)                                                 |
| 13.   | [How do you design a push notification system?](#question-13-how-do-you-design-a-push-notification-system)                                 |
| 14.   | [What is polling vs long-polling vs server-sent events?](#question-14-what-is-polling-vs-long-polling-vs-server-sent-events)               |
| 15.   | [What is API Gateway in microservices?](#question-15-what-is-api-gateway-in-microservices)                                                 |
| 16.   | [How do you ensure secure communication between services?](#question-16-how-do-you-ensure-secure-communication-between-services)           |
| 17.   | [What is OAuth 2.0 and how it's used in system design?](#question-17-what-is-oauth-20-and-how-its-used-in-system-design)                   |
| 18.   | [How do you design an authentication system like OAuth or JWT?](#question-18-how-do-you-design-an-authentication-system-like-oauth-or-jwt) |
| 19.   | [What is CORS and why does it matter in system design?](#question-19-what-is-cors-and-why-does-it-matter-in-system-design)                 |
| 20.   | [What are retries and backoff strategies in networking?](#question-20-what-are-retries-and-backoff-strategies-in-networking)               |

## Question 1. Difference between SQL and NoSQL databases

# Direct Answer

**SQL and NoSQL are two different categories of databases.**

- **SQL databases** are relational databases that store data in tables with predefined schemas and use SQL (Structured Query Language) for querying.
- **NoSQL databases** are non-relational databases designed for flexible schemas, horizontal scalability, and handling large volumes of distributed data.

In interviews, the key distinction is:

> **Choose SQL when data consistency and relationships are important. Choose NoSQL when scalability, flexibility, and high throughput are the primary requirements.**

---

# Comparison Table

| Feature           | SQL Database                         | NoSQL Database                         |
| ----------------- | ------------------------------------ | -------------------------------------- |
| Data Model        | Tables (rows & columns)              | Document, Key-Value, Column, Graph     |
| Schema            | Fixed schema                         | Flexible / schema-less                 |
| Relationships     | Strong support (joins, foreign keys) | Usually denormalized data              |
| Query Language    | SQL                                  | Database-specific APIs/query languages |
| Scalability       | Traditionally vertical               | Designed for horizontal scaling        |
| Consistency       | Strong consistency (ACID)            | Often eventual consistency (BASE)      |
| Transactions      | Full ACID transactions               | Limited or specialized transactions    |
| Complex Queries   | Excellent                            | Usually limited compared to SQL        |
| Data Integrity    | Strong constraints                   | Application often enforces integrity   |
| Typical Use Cases | Banking, ERP, Inventory              | Social media, analytics, IoT, caching  |

---

# Examples

### SQL Databases

- MySQL
- PostgreSQL
- Oracle Database
- Microsoft SQL Server

Example table:

**Users**

| id  | name  | email                                   |
| --- | ----- | --------------------------------------- |
| 1   | John  | [john@test.com](mailto:john@test.com)   |
| 2   | Alice | [alice@test.com](mailto:alice@test.com) |

SQL Query:

```sql
SELECT * FROM Users WHERE id = 1;
```

---

### NoSQL Databases

- MongoDB
- Apache Cassandra
- Redis
- Amazon DynamoDB

Example document:

```json
{
  "id": 1,
  "name": "John",
  "email": "john@test.com"
}
```

Query:

```javascript
db.users.find({ id: 1 });
```

---

# Why SQL Uses Joins and NoSQL Often Doesn't

### SQL

Normalized data:

```text
Users
-----
1, John

Orders
------
101, UserId=1
102, UserId=1
```

Query:

```sql
SELECT *
FROM Users u
JOIN Orders o
ON u.id = o.userId;
```

SQL databases are optimized for relationships.

### NoSQL

Data is often denormalized:

```json
{
  "userId": 1,
  "name": "John",
  "orders": [{ "id": 101 }, { "id": 102 }]
}
```

The entire user document can be fetched with a single lookup.

---

# Scalability Perspective

### SQL Scaling

```text
App
 |
Primary DB
 |
Read Replicas
```

Common approaches:

- Vertical scaling (larger machine)
- Read replicas
- Sharding (more complex)

### NoSQL Scaling

```text
App
 |
+-----+-----+-----+
|Node1|Node2|Node3|
+-----+-----+-----+
```

Designed for:

- Automatic partitioning/sharding
- Distributed storage
- Massive scale

Examples:

- Social media feeds
- Event tracking
- IoT telemetry

---

# ACID vs BASE

| SQL                          | NoSQL                   |
| ---------------------------- | ----------------------- |
| ACID                         | BASE                    |
| Strong consistency           | Eventual consistency    |
| Strict transactions          | High availability       |
| Data correctness prioritized | Scalability prioritized |

### ACID

- Atomicity
- Consistency
- Isolation
- Durability

Example: Bank transfer.

### BASE

- Basically Available
- Soft State
- Eventual Consistency

Example: Social media likes count.

---

# When to Use Which?

### Use SQL When

✅ Banking systems
✅ Payment systems
✅ Inventory management
✅ Order management
✅ Financial records
✅ Strong relationships between entities

Example: An e-commerce order database where order, payment, and inventory updates must succeed together.

---

### Use NoSQL When

✅ User activity tracking
✅ Social media feeds
✅ Real-time analytics
✅ IoT sensor data
✅ Large-scale distributed systems
✅ Rapidly changing schemas

Example: Storing billions of user events per day.

---

# Interview-Ready Summary

> SQL databases store structured data in relational tables and provide strong consistency, ACID transactions, and powerful joins. NoSQL databases use flexible schemas and are designed for horizontal scalability, high throughput, and distributed architectures. SQL is preferred for transactional systems requiring data integrity, while NoSQL is preferred for large-scale applications where scalability and schema flexibility are more important.

## Question 2. When should you use SQL over NoSQL and vice versa?

# Direct Answer

The choice depends on the **data model, consistency requirements, and scale**.

- Use **SQL** when you need **strong consistency, transactions, and complex relationships** between data.
- Use **NoSQL** when you need **massive horizontal scalability, flexible schemas, and very high read/write throughput**.

In practice, many large systems use **both** (polyglot persistence).

---

# When to Use SQL

Choose SQL databases when:

### 1. Strong ACID Transactions Are Required

Example:

- Banking
- Payments
- Trading systems
- Order processing

A money transfer must never leave the system in an inconsistent state.

```text
Debit Account A
Credit Account B
```

Either both succeed or both fail.

---

### 2. Data Has Complex Relationships

Example:

```text
Users
Orders
Products
Payments
Reviews
```

These entities are connected and often queried together.

SQL databases handle:

- Joins
- Foreign keys
- Referential integrity

efficiently.

---

### 3. Data Schema Is Stable

Example:

```text
Customer
 ├── Name
 ├── Email
 └── Address
```

Fields don't change frequently.

---

### 4. Complex Queries and Reporting

Example:

```sql
SELECT region,
       SUM(revenue)
FROM orders
GROUP BY region;
```

Analytics, reporting, aggregations, and ad-hoc queries are usually easier in SQL.

---

# Typical SQL Use Cases

| System               | Why SQL?              |
| -------------------- | --------------------- |
| Banking              | Strong consistency    |
| Payment Gateway      | ACID transactions     |
| ERP Systems          | Complex relationships |
| Inventory Management | Data integrity        |
| Order Management     | Transactions + joins  |
| CRM                  | Structured data       |

---

# When to Use NoSQL

Choose NoSQL when:

### 1. Massive Scale Is Required

Example:

```text
Billions of events/day
Millions of users
Thousands of writes/sec
```

NoSQL systems are designed for horizontal scaling.

---

### 2. Schema Changes Frequently

Example:

User profiles:

```json
{
  "name": "John",
  "age": 25
}
```

Later:

```json
{
  "name": "John",
  "age": 25,
  "linkedin": "...",
  "skills": [...]
}
```

No migration required.

---

### 3. Denormalized Data Works Better

Instead of joining multiple tables:

```text
Users
Orders
Products
```

Store everything together:

```json
{
  "userId": 123,
  "orders": [...]
}
```

Faster reads.

---

### 4. High Write Throughput Is Needed

Examples:

- Clickstream data
- Logs
- Metrics
- IoT telemetry

Millions of writes per second can be distributed across nodes.

---

# Typical NoSQL Use Cases

| System                 | Why NoSQL?            |
| ---------------------- | --------------------- |
| Social Media Feed      | Massive scale         |
| Event Tracking         | High write throughput |
| IoT Data               | Flexible schema       |
| User Activity Logs     | Large volume          |
| Recommendation Systems | Fast lookups          |
| Real-Time Analytics    | Distributed storage   |

---

# Real-World Examples

### E-commerce System

A common interview answer:

```text
                 E-commerce
                      |
      +---------------+---------------+
      |                               |
   SQL DB                         NoSQL DB
      |                               |
 Orders, Payments,             User Activity,
 Inventory, Users              Clickstream,
                               Recommendations
```

### Why?

**Orders and Payments**

- Require transactions
- Require consistency
- Use SQL

**User Behavior Events**

- Massive scale
- Append-heavy writes
- Use NoSQL

---

# Trade-offs

| Factor                 | SQL             | NoSQL                         |
| ---------------------- | --------------- | ----------------------------- |
| Consistency            | Strong          | Often eventual                |
| Transactions           | Excellent       | Limited                       |
| Joins                  | Excellent       | Weak/None                     |
| Horizontal Scaling     | Harder          | Easier                        |
| Schema Flexibility     | Low             | High                          |
| Query Complexity       | Powerful        | Simpler                       |
| Write Throughput       | Moderate        | Very high                     |
| Operational Complexity | Lower initially | Higher distributed complexity |

---

# Interview Rule of Thumb

A simple interview framework:

### Choose SQL if:

- Data correctness is critical
- Transactions are required
- Relationships are important
- Complex queries are common

Examples:

- Banking
- Payments
- Inventory
- Order Management

---

### Choose NoSQL if:

- Scale is the biggest challenge
- Data model evolves frequently
- Reads/writes are extremely high
- Data can tolerate eventual consistency

Examples:

- Social feeds
- Analytics
- Logging
- IoT

---

# Interview-Ready Summary

> Use SQL when you need strong consistency, ACID transactions, and relational data with complex queries. Use NoSQL when you need horizontal scalability, flexible schemas, and very high read/write throughput. In large-scale systems, it's common to use SQL for transactional data such as orders and payments, and NoSQL for high-volume data such as user activity, logs, feeds, and analytics.

## Question 3. What is database normalization?

# Direct Answer

**Database normalization** is the process of organizing data in a database to **reduce redundancy (duplicate data)** and **improve data integrity** by splitting data into related tables and defining relationships between them.

The main goal is:

> **Store each piece of information only once and avoid update, insert, and delete anomalies.**

---

# Why Do We Need Normalization?

Consider a single table:

| OrderId | CustomerName | CustomerEmail                         | Product  | Price |
| ------- | ------------ | ------------------------------------- | -------- | ----- |
| 101     | John         | [john@test.com](mailto:john@test.com) | Laptop   | 1000  |
| 102     | John         | [john@test.com](mailto:john@test.com) | Mouse    | 20    |
| 103     | John         | [john@test.com](mailto:john@test.com) | Keyboard | 50    |

Problems:

### Data Redundancy

```text
John
john@test.com
```

are repeated in every row.

### Update Anomaly

If John's email changes:

```text
john@test.com
→
john123@test.com
```

Every row must be updated.

### Delete Anomaly

If Order 103 is deleted and it was the last order, customer information may be lost.

### Insert Anomaly

Cannot add a customer until they place an order.

---

# Normalized Design

Split the data into separate tables.

### Customers

| CustomerId | Name | Email                                 |
| ---------- | ---- | ------------------------------------- |
| 1          | John | [john@test.com](mailto:john@test.com) |

### Orders

| OrderId | CustomerId |
| ------- | ---------- |
| 101     | 1          |
| 102     | 1          |
| 103     | 1          |

### Products

| ProductId | Name     | Price |
| --------- | -------- | ----- |
| 1         | Laptop   | 1000  |
| 2         | Mouse    | 20    |
| 3         | Keyboard | 50    |

Now customer data exists only once.

---

# Normal Forms

Normalization is usually discussed in terms of **Normal Forms (NF)**.

---

## First Normal Form (1NF)

### Rule

- Each column contains atomic values.
- No repeating groups or arrays.

### Bad

| UserId | Phones  |
| ------ | ------- |
| 1      | 123,456 |

### Good

| UserId | Phone |
| ------ | ----- |
| 1      | 123   |
| 1      | 456   |

---

## Second Normal Form (2NF)

### Rule

- Must already be in 1NF.
- No partial dependency on a composite primary key.

Example:

| StudentId | CourseId | StudentName |
| --------- | -------- | ----------- |

Primary Key:

```text
(StudentId, CourseId)
```

Problem:

```text
StudentName
depends only on StudentId
```

not on the full key.

Move student information to a separate table.

---

## Third Normal Form (3NF)

### Rule

- Must already be in 2NF.
- No transitive dependencies.

Example:

| EmployeeId | DepartmentId | DepartmentName |
| ---------- | ------------ | -------------- |

Problem:

```text
EmployeeId
   ↓
DepartmentId
   ↓
DepartmentName
```

DepartmentName depends on DepartmentId, not directly on EmployeeId.

Split into:

### Employees

| EmployeeId | DepartmentId |

### Departments

| DepartmentId | DepartmentName |

---

# Before vs After Normalization

### Unnormalized

```text
Orders
 ├── CustomerName
 ├── CustomerEmail
 ├── ProductName
 └── Price
```

### Normalized

```text
Customers
     |
     |
Orders
     |
     |
Products
```

Each entity is stored separately.

---

# Advantages

### Reduced Data Redundancy

Less duplicate data.

### Better Data Integrity

One source of truth.

### Easier Updates

Update customer email once.

### Smaller Storage

Less duplicated information.

### Consistent Data

Fewer chances of conflicting values.

---

# Disadvantages

### More Joins

Data is spread across multiple tables.

```sql
SELECT *
FROM Orders o
JOIN Customers c
ON o.customerId = c.id;
```

More joins can increase query complexity.

### Slightly Slower Reads

Complex read-heavy workloads may require multiple table joins.

---

# Normalization vs Denormalization

| Normalization          | Denormalization                        |
| ---------------------- | -------------------------------------- |
| Reduces redundancy     | Introduces redundancy                  |
| Better consistency     | Better read performance                |
| More joins             | Fewer joins                            |
| Smaller storage        | Larger storage                         |
| Ideal for OLTP systems | Ideal for analytics/read-heavy systems |

---

# System Design Perspective

In system design interviews:

### Use Normalization For

- Banking systems
- Payment systems
- Inventory management
- Order management

Where consistency is critical.

### Use Denormalization For

- Social media feeds
- Recommendation systems
- Search indexes
- Analytics systems

Where read performance is more important than perfect normalization.

---

# Interview-Ready Summary

> Database normalization is the process of organizing data into related tables to minimize redundancy and improve data integrity. The most commonly used level is 3NF, where duplicate data and transitive dependencies are removed. Normalization improves consistency and maintainability but may require additional joins, which is why large-scale read-heavy systems sometimes use denormalization for better performance.

## Question 4. What is denormalization and when is it useful?

# Direct Answer

**Denormalization** is the process of intentionally introducing **redundant or duplicated data** into a database to improve **read performance** and reduce expensive joins.

It is essentially the opposite of normalization.

> Normalization optimizes for data consistency; denormalization optimizes for query performance.

---

# Why Denormalize?

Consider a normalized schema:

```text
Customers
---------
customer_id
name

Orders
------
order_id
customer_id
amount
```

To fetch orders with customer names:

```sql
SELECT o.order_id,
       o.amount,
       c.name
FROM Orders o
JOIN Customers c
ON o.customer_id = c.customer_id;
```

At small scale, this is fine.

At very large scale:

- Billions of rows
- Frequent reads
- Multiple joins

Joins can become expensive.

---

# Denormalized Design

Store customer information directly in the order record:

```text
Orders
------
order_id
customer_id
customer_name
amount
```

Now:

```sql
SELECT order_id,
       customer_name,
       amount
FROM Orders;
```

No join required.

---

# Common Denormalization Techniques

### 1. Duplicate Frequently Accessed Data

Normalized:

```text
Users
Orders
```

Denormalized:

```text
Order
 ├── orderId
 ├── customerId
 └── customerName
```

Customer name is duplicated.

---

### 2. Embed Related Data

Common in document databases.

```json
{
  "userId": 1,
  "name": "John",
  "orders": [
    {
      "orderId": 101,
      "amount": 100
    }
  ]
}
```

Everything can be fetched with a single read.

---

### 3. Precomputed Aggregates

Instead of computing repeatedly:

```sql
SELECT COUNT(*)
FROM Likes
WHERE postId = 123;
```

Store:

```text
Posts
------
post_id
like_count
```

Read becomes O(1).

---

# When Is Denormalization Useful?

## 1. Read-Heavy Systems

When reads greatly outnumber writes.

Examples:

- Social media feeds
- Product catalogs
- News feeds
- Search systems

Example:

```text
Reads: 1,000,000/sec
Writes: 5,000/sec
```

Optimizing reads is more valuable.

---

## 2. Large-Scale Distributed Systems

Cross-node joins are expensive.

Example:

```text
Shard 1 -> User Data
Shard 2 -> Order Data
```

A join may require network calls between shards.

Denormalization avoids these distributed joins.

---

## 3. Real-Time User Experiences

Users expect responses in milliseconds.

Examples:

- Instagram feed
- Facebook timeline
- YouTube homepage
- Amazon product page

Precomputed and duplicated data improves latency.

---

## 4. Analytics and Reporting

Data warehouses often use denormalized schemas such as star schemas.

```text
FactSales
   |
   +---- Product
   +---- Customer
   +---- Date
```

Designed for fast reporting queries.

---

# Trade-offs

| Benefit         | Cost                      |
| --------------- | ------------------------- |
| Faster reads    | Duplicate data            |
| Fewer joins     | More storage              |
| Simpler queries | Complex updates           |
| Lower latency   | Risk of inconsistent data |

---

# Example Trade-off

Suppose customer name changes.

Normalized:

```text
Customers
---------
John
```

Update one row.

Denormalized:

```text
Orders
-------
John
John
John
John
John
```

Need to update many records.

This introduces consistency challenges.

---

# Real-World Examples

### Social Media Feed

Instead of:

```text
Posts
Users
Comments
Likes
```

and joining at request time,

store a feed item like:

```json
{
  "postId": 123,
  "authorName": "John",
  "likeCount": 500,
  "commentCount": 20
}
```

Feed generation becomes much faster.

---

### E-commerce Product Page

Store:

```text
Product
 ├── rating
 ├── review_count
 ├── seller_name
 └── inventory_status
```

instead of calculating everything on every request.

---

# System Design Perspective

A common interview pattern:

### OLTP Database (Normalized)

```text
Users
Orders
Payments
Inventory
```

Purpose:

- Data correctness
- Transactions

---

### Read Model / Cache (Denormalized)

```text
UserProfileView
OrderSummaryView
FeedView
```

Purpose:

- Fast reads
- Low latency

This is common in:

- CQRS architectures
- Search systems
- Recommendation systems
- Feed generation systems

---

# Interview-Ready Summary

> Denormalization is the intentional duplication of data to reduce joins and improve read performance. It is useful in read-heavy, large-scale systems where low latency is more important than minimizing redundancy. The trade-off is increased storage usage and more complex updates, since duplicated data must be kept consistent. In practice, transactional systems are often normalized, while read-heavy systems such as feeds, search indexes, and analytics stores are denormalized.

## Question 5. What are the different types of NoSQL databases?

## Question 6. What is a graph database and when is it used?

## Question 7. What is database partitioning?

## Question 8. What is consistent hashing in system design?

## Question 9. How do you handle schema migrations in distributed databases?

## Question 10. How do you design a distributed transaction system?

## Question 11. What is REST API vs GraphQL vs gRPC?

## Question 12. What is WebSocket and when to use it?

## Question 13. How do you design a push notification system?

## Question 14. What is polling vs long-polling vs server-sent events?

## Question 15. What is API Gateway in microservices?

## Question 16. How do you ensure secure communication between services?

## Question 17. What is OAuth 2.0 and how it's used in system design?

## Question 18. How do you design an authentication system like OAuth or JWT?

## Question 19. What is CORS and why does it matter in system design?

## Question 20. What are retries and backoff strategies in networking?
