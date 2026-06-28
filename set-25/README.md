# Set 25

| S.No. | Question                                                                                                                                                                     |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What's the architecture of a low-latency ad bidding system?](#question-1-whats-the-architecture-of-a-low-latency-ad-bidding-system)                                         |
| 2.    | [How do you design a real-time fraud detection system?](#question-2-how-do-you-design-a-real-time-fraud-detection-system)                                                    |
| 3.    | [What is Lambda Architecture in data processing?](#question-3-what-is-lambda-architecture-in-data-processing)                                                                |
| 4.    | [How would you design an append-only log store?](#question-4-how-would-you-design-an-append-only-log-store)                                                                  |
| 5.    | [How do you implement deduplication at scale (like Gmail threads)?](#question-5-how-do-you-implement-deduplication-at-scale-like-gmail-threads)                              |
| 6.    | [How would you design a serverless workflow orchestration engine?](#question-6-how-would-you-design-a-serverless-workflow-orchestration-engine)                              |
| 7.    | [How do you handle multi-cloud deployments in system design?](#question-7-how-do-you-handle-multi-cloud-deployments-in-system-design)                                        |
| 8.    | [What are the trade-offs of eventual vs. strong consistency in e-commerce carts?](#question-8-what-are-the-trade-offs-of-eventual-vs-strong-consistency-in-e-commerce-carts) |
| 9.    | [How would you design a geo-distributed blockchain ledger?](#question-9-how-would-you-design-a-geo-distributed-blockchain-ledger)                                            |
| 10.   | [How do you design a distributed file lock service?](#question-10-how-do-you-design-a-distributed-file-lock-service)                                                         |
| 11.   | [What is sharded counters design, and how would you implement it?](#question-11-what-is-sharded-counters-design-and-how-would-you-implement-it)                              |
| 12.   | [How would you design a multi-tenant logging platform?](#question-12-how-would-you-design-a-multi-tenant-logging-platform)                                                   |
| 13.   | [How do you design a zero-downtime schema migration system?](#question-13-how-do-you-design-a-zero-downtime-schema-migration-system)                                         |
| 14.   | [How would you design the architecture of a CDN edge computing platform?](#question-14-how-would-you-design-the-architecture-of-a-cdn-edge-computing-platform)               |
| 15.   | [How do you design a scalable recommendation system (like Netflix)?](#question-15-how-do-you-design-a-scalable-recommendation-system-like-netflix)                           |
| 16.   | [How would you design a system for real-time multiplayer gaming?](#question-16-how-would-you-design-a-system-for-real-time-multiplayer-gaming)                               |
| 17.   | [How do you design a resilient distributed job queue?](#question-17-how-do-you-design-a-resilient-distributed-job-queue)                                                     |
| 18.   | [What are anti-entropy protocols, and where are they used?](#question-18-what-are-anti-entropy-protocols-and-where-are-they-used)                                            |
| 19.   | [How do you design a privacy-preserving data analytics platform?](#question-19-how-do-you-design-a-privacy-preserving-data-analytics-platform)                               |
| 20.   | [How would you design an operating system scheduler for multi-core CPUs?](#question-20-how-would-you-design-an-operating-system-scheduler-for-multi-core-cpus)               |

## Question 1. What's the architecture of a low-latency ad bidding system?

# Design a Low-Latency Ad Bidding System (Real-Time Bidding)

## Direct answer

A low-latency ad bidding system is typically built as an **event-driven, distributed system** that processes bid requests from ad exchanges, evaluates targeting rules and machine learning models, retrieves user/context data, computes bid prices, and returns a bid response within **50–100 ms**, often targeting **<20 ms internal processing time**.

The architecture prioritizes:

- Extremely low latency
- High availability
- Massive throughput (millions of QPS)
- In-memory data access
- Horizontal scalability
- Graceful degradation under overload

---

# Requirements / Problem Framing

### Functional Requirements

- Receive bid requests from ad exchanges
- Retrieve user profile and campaign information
- Match eligible campaigns
- Predict CTR/CVR
- Compute bid price
- Return winning bid
- Track impressions, clicks, and conversions
- Budget pacing
- Fraud detection

### Non-Functional Requirements

- End-to-end latency <100 ms
- Internal processing target <20 ms
- Millions of requests/sec
- 99.99% availability
- Horizontal scalability
- Fault tolerance
- Real-time analytics

---

# High-Level Architecture

```text
                  Ad Exchange
                       |
                Bid Request API
                       |
                Global Load Balancer
                       |
              ----------------------
              |                    |
         Bid Server           Bid Server
              |                    |
     -------------------------------
     |        |        |           |
 User Cache Campaign Cache ML Model
     |        |        |
     -------------------
             |
     Bid Decision Engine
             |
      Budget Service
             |
      Bid Response (<100ms)

--------------------------------------------

Async Pipeline

Impression
Click
Conversion
Logs
      |
 Kafka / Pulsar
      |
 Stream Processing
      |
 Analytics
Billing
Fraud Detection
Model Training
```

---

# Core Components

## 1. Bid Request API

Responsibilities:

- Receive OpenRTB requests
- Authentication
- Validation
- Rate limiting
- Deserialize payload
- Forward to bidding engine

Should be stateless.

---

## 2. User Profile Store

Contains

- demographics
- interests
- device
- browsing history
- previous clicks

Requirements

- sub-millisecond lookup

Typically:

```
Redis
Aerospike
Memcached
```

Most hot data lives entirely in memory.

---

## 3. Campaign Store

Stores

- targeting rules
- creatives
- advertiser budgets
- bid strategy
- keywords
- frequency caps

Usually replicated across bidding servers.

Updates happen asynchronously.

---

## 4. Targeting Engine

Determines eligible campaigns.

Filters by:

- country
- language
- device
- browser
- age
- interests
- time
- placement
- publisher
- category

Instead of scanning campaigns:

Use inverted indexes.

Example

```
Country -> Campaign IDs

US ->
12
15
82

Mobile ->
15
18
82

Sports ->
82
91
```

Intersect indexes.

```
US

∩

Mobile

∩

Sports
```

Returns a very small candidate set.

---

## 5. ML Prediction Service

Scores each candidate.

Models predict

- CTR
- CVR
- revenue
- quality score

Usually models are

- preloaded in memory
- optimized
- quantized
- vectorized

Avoid remote ML inference.

Local inference is much faster.

---

## 6. Bid Pricing Engine

Computes

```
Expected Value

=

CTR

×

CVR

×

Conversion Value
```

Then applies

- advertiser strategy
- pacing
- reserve price
- bid caps
- quality adjustments

Returns bid amount.

---

## 7. Budget Service

Tracks remaining advertiser budget.

Needs atomic updates.

Common approaches

### Option 1

Distributed counters

```
Redis
```

Fast

---

### Option 2

Local budget cache

Synchronize every few seconds.

Lower latency.

Eventually consistent.

Preferred.

---

## 8. Response Generator

Returns

- bid amount
- creative ID
- metadata
- advertiser ID

Must respond before timeout.

If processing exceeds deadline:

Return no bid.

---

# Data Flow

```text
Receive Bid Request
        |
Validate Request
        |
Lookup User
        |
Retrieve Candidate Campaigns
        |
Apply Targeting
        |
Predict CTR
        |
Calculate Bid
        |
Check Budget
        |
Return Bid
```

Entire flow:

10–20 ms.

---

# Deep Design Considerations

## 1. In-Memory Everything

Disk lookup:

```
5-10 ms
```

Memory lookup:

```
<100 μs
```

Therefore:

- campaign metadata
- user features
- models
- indexes

stay in RAM.

---

## 2. Horizontal Scaling

Bid servers are stateless.

```text
LB

↓

Server 1

Server 2

Server 3

Server N
```

Add more servers to increase throughput.

---

## 3. Caching

Cache

- campaign metadata
- advertiser settings
- targeting indexes
- user features
- ML features

Avoid database calls.

---

## 4. Data Locality

Keep

```
Bid Server

+

User Cache

+

Campaign Cache

+

ML Model
```

on the same machine.

Avoid network hops.

---

## 5. Timeouts

Every component has strict deadlines.

Example

```
User lookup
2 ms

Targeting
3 ms

ML
4 ms

Budget
2 ms

Response
2 ms
```

If one stage exceeds budget:

Skip it or return no bid.

---

## 6. Graceful Degradation

If ML unavailable

↓

Use heuristic bid.

If user profile unavailable

↓

Contextual targeting only.

If budget service unavailable

↓

Reject bids.

---

## 7. Asynchronous Event Pipeline

Never block bid serving for logging.

Instead

```
Bid Won
     |
 Kafka
     |
Consumers

Analytics

Billing

Fraud Detection

Reporting

Model Training
```

Critical path remains fast.

---

# Capacity / Sizing

Assume:

- 5 million bid requests/sec
- Average request = 2 KB
- Response = 1 KB

Incoming traffic:

```
5M × 2KB

≈10 GB/sec
```

Outgoing:

```
5M × 1KB

≈5 GB/sec
```

If one server handles

```
20,000 requests/sec
```

Need approximately

```
5,000,000 / 20,000

≈250 bidding servers
```

(ignoring redundancy).

Campaign metadata:

```
10 million campaigns

Average 1 KB

≈10 GB
```

Can be partitioned or replicated depending on memory capacity.

---

# Security / Observability

### Security

- TLS for all communication
- Request authentication with exchanges
- Creative validation
- Fraud detection
- Rate limiting
- DDoS protection
- Audit logs

### Observability

Monitor:

- Bid latency (P50/P95/P99)
- Timeouts
- No-bid rate
- Win rate
- Budget utilization
- CTR/CVR
- Cache hit ratio
- QPS
- Error rate
- ML inference latency

Distributed tracing helps identify latency hotspots across the bidding pipeline.

---

# Trade-offs

| Decision              | Pros                      | Cons                           |
| --------------------- | ------------------------- | ------------------------------ |
| In-memory cache       | Extremely fast            | Higher RAM cost                |
| Local ML inference    | Very low latency          | Model deployment complexity    |
| Local budget cache    | Fast, scalable            | Eventual consistency           |
| Async logging         | Keeps critical path short | Delayed analytics              |
| Stateless bid servers | Easy horizontal scaling   | Requires external/shared state |
| Heuristic fallback    | Maintains availability    | Lower bidding accuracy         |

---

# Interview-ready summary

> "A real-time ad bidding system is designed around a stateless bidding engine backed by in-memory user and campaign caches, local ML inference, and a fast bid decision engine. The synchronous path is optimized to complete within 20 ms by avoiding disk I/O and minimizing network hops. Budget management uses fast distributed counters or eventually consistent local caches, while impressions, clicks, billing, fraud detection, and model training are handled asynchronously through a streaming platform such as Kafka. The architecture scales horizontally and emphasizes low tail latency, graceful degradation, and high availability."

## Question 2. How do you design a real-time fraud detection system?

# Design a Real-Time Fraud Detection System

## Direct answer

A real-time fraud detection system analyzes transactions as they occur and decides within **10–100 ms** whether to **approve, reject, or flag** them for further review. It combines **rule-based detection**, **machine learning models**, **historical user behavior**, and **stream processing** to identify suspicious activity while minimizing false positives.

The system must balance:

- Low latency
- High detection accuracy
- High availability
- Scalability
- Explainability for fraud decisions

---

# Requirements / Problem Framing

## Functional Requirements

- Evaluate every incoming transaction in real time
- Detect fraudulent patterns
- Assign a fraud risk score
- Approve, reject, or challenge (e.g., OTP/2FA)
- Track user/device/IP history
- Support dynamic fraud rules
- Generate alerts
- Store fraud decisions for audits

## Non-Functional Requirements

- Decision latency: **<50–100 ms**
- Millions of transactions/day
- High availability (99.99%+)
- Horizontally scalable
- Highly accurate with low false positives
- Fault tolerant

---

# High-Level Architecture

```text
                  Payment Gateway
                         |
                  Transaction API
                         |
                  Load Balancer
                         |
                 Fraud Detection Service
                         |
      ------------------------------------------------
      |              |             |                 |
 Rule Engine   Feature Store   ML Inference   Risk Scoring
      |              |             |                 |
      -------------------------------
                     |
             Decision Engine
         (Approve / Reject / Review)
                     |
             Payment Processor

---------------------------------------------------------

        Async Event Pipeline

Transactions
      |
 Kafka / Pulsar
      |
 Stream Processing
      |
 Fraud Analytics
 Rule Updates
 Model Training
 Dashboard
```

---

# Core Components

## 1. Transaction Ingestion

Receives:

- card payments
- bank transfers
- wallet payments
- login events
- account changes

Each event contains:

- User ID
- Merchant
- Amount
- Currency
- Device ID
- IP Address
- Location
- Timestamp

The ingestion layer is stateless and horizontally scalable.

---

## 2. Feature Store

Fraud decisions depend heavily on historical context.

Examples:

- Number of transactions in last minute
- Total spending today
- Failed login attempts
- Previous fraud history
- Known devices
- Trusted locations
- Velocity metrics
- Chargeback history

These features are stored in fast, in-memory systems like Redis or Aerospike for sub-millisecond lookups.

---

## 3. Rule Engine

Fast deterministic checks catch obvious fraud.

Examples:

```text
IF amount > $10,000
AND new device
THEN high risk
```

```text
IF >10 failed logins in 5 minutes
THEN block account
```

```text
IF card used in two countries within 10 minutes
THEN reject
```

Advantages:

- Extremely fast
- Easy to explain
- Easy to update

Limitations:

- Cannot detect unknown fraud patterns.

---

## 4. ML Inference Service

Machine learning detects subtle patterns missed by static rules.

Typical features include:

- Transaction amount deviation
- Merchant category
- Device fingerprint
- Time of day
- User spending habits
- Geo-location changes
- IP reputation
- Historical fraud rate

Common models:

- Gradient Boosted Trees (XGBoost, LightGBM)
- Random Forest
- Deep Neural Networks
- Graph Neural Networks (for fraud rings)

Models are loaded into memory to avoid network latency.

---

## 5. Risk Scoring Engine

Combines rule-based and ML outputs into a single score.

Example:

```text
Risk Score =
0.6 × ML Score
+ 0.3 × Rule Score
+ 0.1 × Historical Reputation
```

Decision thresholds:

| Score  | Action                |
| ------ | --------------------- |
| 0–30   | Approve               |
| 31–70  | Challenge (OTP / MFA) |
| 71–100 | Reject                |

Thresholds can vary by merchant, geography, or transaction type.

---

## 6. Decision Engine

Returns one of:

- **Approve** – transaction proceeds.
- **Reject** – transaction blocked.
- **Review** – manual review or step-up authentication.

Every decision is logged with the contributing rules and model outputs for auditing.

---

# Data Flow

```text
Receive Transaction
        |
Validate Request
        |
Retrieve Historical Features
        |
Execute Rule Engine
        |
Run ML Model
        |
Compute Risk Score
        |
Decision Engine
        |
Approve / Reject / Review
```

Target processing time:

**20–50 ms**

---

# Deep Design Considerations

## 1. Low-Latency Feature Access

Avoid database reads during the critical path.

Keep frequently accessed features in memory:

- Recent transactions
- Device history
- Velocity counters
- User profiles

---

## 2. Streaming Feature Updates

Recent behavior is often more predictive than long-term history.

Example:

```text
Transaction A

↓

Kafka

↓

Feature Updater

↓

Redis

↓

Next transaction uses updated features
```

This enables near real-time behavioral analysis.

---

## 3. Velocity Detection

Track rapid activity using sliding windows.

Examples:

- Transactions per minute
- Spending in last hour
- Login attempts
- Different cards from same IP
- Different accounts on same device

Use windowed aggregations in stream processors.

---

## 4. Device Fingerprinting

Identify devices using:

- Browser attributes
- OS version
- Screen resolution
- Installed fonts
- Hardware identifiers (where permitted)
- Mobile device identifiers

Useful for detecting account takeovers and device spoofing.

---

## 5. Graph-Based Fraud Detection

Represent relationships as a graph:

```text
User
  |
Device
  |
Card
  |
Merchant
```

Clusters with excessive shared devices, IPs, or payment methods often indicate organized fraud.

Graph databases or graph analytics engines can uncover fraud rings offline or near real time.

---

## 6. Adaptive Rules

Fraud patterns evolve quickly.

Rules should be:

- Configurable
- Versioned
- Deployable without service restarts
- Gradually rolled out (canary deployments)

---

## 7. Asynchronous Analytics

Do not block transaction processing for:

- Reporting
- Dashboards
- Model retraining
- Chargeback analysis
- Fraud investigations

Instead, publish events to Kafka/Pulsar for downstream consumers.

---

# Capacity / Sizing

Assume:

- **100,000 transactions/sec**
- Request size: **2 KB**
- Decision latency: **<50 ms**

Incoming traffic:

```text
100,000 × 2 KB ≈ 200 MB/sec
```

If one fraud server handles:

```text
5,000 TPS
```

Servers required:

```text
100,000 / 5,000 = 20 servers
```

(plus redundancy for failover and peak traffic).

---

# Security / Observability

## Security

- TLS for all communication
- Authentication between services
- Encryption of sensitive data at rest
- PCI DSS compliance for payment systems
- Tokenization of card numbers
- Immutable audit logs
- Fine-grained access control

## Observability

Monitor:

- Decision latency (P50/P95/P99)
- Fraud detection rate
- False positive rate
- False negative rate
- Rule hit frequency
- ML inference latency
- Feature store latency
- Transactions per second
- Error rates
- Model drift
- Cache hit ratio

Distributed tracing helps isolate latency bottlenecks across the fraud pipeline.

---

# Trade-offs

| Decision                | Pros                       | Cons                                          |
| ----------------------- | -------------------------- | --------------------------------------------- |
| Rule engine             | Fast, explainable          | Limited to known patterns                     |
| ML models               | Detect unknown fraud       | Harder to explain, requires retraining        |
| In-memory feature store | Very low latency           | Higher infrastructure cost                    |
| Local model inference   | Eliminates network latency | More complex model deployment                 |
| Streaming updates       | Fresh behavioral features  | Operational complexity                        |
| Graph analytics         | Detects fraud rings        | Resource-intensive, often not fully real time |

---

# Interview-ready summary

> "A real-time fraud detection system processes every transaction through a low-latency pipeline that combines in-memory feature retrieval, deterministic rule evaluation, and local ML inference to produce a risk score within tens of milliseconds. The system keeps the synchronous path lightweight while using streaming platforms for analytics, feature updates, and model retraining. It scales horizontally, supports evolving fraud rules, and balances fraud detection accuracy with a low false positive rate through configurable decision thresholds and continuous monitoring."

## Question 3. What is Lambda Architecture in data processing?

# Direct answer

**Lambda Architecture** is a data processing architecture that combines **batch processing** and **stream processing** to provide both **accurate historical results** and **low-latency real-time results**.

The core idea is:

- The **Batch Layer** computes accurate results over all historical data.
- The **Speed Layer** processes recent events in real time before the batch layer catches up.
- The **Serving Layer** merges results from both layers to answer queries.

It was introduced to overcome the trade-off between the high accuracy of batch systems and the low latency of stream processing systems.

---

# Why was Lambda Architecture introduced?

Traditional batch systems (e.g., Hadoop) are highly accurate but slow.

```text
Event arrives
      |
Batch job runs every 6 hours
      |
Result available after hours
```

Pure streaming systems provide immediate results but can struggle with late-arriving data, failures, or complex recomputations.

Lambda combines the strengths of both approaches.

---

# High-Level Architecture

```text
                  Data Sources
                       |
                Event Ingestion
                 (Kafka/Pulsar)
                       |
        ---------------------------------
        |                               |
   Batch Layer                    Speed Layer
 (Historical Processing)     (Real-time Processing)
        |                               |
 Batch Views                    Real-time Views
        |                               |
        -----------Serving Layer---------
                    |
               Query/API Layer
```

---

# Components

## 1. Batch Layer

The batch layer stores the **immutable master dataset** and periodically recomputes results from scratch.

Responsibilities:

- Store all raw events
- Perform large-scale computations
- Generate accurate batch views
- Correct errors from previous computations

Typical technologies:

- Hadoop
- Spark
- Distributed file systems (e.g., HDFS or cloud object storage)

Example:

```text
All purchase events

↓

Nightly Spark job

↓

Total revenue per merchant
```

### Advantages

- Highly accurate
- Fault tolerant
- Easy to recompute from raw data

### Disadvantage

High latency.

---

## 2. Speed Layer

The speed layer processes new events immediately.

Responsibilities:

- Process incoming events
- Compute incremental updates
- Produce low-latency views

Typical technologies:

- Apache Flink
- Apache Storm
- Spark Streaming
- Kafka Streams

Example:

```text
Purchase Event

↓

Update merchant revenue immediately
```

Latency:

Milliseconds to seconds.

---

## 3. Serving Layer

The serving layer exposes queryable data.

When a query arrives:

```text
Historical Batch Result

+

Real-time Increment

↓

Final Answer
```

Example:

Suppose:

Batch layer says

```text
Revenue = $2,000,000
```

Speed layer says

```text
Revenue since last batch = $8,000
```

User query returns

```text
$2,008,000
```

---

# Data Flow

```text
Event arrives
      |
      +----------------------+
      |                      |
      |                  Speed Layer
      |                      |
      |               Increment View
      |
Batch Storage
      |
Batch Processing
      |
Batch View
      |
Serving Layer
      |
Client Query
```

---

# Example: Real-Time Analytics Dashboard

Suppose you're building an analytics dashboard.

Without Lambda:

```text
Sales update every 24 hours
```

Dashboard is stale.

With Lambda:

```text
Yesterday's totals

+

Today's live sales

↓

Current dashboard
```

Users always see near real-time metrics while maintaining historical accuracy.

---

# Advantages

### 1. Low Latency

Recent events become visible almost immediately.

---

### 2. Accurate Historical Results

Batch jobs periodically recompute results from the complete dataset, correcting any errors or missed events.

---

### 3. Fault Tolerance

If the streaming layer misses events due to failures, the next batch recomputation restores correctness.

---

### 4. Scalability

Both batch and stream processing can scale horizontally by adding more workers.

---

# Challenges

The biggest drawback is **operational complexity**.

You maintain two processing pipelines:

- Batch pipeline
- Streaming pipeline

Both must implement the same business logic.

Example:

```text
Fraud Detection Rule

↓

Implemented in Spark

AND

Implemented again in Flink
```

This duplication increases development effort and the risk of inconsistencies.

---

# Lambda vs. Kappa Architecture

| Feature                  | Lambda                                 | Kappa                                  |
| ------------------------ | -------------------------------------- | -------------------------------------- |
| Processing pipelines     | Two (batch + stream)                   | One (stream only)                      |
| Historical recomputation | Batch recomputation                    | Replay event log                       |
| Complexity               | Higher                                 | Lower                                  |
| Latency                  | Low                                    | Very low                               |
| Code duplication         | Yes                                    | No                                     |
| Best for                 | Mixed historical + real-time workloads | Event-driven systems with durable logs |

---

# Common Use Cases

Lambda Architecture is well suited for:

- Real-time fraud detection
- Recommendation systems
- Clickstream analytics
- IoT sensor processing
- Financial trading analytics
- Network monitoring
- Operational dashboards
- Log analytics

---

# Trade-offs

| Pros                              | Cons                                 |
| --------------------------------- | ------------------------------------ |
| Low-latency results               | Two processing pipelines to maintain |
| Accurate historical recomputation | Duplicate business logic             |
| Fault tolerant                    | Higher infrastructure cost           |
| Scales well                       | More operational complexity          |
| Handles late-arriving data        | Harder to test and debug             |

---

# Interview-ready summary

> "Lambda Architecture combines a **batch layer** for accurate historical computation with a **speed layer** for low-latency processing, and a **serving layer** that merges results from both. This provides real-time responsiveness while preserving correctness through periodic batch recomputation. Its main trade-off is operational complexity because the same business logic must often be implemented and maintained in both the batch and streaming pipelines. Today, many organizations prefer Kappa Architecture when a durable event log and mature stream processing framework can satisfy both real-time and replay requirements."

## Question 4. How would you design an append-only log store?

## Question 5. How do you implement deduplication at scale (like Gmail threads)?

## Question 6. How would you design a serverless workflow orchestration engine?

## Question 7. How do you handle multi-cloud deployments in system design?

## Question 8. What are the trade-offs of eventual vs. strong consistency in e-commerce carts?

## Question 9. How would you design a geo-distributed blockchain ledger?

## Question 10. How do you design a distributed file lock service?

## Question 11. What is sharded counters design, and how would you implement it?

## Question 12. How would you design a multi-tenant logging platform?

## Question 13. How do you design a zero-downtime schema migration system?

## Question 14. How would you design the architecture of a CDN edge computing platform?

## Question 15. How do you design a scalable recommendation system (like Netflix)?

## Question 16. How would you design a system for real-time multiplayer gaming?

## Question 17. How do you design a resilient distributed job queue?

## Question 18. What are anti-entropy protocols, and where are they used?

## Question 19. How do you design a privacy-preserving data analytics platform?

## Question 20. How would you design an operating system scheduler for multi-core CPUs?
