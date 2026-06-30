# Set 11

| S.No. | Question                                                                                                                                                      |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What is the difference between batch processing and stream processing?](#question-1-what-is-the-difference-between-batch-processing-and-stream-processing)   |
| 2.    | [Explain ETL (Extract, Transform, Load) in system design](#question-2-explain-etl-extract-transform-load-in-system-design)                                    |
| 3.    | [What is data warehousing and how does it differ from OLTP?](#question-3-what-is-data-warehousing-and-how-does-it-differ-from-oltp)                           |
| 4.    | [What is a distributed transaction?](#question-4-what-is-a-distributed-transaction)                                                                           |
| 5.    | [Explain the concept of ACID in databases](#question-5-explain-the-concept-of-acid-in-databases)                                                              |
| 6.    | [What is BASE in NoSQL systems?](#question-6-what-is-base-in-nosql-systems)                                                                                   |
| 7.    | [What is a data lake?](#question-7-what-is-a-data-lake)                                                                                                       |
| 8.    | [What is the difference between consistency models (strong, weak, causal)?](#question-8-what-is-the-difference-between-consistency-models-strong-weak-causal) |
| 9.    | [What is sticky partitioning in distributed systems?](#question-9-what-is-sticky-partitioning-in-distributed-systems)                                         |
| 10.   | [What is clock skew and how does it affect distributed systems?](#question-10-what-is-clock-skew-and-how-does-it-affect-distributed-systems)                  |
| 11.   | [How do you design a music streaming service like Spotify?](#question-11-how-do-you-design-a-music-streaming-service-like-spotify)                            |
| 12.   | [How do you design a podcast hosting platform?](#question-12-how-do-you-design-a-podcast-hosting-platform)                                                    |
| 13.   | [How do you design a live auction system like eBay?](#question-13-how-do-you-design-a-live-auction-system-like-ebay)                                          |
| 14.   | [How do you design a food recipe sharing platform?](#question-14-how-do-you-design-a-food-recipe-sharing-platform)                                            |
| 15.   | [How do you design a job portal like LinkedIn?](#question-15-how-do-you-design-a-job-portal-like-linkedin)                                                    |
| 16.   | [How do you design a Q&A platform like Quora/StackOverflow?](#question-16-how-do-you-design-a-qa-platform-like-quorastackoverflow)                            |
| 17.   | [How do you design a travel booking aggregator?](#question-17-how-do-you-design-a-travel-booking-aggregator)                                                  |
| 18.   | [How do you design a digital signature service like DocuSign?](#question-18-how-do-you-design-a-digital-signature-service-like-docusign)                      |
| 19.   | [How do you design a currency conversion system?](#question-19-how-do-you-design-a-currency-conversion-system)                                                |
| 20.   | [How do you design an API marketplace like RapidAPI?](#question-20-how-do-you-design-an-api-marketplace-like-rapidapi)                                        |

## Question 1. What is the difference between batch processing and stream processing?

# Difference Between Batch Processing and Stream Processing

## Direct answer

**Batch processing** processes a collection of data together at scheduled intervals, while **stream processing** processes data continuously as it arrives.

- **Batch processing** is optimized for high throughput and efficiency when real-time results are not required.
- **Stream processing** is optimized for low latency and real-time decision-making.

---

## Comparison

| Feature          | Batch Processing          | Stream Processing                                        |
| ---------------- | ------------------------- | -------------------------------------------------------- |
| Processing model | Process data in batches   | Process each event (or small micro-batches) continuously |
| Latency          | Minutes to hours          | Milliseconds to seconds                                  |
| Data arrival     | Historical data           | Continuous data stream                                   |
| Response time    | Delayed                   | Near real-time                                           |
| Throughput       | Very high                 | High, but optimized for low latency                      |
| Complexity       | Simpler                   | More complex                                             |
| Fault tolerance  | Easier to recover         | Requires checkpointing and state management              |
| Typical storage  | Data lake, warehouse      | Message queues, event logs                               |
| Best for         | Analytics, reporting, ETL | Monitoring, fraud detection, live dashboards             |

---

## How they work

### Batch Processing

```
Data Sources
     │
     ▼
Store Data
     │
     ▼
Run Scheduled Job
     │
     ▼
Process Entire Batch
     │
     ▼
Generate Results
```

Data accumulates first, then a scheduled job processes everything together.

**Example:**

- Sales data collected all day
- Generate daily revenue report at midnight

---

### Stream Processing

```
Producer
    │
    ▼
Message Queue
    │
    ▼
Streaming Engine
    │
    ▼
Immediate Processing
    │
    ▼
Live Dashboard / Alert / Database
```

Every incoming event is processed almost immediately.

**Example:**

- Credit card transaction arrives
- Fraud detection runs instantly
- Suspicious transaction is blocked within milliseconds

---

## Common use cases

### Batch Processing

- Daily sales reports
- Payroll processing
- ETL pipelines
- Billing systems
- Log aggregation
- Machine learning model training
- Monthly financial reports

**Example:**

```
Every night:
Read 100 million transactions
→ Aggregate
→ Produce report
```

---

### Stream Processing

- Fraud detection
- Live sports score updates
- Ride tracking
- Stock trading
- IoT sensor monitoring
- Real-time recommendations
- Social media feeds

**Example:**

```
Transaction received
→ Check fraud rules
→ Approve/Reject immediately
```

---

## Technologies

### Batch Processing

- Apache Hadoop
- Apache Spark (batch mode)
- SQL-based ETL jobs
- Data warehouses

---

### Stream Processing

- Apache Kafka
- Apache Flink
- Apache Spark Structured Streaming
- Apache Storm

---

## Advantages and disadvantages

### Batch Processing

**Advantages**

- Simple architecture
- Efficient for large datasets
- Lower infrastructure cost
- Easier retries and debugging

**Disadvantages**

- High latency
- No real-time insights
- Data becomes stale between runs

---

### Stream Processing

**Advantages**

- Real-time responses
- Immediate alerts
- Continuously updated dashboards
- Better user experience for live applications

**Disadvantages**

- More complex architecture
- Stateful processing is harder
- Requires careful handling of failures, ordering, and duplicate events
- Higher operational cost

---

## When to choose which?

Choose **Batch Processing** when:

- Real-time processing is unnecessary.
- Processing huge historical datasets.
- Running nightly ETL or analytics jobs.
- Cost efficiency is more important than latency.

Choose **Stream Processing** when:

- Low latency is critical.
- Detecting fraud or anomalies.
- Processing IoT or telemetry data.
- Building live dashboards or notifications.

---

## Can they be combined?

Yes. Many large-scale systems use both.

Example architecture:

```
Users
   │
   ▼
Kafka
   │
   ├────────► Stream Processing
   │              │
   │              ▼
   │        Live Dashboard
   │
   ▼
Data Lake
   │
   ▼
Batch Processing
   │
   ▼
Business Reports & Analytics
```

This is often called the **Lambda Architecture** pattern, where streaming provides immediate insights and batch processing recomputes accurate historical views.

---

## Interview-ready summary

> **Batch processing** handles large volumes of accumulated data at scheduled intervals, offering high throughput but higher latency. **Stream processing** handles events continuously as they arrive, enabling real-time analytics and actions with low latency. Batch is ideal for ETL, reporting, and analytics, while stream processing is best for fraud detection, live monitoring, IoT, and real-time user experiences. Many production systems combine both to balance throughput, latency, and accuracy.

## Question 2. Explain ETL (Extract, Transform, Load) in system design

# ETL (Extract, Transform, Load) in System Design

## Direct answer

**ETL (Extract, Transform, Load)** is a data integration process used to collect data from one or more source systems, clean and transform it into a consistent format, and load it into a destination such as a data warehouse or data lake for reporting, analytics, or machine learning.

It is a fundamental part of data engineering and business intelligence pipelines.

---

## What does ETL stand for?

### 1. Extract

The pipeline reads data from one or more source systems.

Typical sources include:

- Relational databases
- NoSQL databases
- Application APIs
- Log files
- CSV/Excel files
- Event streams
- Cloud storage

Example:

```text
Orders Database
Customers Database
Payment Service
Application Logs
        │
        ▼
     Extract Data
```

---

### 2. Transform

The extracted data is processed to make it consistent and usable.

Common transformations include:

- Cleaning invalid records
- Removing duplicates
- Standardizing formats (e.g., dates, currencies)
- Joining multiple datasets
- Aggregating metrics
- Validating business rules
- Masking sensitive data
- Calculating derived fields

Example:

Input:

```text
Order:
Price = "1000"
Currency = USD
```

Transform:

```text
Price_INR = 83,000
Tax = 18%
Final Amount = 97,940
```

---

### 3. Load

The transformed data is written to the destination system.

Common destinations:

- Data warehouse
- Data lake
- Analytics database
- Search index
- Reporting system

Example:

```text
Cleaned Data
      │
      ▼
Data Warehouse
      │
      ▼
BI Dashboard
```

---

## ETL workflow

```text
          Data Sources
    ┌────────┬────────┬────────┐
    │Orders  │Users   │Payments│
    └────────┴────────┴────────┘
             │
             ▼
        Extract Layer
             │
             ▼
     Transform & Validate
             │
             ▼
      Load to Warehouse
             │
             ▼
 Dashboards / Analytics / ML
```

---

## Example: E-commerce company

Suppose an online store wants a daily sales report.

### Step 1: Extract

Collect:

- Orders
- Customers
- Products
- Payments

### Step 2: Transform

- Remove duplicate orders
- Convert all timestamps to UTC
- Join orders with customer details
- Calculate total revenue
- Categorize products

### Step 3: Load

Store the processed data in the analytics warehouse.

Business users can now run queries like:

```sql
Revenue by country
Top-selling products
Daily active customers
Average order value
```

---

## Why ETL is important

Without ETL:

```text
Orders DB
Customers DB
Payments DB
Inventory DB

Different schemas
Different formats
Duplicate records
Missing values
```

Analytics become slow and inconsistent.

With ETL:

```text
Unified Clean Dataset

Consistent
Validated
Fast to query
Optimized for reporting
```

---

## Batch ETL vs Streaming ETL

| Feature    | Batch ETL        | Streaming ETL           |
| ---------- | ---------------- | ----------------------- |
| Processing | Scheduled        | Continuous              |
| Latency    | Minutes to hours | Milliseconds to seconds |
| Use case   | Nightly reports  | Live dashboards         |
| Cost       | Lower            | Higher                  |
| Complexity | Simpler          | More complex            |

Example:

**Batch**

```text
Run every midnight
```

**Streaming**

```text
Process each order immediately
```

---

## ETL vs ELT

| ETL                                       | ELT                                             |
| ----------------------------------------- | ----------------------------------------------- |
| Transform before loading                  | Load before transforming                        |
| Requires a separate processing engine     | Uses the destination system for transformations |
| Better when storage or compute is limited | Well suited to modern cloud data warehouses     |
| Traditional data warehousing approach     | Common in cloud-native architectures            |

### ETL

```text
Extract
   │
Transform
   │
Load
```

### ELT

```text
Extract
   │
Load
   │
Transform
```

Modern cloud platforms often favor ELT because they can efficiently transform large datasets after loading.

---

## Scalability considerations

In large-scale systems, ETL pipelines are designed to handle growing data volumes efficiently:

- **Parallel extraction** from multiple sources.
- **Partitioned processing** (e.g., by date or region).
- **Distributed transformation** across clusters.
- **Incremental loads** using change data capture (CDC) instead of full reloads.
- **Retry and checkpointing** to recover from failures.
- **Workflow orchestration** to manage dependencies and scheduling.

---

## Common challenges

| Challenge           | Solution                            |
| ------------------- | ----------------------------------- |
| Large datasets      | Distributed processing              |
| Duplicate records   | Deduplication using unique keys     |
| Schema changes      | Schema evolution and versioning     |
| Data quality issues | Validation and cleansing rules      |
| Failed jobs         | Retry mechanisms and checkpointing  |
| Late-arriving data  | Watermarks and reprocessing windows |

---

## Popular ETL technologies

Some widely used ETL and data processing tools include:

- Apache Spark
- Apache Airflow
- Apache NiFi
- Apache Kafka (for streaming ETL)
- dbt (commonly used in ELT workflows)

---

## Interview-ready summary

> **ETL (Extract, Transform, Load)** is a data pipeline pattern that extracts data from multiple sources, transforms it into a clean and consistent format, and loads it into an analytics system. It's widely used for reporting, business intelligence, and machine learning. Traditional ETL performs transformations before loading, while modern cloud architectures often use ELT, where raw data is loaded first and transformed within the data warehouse. In large-scale systems, ETL pipelines emphasize parallel processing, incremental updates, fault tolerance, and data quality.

## Question 3. What is data warehousing and how does it differ from OLTP?

# Data Warehousing vs OLTP

## Direct answer

A **data warehouse** is a centralized repository designed for **analytics, reporting, and business intelligence (OLAP)**. It stores large volumes of historical data from multiple sources and is optimized for complex read-heavy queries.

An **OLTP (Online Transaction Processing)** system is designed to **handle day-to-day business transactions** such as creating orders, processing payments, or updating inventory. It is optimized for fast, reliable, concurrent reads and writes.

In short:

- **OLTP = Run the business**
- **Data Warehouse (OLAP) = Analyze the business**

---

## Purpose

### OLTP

Supports operational applications.

Examples:

- Placing an order
- Booking a ticket
- Banking transactions
- Updating inventory
- User login

Characteristics:

- Thousands to millions of small transactions
- Low latency
- High concurrency
- Strong consistency

---

### Data Warehouse

Supports analytics and decision-making.

Examples:

- Monthly revenue reports
- Sales trends
- Customer segmentation
- Forecasting
- Executive dashboards

Characteristics:

- Large analytical queries
- Historical data
- Read-heavy workloads
- Complex aggregations

---

## Architecture

### OLTP System

```text
Users
   │
   ▼
Application Servers
   │
   ▼
Operational Database
```

The application reads and writes directly to the operational database.

---

### Data Warehouse

```text
Operational DBs
CRM
Payment System
Logs
Inventory
      │
      ▼
ETL / ELT Pipeline
      │
      ▼
Data Warehouse
      │
      ▼
BI Dashboards
Data Analysts
ML Models
```

Data from multiple operational systems is periodically or continuously loaded into the warehouse.

---

## Comparison

| Feature       | OLTP                       | Data Warehouse (OLAP)                        |
| ------------- | -------------------------- | -------------------------------------------- |
| Purpose       | Process transactions       | Analytics and reporting                      |
| Data          | Current operational data   | Historical integrated data                   |
| Workload      | Read + Write               | Mostly Read                                  |
| Query type    | Simple CRUD operations     | Complex aggregations and joins               |
| Response time | Milliseconds               | Seconds to minutes (depending on query size) |
| Users         | Applications and end users | Analysts, BI tools, data scientists          |
| Data volume   | Current records            | Years of historical data                     |
| Schema        | Highly normalized          | Often denormalized (star/snowflake schemas)  |
| Updates       | Continuous                 | Periodic batch or streaming loads            |
| Optimization  | Fast transactions          | Fast analytical queries                      |

---

## Example

### OLTP Database

An e-commerce application stores each order.

```text
Orders

OrderID
CustomerID
ProductID
Quantity
Price
Timestamp
```

Typical query:

```sql
SELECT * FROM Orders
WHERE OrderID = 12345;
```

Returns one record very quickly.

---

### Data Warehouse

Stores years of sales history.

Typical query:

```sql
Revenue by country
for the last five years
grouped by month
```

This query may scan billions of rows and aggregate large datasets efficiently.

---

## Why not run analytics on OLTP?

Suppose an online shopping site receives:

- 20,000 orders per second
- Millions of customers browsing

If an analyst runs:

```sql
SELECT
Country,
SUM(Sales)
FROM Orders
GROUP BY Country;
```

on the production database:

- Large table scans consume CPU and I/O.
- Customer transactions slow down.
- User experience degrades.
- Lock contention may increase (depending on the database and isolation level).

A separate data warehouse isolates analytical workloads from transactional workloads.

---

## Data flow

```text
Users
   │
   ▼
OLTP Database
   │
   ▼
ETL / ELT
   │
   ▼
Data Warehouse
   │
   ▼
Reports
Dashboards
ML
```

This separation allows each system to be optimized for its specific workload.

---

## Schema design

### OLTP

Typically uses **normalized** schemas to reduce redundancy and maintain data integrity.

Example:

```text
Customers
Orders
Products
Payments
```

Benefits:

- Minimal duplication
- Efficient updates
- Strong referential integrity

---

### Data Warehouse

Often uses **denormalized** schemas to speed up analytical queries.

Common models:

- **Star Schema**: One central fact table connected to multiple dimension tables.
- **Snowflake Schema**: Dimension tables are further normalized.

Example:

```text
          Product
             │
Customer ─ Sales Fact ─ Date
             │
          Store
```

Benefits:

- Fewer joins
- Faster aggregations
- Better analytical performance

---

## Design considerations

A well-designed data warehouse typically includes:

- **Columnar storage** for efficient scans of selected columns.
- **Partitioning** (often by date) to reduce the amount of data scanned.
- **Compression** to lower storage costs and improve I/O.
- **Materialized views** or precomputed aggregates for frequently used reports.
- **Incremental ETL/ELT** using Change Data Capture (CDC) to avoid full reloads.
- **Data quality checks** to validate incoming data before analysis.

---

## Trade-offs

| OLTP                            | Data Warehouse                             |
| ------------------------------- | ------------------------------------------ |
| Excellent for transactions      | Poor for transactional updates             |
| Highly consistent               | Data may be slightly delayed due to ETL    |
| Supports many concurrent writes | Optimized for large scans and aggregations |
| Smaller working dataset         | Stores massive historical datasets         |

---

## Interview-ready summary

> **OLTP systems** power day-to-day business operations and are optimized for fast, concurrent transactional workloads using normalized schemas. **Data warehouses (OLAP)** consolidate historical data from multiple sources through ETL/ELT pipelines and are optimized for analytical queries using denormalized schemas. Separating OLTP and OLAP workloads improves application performance while enabling efficient reporting, business intelligence, and machine learning on large historical datasets.

## Question 4. What is a distributed transaction?

# Distributed Transaction

## Direct answer

A **distributed transaction** is a transaction that spans **multiple independent systems** (such as databases, microservices, or data centers) and must maintain the **ACID** properties across all of them.

The main challenge is ensuring that **either all participating systems commit the transaction or all roll it back**, even in the presence of failures.

---

## Why are distributed transactions needed?

In a monolithic application, a single database transaction is straightforward:

```text
Application
     │
     ▼
Single Database
     │
BEGIN
UPDATE A
UPDATE B
COMMIT
```

The database guarantees atomicity.

In a distributed system:

```text
Order Service ─────► Orders DB
      │
      ├────────────► Payment Service ─► Payments DB
      │
      └────────────► Inventory Service ─► Inventory DB
```

A single business operation now involves multiple services and databases. If one succeeds and another fails, the system can become inconsistent unless coordination is used.

---

## Example

Consider placing an order on an e-commerce platform:

1. Create the order.
2. Charge the customer's credit card.
3. Reserve inventory.
4. Schedule shipping.

Desired outcome:

```text
✓ Order created
✓ Payment charged
✓ Inventory reserved
✓ Shipping scheduled
```

Failure scenario:

```text
✓ Payment charged
✓ Order created
✗ Inventory reservation failed
```

Without coordination, the customer is charged for an order that cannot be fulfilled.

---

## Common approaches

### 1. Two-Phase Commit (2PC)

A coordinator manages the transaction in two phases.

#### Phase 1: Prepare

```text
Coordinator
     │
     ├──► Payment: Prepare?
     ├──► Inventory: Prepare?
     └──► Order: Prepare?
```

Each participant:

- Executes the operation without committing.
- Writes enough information to recover.
- Replies **YES** or **NO**.

#### Phase 2: Commit or Abort

If everyone votes YES:

```text
Coordinator
     │
     ├──► Commit Payment
     ├──► Commit Inventory
     └──► Commit Order
```

If any participant votes NO:

```text
Coordinator
     │
     ├──► Rollback Payment
     ├──► Rollback Inventory
     └──► Rollback Order
```

### Advantages

- Strong consistency.
- Guarantees atomicity across participants.

### Disadvantages

- Coordinator can become a bottleneck.
- Participants may remain blocked while waiting for the coordinator.
- Increased latency due to multiple network round trips.
- Doesn't scale well in highly distributed microservice architectures.

---

### 2. Saga Pattern

Instead of one global transaction, a business workflow is split into **local transactions**.

Each service:

1. Commits its own transaction.
2. Publishes an event.
3. If a later step fails, previously completed steps execute **compensating transactions**.

Example:

```text
Create Order
      │
      ▼
Charge Payment
      │
      ▼
Reserve Inventory
      │
      ▼
Ship Order
```

If inventory reservation fails:

```text
Refund Payment
Cancel Order
```

### Advantages

- High scalability.
- No blocking coordinator.
- Well suited for microservices.
- Better availability.

### Disadvantages

- Temporary inconsistency is possible.
- Requires carefully designed compensation logic.
- More complex business workflows.

---

## 2PC vs Saga

| Feature          | Two-Phase Commit                  | Saga          |
| ---------------- | --------------------------------- | ------------- |
| Consistency      | Strong                            | Eventual      |
| Blocking         | Yes                               | No            |
| Scalability      | Moderate                          | High          |
| Latency          | Higher                            | Lower         |
| Failure handling | Rollback                          | Compensation  |
| Best for         | Traditional distributed databases | Microservices |

---

## Design considerations

When designing distributed transactions, consider:

- **Idempotency:** Retrying a request should not create duplicate effects.
- **Timeouts:** Handle participants that become slow or unavailable.
- **Retries:** Use exponential backoff for transient failures.
- **Dead-letter queues:** Capture events that repeatedly fail.
- **Correlation IDs:** Track a transaction across services for debugging.
- **Monitoring and tracing:** Observe the end-to-end transaction lifecycle.

---

## Trade-offs

| Strong consistency (2PC)           | Eventual consistency (Saga)         |
| ---------------------------------- | ----------------------------------- |
| Guarantees all-or-nothing behavior | Better availability and scalability |
| Higher latency                     | Faster local commits                |
| Blocking during failures           | No global locks                     |
| Simpler recovery semantics         | Requires compensation logic         |

In modern cloud-native systems, the trade-off often favors **availability and scalability**, making the Saga pattern more common than distributed locking protocols.

---

## Real-world examples

### Use distributed transactions (or equivalent coordination) for:

- Bank fund transfers
- Financial ledger updates
- Airline seat booking with strict consistency requirements

### Prefer Saga for:

- E-commerce order processing
- Food delivery workflows
- Ride-sharing trip lifecycle
- Hotel reservation systems

---

## Interview-ready summary

> A **distributed transaction** coordinates a single business operation across multiple databases or services while preserving consistency. The traditional solution is **Two-Phase Commit (2PC)**, which provides strong consistency but introduces blocking and scalability challenges. Modern microservice architectures typically prefer the **Saga pattern**, where each service performs a local transaction and failures are handled through compensating actions. This sacrifices immediate consistency for better scalability, availability, and fault tolerance, making Saga the more common choice in large-scale distributed systems.

## Question 5. Explain the concept of ACID in databases

# ACID in Databases

## Direct answer

**ACID** is a set of four properties that guarantee reliable and correct database transactions, even in the presence of failures such as crashes, power outages, or concurrent access.

ACID stands for:

- **A** – Atomicity
- **C** – Consistency
- **I** – Isolation
- **D** – Durability

Together, these properties ensure that transactions are processed safely and maintain data integrity.

---

## What is a transaction?

A **transaction** is a sequence of database operations treated as a single unit of work.

Example: Bank transfer of ₹500

```text
Account A = ₹5000
Account B = ₹2000

Transaction:
1. Debit ₹500 from A
2. Credit ₹500 to B
```

Both operations must succeed together.

---

## 1. Atomicity

### Definition

**Atomicity** means **all operations in a transaction succeed, or none of them do**.

There is no partial completion.

### Example

```text
BEGIN

Debit A ₹500
Credit B ₹500

COMMIT
```

If the system crashes after debiting A but before crediting B:

Without Atomicity:

```text
A = ₹4500
B = ₹2000
```

Money is lost.

With Atomicity:

```text
ROLLBACK

A = ₹5000
B = ₹2000
```

The database undoes all changes, leaving the data unchanged.

### Key idea

> A transaction is **all-or-nothing**.

---

## 2. Consistency

### Definition

**Consistency** ensures that a transaction moves the database from **one valid state to another valid state**, preserving all integrity constraints and business rules.

### Example

Suppose a rule states:

```text
Account balance cannot be negative.
```

Transaction:

```text
Withdraw ₹6000
Current Balance = ₹5000
```

The database rejects the transaction because it would violate the constraint.

### Before

```text
Balance = ₹5000
```

### After

```text
Still ₹5000
```

No invalid state is committed.

### Key idea

> Every committed transaction must satisfy database constraints.

---

## 3. Isolation

### Definition

**Isolation** ensures that **concurrent transactions do not interfere with each other**. Each transaction behaves as if it is executing alone.

### Example

Suppose two users try to buy the last item in stock.

Current inventory:

```text
Stock = 1
```

Two transactions:

```text
T1: Buy item
T2: Buy item
```

Without isolation:

```text
T1 reads Stock = 1
T2 reads Stock = 1

T1 updates Stock = 0
T2 updates Stock = 0
```

Two customers successfully purchase one item.

With proper isolation:

```text
T1 completes first
T2 waits

T2 reads Stock = 0
Purchase rejected
```

Only one purchase succeeds.

### Key idea

> Concurrent transactions should not produce incorrect results.

---

## 4. Durability

### Definition

Once a transaction is **committed**, its changes are **permanent**, even if the system crashes immediately afterward.

### Example

```text
Transfer completed
COMMIT
```

Immediately after commit:

```text
Power failure
```

After the database restarts:

```text
Transfer is still present.
```

The committed data survives because it has been persisted (typically using mechanisms like transaction logs and recovery).

### Key idea

> A committed transaction is never lost.

---

## ACID summary

| Property    | Meaning                                       | Example                                |
| ----------- | --------------------------------------------- | -------------------------------------- |
| Atomicity   | All operations succeed or all fail            | Money isn't partially transferred      |
| Consistency | Database remains valid after each transaction | No negative balances                   |
| Isolation   | Concurrent transactions don't interfere       | Two users can't buy the same last item |
| Durability  | Committed changes survive crashes             | Data remains after power failure       |

---

## Real-world example

Online banking transfer:

```text
Transfer ₹1000

1. Debit Account A
2. Credit Account B
3. Commit
```

How ACID applies:

- **Atomicity:** Both debit and credit happen together.
- **Consistency:** Total money in the system remains unchanged.
- **Isolation:** Simultaneous transfers don't corrupt balances.
- **Durability:** Once confirmed, the transfer persists despite failures.

---

## ACID vs BASE

Modern distributed systems sometimes relax ACID guarantees to improve scalability and availability.

| ACID                           | BASE                                     |
| ------------------------------ | ---------------------------------------- |
| Strong consistency             | Eventual consistency                     |
| Immediate correctness          | Temporary inconsistency allowed          |
| Common in relational databases | Common in distributed NoSQL systems      |
| Prioritizes data integrity     | Prioritizes availability and scalability |

Examples:

- **ACID:** Banking, payment systems, inventory management.
- **BASE:** Social media feeds, analytics, recommendation systems.

---

## Trade-offs

Strong ACID guarantees provide excellent data integrity but can reduce scalability in distributed systems:

- **Atomicity** may require distributed coordination (e.g., two-phase commit).
- **Isolation** can reduce throughput because transactions may wait for locks or conflict resolution.
- **Durability** adds write latency since data must be persisted before acknowledging success.
- Distributed systems often relax consistency or isolation where temporary inconsistencies are acceptable to achieve better performance and availability.

---

## Interview-ready summary

> **ACID** is a set of transaction guarantees that ensure reliable database operations. **Atomicity** ensures all-or-nothing execution, **Consistency** ensures every transaction preserves database rules, **Isolation** prevents concurrent transactions from interfering with one another, and **Durability** guarantees that committed data survives failures. ACID is essential for systems like banking, payments, and inventory management, where correctness is more important than maximizing scalability.

## Question 6. What is BASE in NoSQL systems?

## Question 7. What is a data lake?

## Question 8. What is the difference between consistency models (strong, weak, causal)?

## Question 9. What is sticky partitioning in distributed systems?

## Question 10. What is clock skew and how does it affect distributed systems?

## Question 11. How do you design a music streaming service like Spotify?

## Question 12. How do you design a podcast hosting platform?

## Question 13. How do you design a live auction system like eBay?

## Question 14. How do you design a food recipe sharing platform?

## Question 15. How do you design a job portal like LinkedIn?

## Question 16. How do you design a Q&A platform like Quora/StackOverflow?

## Question 17. How do you design a travel booking aggregator?

## Question 18. How do you design a digital signature service like DocuSign?

## Question 19. How do you design a currency conversion system?

## Question 20. How do you design an API marketplace like RapidAPI?
