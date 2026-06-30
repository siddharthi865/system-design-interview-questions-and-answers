# Set 6

| S.No. | Question                                                                                                                                                              |
| ----- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What is the difference between throughput and bandwidth?](#question-1-what-is-the-difference-between-throughput-and-bandwidth)                                       |
| 2.    | [What are bottlenecks in a system? How do you identify and fix them?](#question-2-what-are-bottlenecks-in-a-system-how-do-you-identify-and-fix-them)                  |
| 3.    | [What is a Single Point of Failure (SPOF)?](#question-3-what-is-a-single-point-of-failure-spof)                                                                       |
| 4.    | [What is the difference between synchronous and asynchronous communication?](#question-4-what-is-the-difference-between-synchronous-and-asynchronous-communication)   |
| 5.    | [What is the difference between online (OLTP) and analytical (OLAP) databases?](#question-5-what-is-the-difference-between-online-oltp-and-analytical-olap-databases) |
| 6.    | [What is a quorum in distributed systems?](#question-6-what-is-a-quorum-in-distributed-systems)                                                                       |
| 7.    | [What is data locality and why is it important?](#question-7-what-is-data-locality-and-why-is-it-important)                                                           |
| 8.    | [What is a proxy server? How does it help in scaling?](#question-8-what-is-a-proxy-server-how-does-it-help-in-scaling)                                                |
| 9.    | [Difference between reverse proxy and forward proxy](#question-9-difference-between-reverse-proxy-and-forward-proxy)                                                  |
| 10.   | [What is edge computing? How does it affect system design?](#question-10-what-is-edge-computing-how-does-it-affect-system-design)                                     |
| 11.   | [How do you design an online multiplayer game system?](#question-11-how-do-you-design-an-online-multiplayer-game-system)                                              |
| 12.   | [How do you design an online exam/test-taking platform?](#question-12-how-do-you-design-an-online-examtest-taking-platform)                                           |
| 13.   | [How do you design a real-time bidding system (like Google Ads)?](#question-13-how-do-you-design-a-real-time-bidding-system-like-google-ads)                          |
| 14.   | [How do you design a digital wallet system like Paytm/PhonePe?](#question-14-how-do-you-design-a-digital-wallet-system-like-paytmphonepe)                             |
| 15.   | [How do you design a microblogging platform like Twitter?](#question-15-how-do-you-design-a-microblogging-platform-like-twitter)                                      |
| 16.   | [How do you design a logging system for billions of events per day?](#question-16-how-do-you-design-a-logging-system-for-billions-of-events-per-day)                  |
| 17.   | [How do you design a recommendation system like Netflix?](#question-17-how-do-you-design-a-recommendation-system-like-netflix)                                        |
| 18.   | [How do you design a content moderation system for social media?](#question-18-how-do-you-design-a-content-moderation-system-for-social-media)                        |
| 19.   | [How do you design a search engine like Google?](#question-19-how-do-you-design-a-search-engine-like-google)                                                          |
| 20.   | [How do you design a collaborative whiteboard tool?](#question-20-how-do-you-design-a-collaborative-whiteboard-tool)                                                  |

## Question 1. What is the difference between throughput and bandwidth?

## Direct answer

**Bandwidth** is the **maximum amount of data that can be transmitted over a network or communication channel per unit time**, while **throughput** is the **actual amount of data successfully transmitted per unit time**.

Think of it this way:

- **Bandwidth = Capacity**
- **Throughput = Actual achieved performance**

Throughput is almost always **less than or equal to bandwidth** due to protocol overhead, congestion, packet loss, latency, and hardware limitations.

---

## Intuition

Imagine a highway:

- **Bandwidth** = Number of lanes on the highway.
- **Throughput** = Number of cars that actually reach the destination every minute.

Even with an 8-lane highway (high bandwidth), traffic jams or accidents can reduce the number of cars arriving (lower throughput).

---

## Comparison

| Aspect     | Bandwidth                    | Throughput                                           |
| ---------- | ---------------------------- | ---------------------------------------------------- |
| Meaning    | Maximum theoretical capacity | Actual successful data transfer rate                 |
| Nature     | Theoretical/Configured       | Measured in real-world conditions                    |
| Unit       | bps (Mbps, Gbps)             | bps (Mbps, Gbps)                                     |
| Depends on | Link capacity                | Network conditions, congestion, latency, packet loss |
| Value      | Upper limit                  | Always ≤ Bandwidth                                   |

---

## Example

Suppose you have:

- Network link = **1 Gbps bandwidth**

### Ideal case

- No congestion
- No packet loss
- Efficient protocol

```
Bandwidth = 1 Gbps
Throughput ≈ 980 Mbps
```

(The small difference is due to protocol headers and overhead.)

### Congested network

```
Bandwidth = 1 Gbps
Throughput = 350 Mbps
```

Although the link can carry 1 Gbps, congestion limits the actual transfer rate.

---

## What affects throughput?

Several factors reduce throughput:

- **Network congestion**
- **Packet loss and retransmissions**
- **High latency (especially TCP connections)**
- **Protocol overhead (TCP/IP headers, TLS, etc.)**
- **Server processing capacity**
- **Client processing speed**
- **Disk read/write speed**
- **Load balancer or firewall bottlenecks**

---

## System design relevance

In system design interviews, understanding the distinction helps identify bottlenecks:

- **Bandwidth bottleneck:** The network link itself cannot carry more data.
- **Throughput bottleneck:** The system is unable to utilize the available bandwidth due to software, infrastructure, or network inefficiencies.

For example:

- A storage service may have a **10 Gbps network interface** (high bandwidth).
- But if the database can only read data at **2 Gbps**, the overall throughput is limited to **2 Gbps**.

---

## Related concept: Throughput vs Latency

These are often discussed together but measure different things.

| Metric         | Measures                                                     |
| -------------- | ------------------------------------------------------------ |
| **Latency**    | Time taken for a single request to complete                  |
| **Throughput** | Number of requests or amount of data processed per unit time |

Example:

- API latency = **50 ms/request**
- Throughput = **20,000 requests/second**

A system can have low latency but low throughput (fast individual requests, limited parallelism), or high throughput but higher latency (many concurrent requests with longer wait times).

---

## Interview-ready summary

> **Bandwidth is the maximum theoretical capacity of a communication channel, whereas throughput is the actual amount of data successfully transferred over that channel. Throughput is always less than or equal to bandwidth because of factors like congestion, latency, protocol overhead, and packet loss. In system design, bandwidth represents the network's potential, while throughput reflects the system's real-world performance.**

## Question 2. What are bottlenecks in a system? How do you identify and fix them?

# Direct answer

A **bottleneck** is any component in a system that limits the overall performance, scalability, or throughput of the entire system. Even if other components are fast, the slowest component determines the system's maximum performance.

A common interview principle is:

> **A system is only as fast as its slowest component.**

---

# Intuition

Imagine an assembly line in a factory:

- Machine A processes **1,000 items/min**
- Machine B processes **200 items/min**
- Machine C processes **800 items/min**

Although two machines are very fast, the entire factory can only produce **200 items/min** because Machine B is the bottleneck.

The same principle applies to distributed systems.

---

# Common bottlenecks in distributed systems

| Component          | Possible bottleneck                                              |
| ------------------ | ---------------------------------------------------------------- |
| CPU                | High computation, encryption, compression                        |
| Memory             | Insufficient RAM causing swapping or frequent garbage collection |
| Disk I/O           | Slow reads/writes, HDD limitations                               |
| Network            | Limited bandwidth, high latency, packet loss                     |
| Database           | Slow queries, table scans, lock contention                       |
| Cache              | Low hit rate, overloaded cache server                            |
| Load Balancer      | Uneven traffic distribution                                      |
| Application Server | Too few instances, thread pool exhaustion                        |
| Message Queue      | Slow consumers, queue backlog                                    |
| External APIs      | High response times, rate limits                                 |

---

# How to identify bottlenecks

A systematic approach is to measure every layer instead of guessing.

### 1. Monitor system metrics

Track metrics such as:

- CPU utilization
- Memory usage
- Disk I/O
- Network throughput
- Request latency
- Error rate
- Requests per second (RPS)
- Queue length

Example:

```
CPU = 95%
Memory = 40%
Disk = 20%
```

This strongly suggests the CPU is the bottleneck.

---

### 2. Analyze latency

Break down request latency into stages.

Example:

```
Client
   ↓
Load Balancer      2 ms
   ↓
API Server         8 ms
   ↓
Database          120 ms
   ↓
Response
```

Since the database dominates the request time, it is the likely bottleneck.

---

### 3. Use distributed tracing

Tracing tools show where requests spend time across services.

Typical flow:

```
User Request
      │
      ▼
API Gateway
      │
      ▼
User Service
      │
      ▼
Database
```

If one service consistently has much higher latency, it is a candidate bottleneck.

---

### 4. Monitor queue lengths

Growing queues indicate downstream components cannot keep up.

Example:

```
Incoming requests = 10,000/sec

Processing capacity = 6,000/sec

Queue keeps growing
```

The consumer or processing service is the bottleneck.

---

### 5. Load testing

Gradually increase traffic until performance degrades.

Observe:

- Response time
- Throughput
- CPU usage
- Error rate

The first component to saturate often reveals the bottleneck.

---

# How to fix bottlenecks

The solution depends on the limiting resource.

| Bottleneck    | Common fixes                                                    |
| ------------- | --------------------------------------------------------------- |
| CPU           | Optimize algorithms, add more instances, parallelize work       |
| Memory        | Increase RAM, reduce object allocation, tune garbage collection |
| Disk          | Use SSDs, batching, asynchronous writes                         |
| Network       | Compression, CDNs, larger bandwidth, reduce payload size        |
| Database      | Add indexes, optimize queries, caching, replication, sharding   |
| Cache         | Increase cache size, improve cache keys, raise hit rate         |
| Load Balancer | Better routing algorithms, autoscaling                          |
| Queue         | Add more consumers, partition queues, increase parallelism      |

---

# Example: Database bottleneck

Architecture:

```
Users
   │
   ▼
API Servers
   │
   ▼
Database
```

Problem:

```
Database CPU = 98%

API CPU = 20%

Network = Normal
```

The database is saturated.

Possible fixes:

- Add indexes
- Optimize slow queries
- Introduce a cache
- Add read replicas
- Partition (shard) data
- Batch writes
- Move analytics to separate systems

---

# Example: Network bottleneck

Suppose an image service sends 20 MB images.

```
Bandwidth = 1 Gbps

Average image = 20 MB
```

Fixes:

- Compress images
- Resize images
- Use a CDN
- Serve images closer to users
- Upgrade network capacity

---

# Example: CPU bottleneck

```
CPU = 100%

Memory = 35%

Network = 20%
```

Possible solutions:

- Optimize expensive computations
- Cache repeated results
- Use asynchronous processing
- Scale horizontally
- Move heavy tasks to worker services

---

# Best practices

- **Measure before optimizing**—avoid guessing.
- Optimize the biggest bottleneck first.
- Re-measure after every change.
- Continuously monitor production systems.
- Use autoscaling for predictable load increases.
- Cache frequently accessed data.
- Design services to scale independently.

---

# Trade-offs

| Solution                | Advantages                             | Disadvantages                                         |
| ----------------------- | -------------------------------------- | ----------------------------------------------------- |
| Vertical scaling        | Simple to implement                    | Hardware limits, higher cost                          |
| Horizontal scaling      | Better scalability and fault tolerance | Increased complexity                                  |
| Caching                 | Reduces latency and database load      | Cache invalidation challenges                         |
| Sharding                | Scales database capacity               | More complex queries and operations                   |
| Asynchronous processing | Improves responsiveness                | Added operational complexity and eventual consistency |

---

# Interview-ready summary

> **A bottleneck is the component that limits a system's overall performance or scalability. To identify it, monitor key metrics like CPU, memory, disk I/O, network, latency, and queue lengths, and use load testing and distributed tracing to pinpoint where requests spend the most time. Once identified, fix the bottleneck by optimizing code, adding caching, scaling horizontally, tuning databases, or increasing parallelism. Since removing one bottleneck often exposes the next, performance tuning is an iterative process of measure, optimize, and re-measure.**

## Question 3. What is a Single Point of Failure (SPOF)?

# Direct answer

A **Single Point of Failure (SPOF)** is any component in a system whose failure causes the **entire system or a critical part of it to become unavailable** because there is no redundant or backup component.

In distributed systems, identifying and eliminating SPOFs is essential for achieving **high availability** and **fault tolerance**.

---

# Intuition

Imagine a bridge connecting an island to the mainland.

- If there's **only one bridge**, and it collapses, no one can enter or leave the island.
- That bridge is a **Single Point of Failure**.

If there are **multiple bridges**, traffic can continue even if one fails.

---

# Example

Consider this architecture:

```text
         Users
           │
           ▼
     Load Balancer
           │
           ▼
      Application Server
           │
           ▼
        Database
```

Suppose there is:

- One load balancer
- One application server
- One database

If the database crashes:

```text
Database ❌
```

The entire application stops working.

The database is a **Single Point of Failure**.

---

# Common SPOFs in distributed systems

| Component                 | Why it's a SPOF                                    |
| ------------------------- | -------------------------------------------------- |
| Single database           | Data becomes unavailable if it fails               |
| Single application server | No server to handle requests                       |
| Single load balancer      | All incoming traffic is blocked                    |
| Single cache server       | Cache becomes unavailable, increasing backend load |
| Single message broker     | Producers and consumers cannot communicate         |
| Single DNS server         | Clients cannot resolve the service                 |
| Single network link       | Loss of connectivity                               |
| Single data center        | Regional outage affects the entire system          |

---

# How to eliminate SPOFs

The general strategy is to add **redundancy** so another component can take over if one fails.

### 1. Multiple application servers

Instead of:

```text
Users
   │
   ▼
App Server
```

Use:

```text
           Users
             │
             ▼
       Load Balancer
        /         \
       ▼           ▼
   App Server 1  App Server 2
```

If one server fails, the other continues serving requests.

---

### 2. Database replication

Instead of one database:

```text
App
 │
 ▼
Database
```

Use:

```text
             Primary
               │
        ┌──────┴──────┐
        ▼             ▼
    Replica 1     Replica 2
```

If the primary fails, a replica can be promoted.

---

### 3. Multiple load balancers

A load balancer itself can become a SPOF.

Solution:

```text
           DNS
         /     \
        ▼       ▼
      LB1     LB2
        \     /
         ▼   ▼
      Application Servers
```

---

### 4. Multi-region deployment

Instead of one data center:

```text
Region A
```

Deploy:

```text
Region A

Region B
```

If one region experiences an outage, traffic is routed to the other.

---

# Detecting SPOFs

Ask these questions for every critical component:

- What happens if this component fails?
- Is there another instance that can immediately take over?
- Can traffic be automatically redirected?
- Is the failover automatic or manual?
- Does data remain available during failure?

If the answer is **"the system stops working"**, you've found a SPOF.

---

# SPOF vs Bottleneck

| Aspect   | Single Point of Failure | Bottleneck              |
| -------- | ----------------------- | ----------------------- |
| Problem  | Availability            | Performance             |
| Effect   | System outage           | Slow system             |
| Cause    | No redundancy           | Limited capacity        |
| Solution | Redundancy and failover | Optimization or scaling |

A component can be **both** a bottleneck and a SPOF. For example, a single database handling all traffic may both limit throughput and cause a complete outage if it fails.

---

# Trade-offs

| Approach                 | Advantages                        | Disadvantages                                           |
| ------------------------ | --------------------------------- | ------------------------------------------------------- |
| Redundant servers        | Higher availability               | Increased infrastructure cost                           |
| Active-passive failover  | Simpler management                | Standby resources may be underutilized                  |
| Active-active deployment | Better utilization and resilience | More complex synchronization and routing                |
| Multi-region deployment  | Survives regional outages         | Higher cost, added latency, data consistency challenges |

---

# Real-world examples

- **Web applications:** Multiple web servers behind a load balancer.
- **Cloud databases:** Primary with one or more replicas and automatic failover.
- **CDNs:** Content served from many geographically distributed edge locations.
- **Kubernetes:** Multiple control-plane nodes prevent the control plane from becoming a SPOF.

---

# Interview-ready summary

> **A Single Point of Failure (SPOF) is a component whose failure causes the entire system or a critical service to become unavailable because there is no backup or redundancy. Common SPOFs include a single database, application server, load balancer, or data center. To eliminate SPOFs, we introduce redundancy through replication, multiple instances, automatic failover, and multi-region deployments, ensuring the system continues operating even when individual components fail.**

## Question 4. What is the difference between synchronous and asynchronous communication?

# Direct answer

**Synchronous communication** means the caller sends a request and **waits for the response before continuing**. **Asynchronous communication** means the caller sends a request and **continues its work without waiting**; the response or result is handled later.

In distributed systems:

- **Synchronous = Blocking**
- **Asynchronous = Non-blocking**

---

# Intuition

Imagine ordering food at a restaurant.

### Synchronous

You order at the counter and **stand there until your food is ready**.

```
Order → Wait → Receive Food → Leave
```

You can't do anything else while waiting.

### Asynchronous

You order, receive a buzzer, **sit down or do something else**, and collect your food when it's ready.

```
Order → Continue doing other things
             ↓
      Notification
             ↓
       Collect Food
```

---

# Comparison

| Aspect                | Synchronous       | Asynchronous                                             |
| --------------------- | ----------------- | -------------------------------------------------------- |
| Waits for response    | Yes               | No                                                       |
| Caller blocked        | Yes               | No                                                       |
| Coupling              | Tighter           | Looser                                                   |
| Latency               | Higher for caller | Lower perceived latency                                  |
| Scalability           | Lower             | Higher                                                   |
| Complexity            | Simpler           | More complex                                             |
| Failure handling      | Immediate         | Often requires retries, dead-letter queues, or callbacks |
| Typical communication | HTTP, gRPC        | Message queues, event streaming                          |

---

# Synchronous communication

The client waits until the server responds.

```
Client
   │
   │ Request
   ▼
Server
   │
   │ Processing
   ▼
Response
   │
   ▼
Client continues
```

### Example

A user logs in:

1. User submits credentials.
2. Authentication service verifies them.
3. Returns success or failure.
4. Only then can the user access the application.

This is naturally synchronous because the client needs the result immediately.

---

# Asynchronous communication

The sender doesn't wait for the work to complete.

```
Producer
    │
    ▼
Message Queue
    │
    ▼
Consumer
```

The producer can continue immediately after publishing the message.

### Example

Uploading a video:

1. User uploads the file.
2. Service stores it.
3. A message is placed on a queue.
4. Video processing workers transcode it.
5. User is notified when processing finishes.

The upload request returns quickly while processing happens in the background.

---

# Real-world examples

### Synchronous

- User login
- Payment authorization
- Fetching a product page
- Checking account balance
- Searching for products

These operations require an immediate response.

### Asynchronous

- Sending emails
- SMS notifications
- Video transcoding
- Image processing
- Analytics event processing
- Log aggregation
- Background report generation

These can happen later without blocking the user.

---

# Advantages and disadvantages

## Synchronous

### Advantages

- Easier to understand and implement
- Immediate response
- Simpler debugging
- Strong request-response semantics

### Disadvantages

- Caller is blocked
- Higher end-to-end latency
- Lower scalability under heavy load
- One slow service can delay others
- Cascading failures are more likely if downstream services become slow or unavailable

---

## Asynchronous

### Advantages

- Better scalability
- Higher throughput
- Better resource utilization
- Decouples services
- More resilient to temporary failures
- Enables background processing

### Disadvantages

- More complex design
- Harder debugging and tracing
- Results are not immediate
- Requires retries, idempotency, and careful error handling
- Often involves eventual consistency

---

# When to use which?

| Use case              | Recommended approach | Reason                                                          |
| --------------------- | -------------------- | --------------------------------------------------------------- |
| Login                 | Synchronous          | User needs immediate authentication result                      |
| Payment authorization | Synchronous          | Must know if payment succeeded before proceeding                |
| Place order           | Hybrid               | Confirm order synchronously, process fulfillment asynchronously |
| Send email            | Asynchronous         | Doesn't need to block the user                                  |
| Generate reports      | Asynchronous         | Long-running task                                               |
| Video encoding        | Asynchronous         | Resource-intensive background processing                        |
| Notifications         | Asynchronous         | Improves responsiveness                                         |

---

# Hybrid approach (common in modern systems)

Many large-scale systems combine both approaches.

Example: **E-commerce checkout**

```
User
   │
   ▼
Order Service
   │
   ├── Synchronous → Payment Service
   │
   └── Asynchronous → Queue
                        │
                        ├── Email Service
                        ├── Inventory Update
                        ├── Shipping Service
                        └── Analytics
```

The customer receives immediate confirmation that the order was placed, while secondary tasks are processed asynchronously to improve responsiveness and scalability.

---

# Trade-offs

| Synchronous                       | Asynchronous                                  |
| --------------------------------- | --------------------------------------------- |
| Simpler programming model         | Better scalability                            |
| Immediate feedback                | Higher throughput                             |
| Lower system complexity           | Better fault isolation                        |
| Tighter service coupling          | Looser service coupling                       |
| Higher risk of cascading failures | More resilient but operationally more complex |

---

# Interview-ready summary

> **Synchronous communication is a request-response model where the caller waits for the result before continuing, making it suitable for operations that require an immediate answer, such as authentication or payment authorization. Asynchronous communication allows the caller to continue without waiting, making it ideal for background tasks like notifications, media processing, and analytics. In distributed systems, synchronous communication offers simplicity, while asynchronous communication improves scalability, fault tolerance, and throughput at the cost of additional complexity.**

## Question 5. What is the difference between online (OLTP) and analytical (OLAP) databases?

# Direct answer

**OLTP (Online Transaction Processing)** databases are optimized for **fast, frequent transactional operations** such as inserts, updates, and deletes. They power day-to-day application workloads.

**OLAP (Online Analytical Processing)** databases are optimized for **complex analytical queries** over large volumes of historical data. They power reporting, business intelligence, and data analytics.

In simple terms:

- **OLTP = Run the business**
- **OLAP = Analyze the business**

---

# Intuition

Consider an e-commerce company.

### OLTP

When a customer:

- Places an order
- Makes a payment
- Updates their address
- Adds an item to the cart

The system must process each transaction quickly and correctly.

### OLAP

The business wants answers to questions like:

- What were the top-selling products last quarter?
- Which cities generated the most revenue?
- What is the monthly sales trend?

These queries analyze millions of records and are not time-critical.

---

# Comparison

| Aspect        | OLTP                             | OLAP                                       |
| ------------- | -------------------------------- | ------------------------------------------ |
| Purpose       | Process day-to-day transactions  | Analyze historical data                    |
| Workload      | Many small read/write operations | Fewer but complex read-heavy queries       |
| Query type    | Simple CRUD operations           | Aggregations, joins, trend analysis        |
| Data          | Current operational data         | Historical and aggregated data             |
| Response time | Milliseconds                     | Seconds to minutes                         |
| Updates       | Continuous                       | Periodic batch or streaming loads          |
| Schema        | Highly normalized                | Often denormalized (Star/Snowflake schema) |
| Users         | End users, applications          | Analysts, data scientists, BI tools        |
| Primary goal  | Fast transactions                | Fast analytics                             |

---

# OLTP architecture

```text
Users
   │
   ▼
Application
   │
   ▼
OLTP Database
```

Characteristics:

- Thousands of concurrent users
- Frequent inserts and updates
- ACID transactions
- Low latency
- High availability

Example queries:

```sql
INSERT INTO Orders (...);

UPDATE Accounts
SET balance = balance - 100
WHERE account_id = 101;

SELECT * FROM Products
WHERE product_id = 500;
```

These queries typically access a small number of rows.

---

# OLAP architecture

```text
Applications
        │
        ▼
ETL / ELT Pipeline
        │
        ▼
Data Warehouse
        │
        ▼
BI Dashboard / Analysts
```

Characteristics:

- Large scans across millions or billions of rows
- Complex joins and aggregations
- Historical data
- Read-heavy workload

Example query:

```sql
SELECT
    city,
    SUM(total_sales)
FROM sales
WHERE order_date >= '2026-01-01'
GROUP BY city
ORDER BY SUM(total_sales) DESC;
```

This query may process millions of records.

---

# Data modeling

## OLTP

Typically uses **normalized** schemas to reduce redundancy and maintain data integrity.

Example:

```text
Customers
Orders
Products
Payments
```

Each entity is stored separately, with relationships maintained through keys.

### Advantages

- Less duplicate data
- Easier updates
- Strong consistency

---

## OLAP

Often uses **denormalized** schemas, such as **Star** or **Snowflake** schemas, to optimize analytical queries.

Example (Star Schema):

```text
          Date
            │
            │
Product ─ Sales ─ Customer
            │
            │
         Location
```

Fact tables store measurable events (e.g., sales), while dimension tables provide descriptive attributes (e.g., product, customer, date).

### Advantages

- Faster aggregations
- Fewer joins
- Better query performance for analytics

---

# Real-world examples

## OLTP systems

- Banking transactions
- ATM withdrawals
- E-commerce checkout
- Ride booking
- Hotel reservations
- Inventory updates

## OLAP systems

- Sales dashboards
- Financial reporting
- Customer segmentation
- Fraud detection analysis
- Demand forecasting
- Marketing campaign analysis

---

# Can the same database do both?

Technically, yes—but it's usually not a good idea at scale.

Example:

If analysts run a query like:

```sql
SELECT *
FROM Orders
WHERE order_date > '2024-01-01';
```

that scans billions of rows, it can consume significant CPU and I/O resources, slowing down customer-facing operations such as placing new orders.

This is why production systems typically separate operational and analytical workloads.

---

# Common technologies

| OLTP                              | OLAP            |
| --------------------------------- | --------------- |
| PostgreSQL                        | Snowflake       |
| MySQL                             | Google BigQuery |
| Microsoft SQL Server              | Amazon Redshift |
| Oracle Database                   | ClickHouse      |
| MongoDB (transactional workloads) | Apache Druid    |

---

# Trade-offs

| OLTP                           | OLAP                                          |
| ------------------------------ | --------------------------------------------- |
| Excellent for transactions     | Excellent for analytics                       |
| Very low latency               | Optimized for large-scale scans               |
| Strong ACID guarantees         | Optimized for aggregations                    |
| Handles many concurrent writes | Handles complex analytical queries            |
| Poor for heavy reporting       | Poor for high-frequency transactional updates |

---

# Interview-ready summary

> **OLTP databases are designed for fast, concurrent transactional workloads with frequent inserts, updates, and point lookups. They prioritize low latency, ACID guarantees, and data integrity. OLAP databases are designed for analytical workloads over large historical datasets, supporting complex aggregations, reporting, and business intelligence. In large-scale systems, OLTP powers the application's operational data, while OLAP powers dashboards and analytics, typically using separate databases to avoid analytical queries impacting transactional performance.**

## Question 6. What is a quorum in distributed systems?

## Question 7. What is data locality and why is it important?

## Question 8. What is a proxy server? How does it help in scaling?

## Question 9. Difference between reverse proxy and forward proxy

## Question 10. What is edge computing? How does it affect system design?

## Question 11. How do you design an online multiplayer game system?

## Question 12. How do you design an online exam/test-taking platform?

## Question 13. How do you design a real-time bidding system (like Google Ads)?

## Question 14. How do you design a digital wallet system like Paytm/PhonePe?

## Question 15. How do you design a microblogging platform like Twitter?

## Question 16. How do you design a logging system for billions of events per day?

## Question 17. How do you design a recommendation system like Netflix?

## Question 18. How do you design a content moderation system for social media?

## Question 19. How do you design a search engine like Google?

## Question 20. How do you design a collaborative whiteboard tool?
