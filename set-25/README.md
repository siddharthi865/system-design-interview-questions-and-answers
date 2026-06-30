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

# Design an Append-Only Log Store

## Direct answer

An **append-only log store** is a storage system where data is **never updated or deleted in place**. Every new record is appended sequentially to the end of the log, creating an immutable history of events. Reads are performed by offset or key-based indexes, while background processes handle indexing, compaction, and retention.

This design is widely used in distributed systems because sequential appends maximize write throughput, simplify replication, enable replay, and provide strong durability guarantees.

Examples include:

- **Apache Kafka** (event log)
- **Apache Pulsar** (segment-based log)
- Database **Write-Ahead Logs (WAL)**
- Raft/Paxos replicated logs
- Event Sourcing systems

---

# Requirements / Problem Framing

## Functional Requirements

- Append records in order
- Read records by offset
- Sequential scans
- Support multiple log segments
- Persist data durably
- Replicate logs across nodes
- Support retention and compaction
- Recover after crashes

## Non-Functional Requirements

- Very high write throughput
- Low append latency
- Sequential disk I/O
- Durable writes
- Horizontally scalable
- Fault tolerant
- Efficient replay

---

# High-Level Architecture

```text
              Producers
                  |
          Append API Servers
                  |
        ------------------------
        |                      |
   Active Log Segment     Index Manager
        |                      |
        |               Offset Index
        |               Timestamp Index
        |
     SSD / Disk
        |
 Segment Rotation
        |
 Replication Service
        |
 Followers

------------------------------

Background Services

Compaction
Retention
Checksum Validation
Snapshotting
Monitoring
```

---

# Data Model

Each record contains:

```text
Offset
Timestamp
Key (optional)
Value
Headers
Checksum
```

Example:

```text
Offset: 105
Timestamp: 10:05:11
Key: User123
Value: Purchase Event
Checksum: CRC32
```

The **offset** uniquely identifies a record and preserves ordering.

---

# Storage Layout

Logs are divided into immutable **segments**.

```text
Segment-1

Offset 0
Offset 1
Offset 2
...
Offset 99999

------------------------

Segment-2

Offset 100000
Offset 100001
Offset 100002
```

Only the **active segment** accepts writes.

Completed segments become read-only.

Benefits:

- Easier replication
- Efficient cleanup
- Parallel reads
- Faster recovery

---

# Write Path

```text
Producer
    |
Append Request
    |
Leader
    |
Append to Memory Buffer
    |
Sequential Disk Write
    |
Update Offset
    |
Replicate
    |
ACK Producer
```

### Why Sequential Writes?

Sequential disk writes are much faster than random writes.

Advantages:

- Higher throughput
- Lower disk seek overhead
- Better SSD/HDD utilization

---

# Read Path

Reads are typically by offset.

```text
Consumer
     |
Offset = 2,500,000
     |
Locate Segment
     |
Binary Search Index
     |
Sequential Read
```

Indexes map offsets to physical file positions, avoiding full-file scans.

---

# Indexing

Instead of indexing every record, use **sparse indexes**.

Example:

```text
Offset        File Position

0             0
1000          65 KB
2000          128 KB
3000          194 KB
```

Read process:

1. Find the nearest indexed offset.
2. Seek to the file position.
3. Scan forward until the target offset.

This reduces memory usage while keeping lookups fast.

---

# Segment Rotation

When a segment reaches a size threshold (e.g., 1 GB):

```text
Current Segment

↓

Close

↓

Create New Active Segment
```

Older segments become immutable.

Benefits:

- Simplifies replication
- Enables retention policies
- Improves recovery

---

# Replication

A leader appends records and replicates them to followers.

```text
            Leader

          Offset 105

         /          \

Follower A      Follower B
```

Flow:

1. Producer writes to leader.
2. Leader appends locally.
3. Followers replicate the entry.
4. Leader acknowledges once the desired replication quorum is reached.

This ensures durability even if the leader fails.

---

# Crash Recovery

On restart:

1. Read the latest segment.
2. Verify checksums.
3. Truncate any partially written records.
4. Rebuild in-memory indexes.
5. Resume appending from the last committed offset.

Because data is immutable, recovery is straightforward and deterministic.

---

# Log Compaction

Without compaction:

```text
User1 -> Name=A
User1 -> Name=B
User1 -> Name=C
```

All updates remain forever.

With **log compaction**:

```text
User1 -> Name=C
```

Only the latest value for each key is retained while preserving append semantics during normal operation.

Useful for:

- Configuration topics
- Metadata
- State machine replication

---

# Retention

Retention prevents unbounded growth.

Common policies:

- **Time-based:** Keep logs for 7 days.
- **Size-based:** Keep the latest 10 TB.
- **Compaction-based:** Retain the latest record per key.

Expired segments are deleted by background workers.

---

# Deep Design Considerations

## 1. Batching

Instead of writing one record at a time:

```text
100 Records

↓

Single Disk Write
```

Benefits:

- Higher throughput
- Fewer system calls
- Better disk efficiency

---

## 2. Zero-Copy Reads

For large reads, use OS facilities such as `sendfile()` or memory-mapped files.

Benefits:

- Reduced CPU usage
- Fewer memory copies
- Higher throughput

This technique is heavily used in systems like Kafka.

---

## 3. Checksums

Each record includes a checksum.

```text
Record

+

CRC32
```

During reads, verify the checksum to detect corruption caused by disk or network errors.

---

## 4. Backpressure

If producers outpace disk throughput:

```text
Producer

↓

Queue Full

↓

Slow Producer
```

Mechanisms include:

- Rate limiting
- Bounded queues
- Flow control
- Rejecting writes under sustained overload

---

## 5. Partitioning

A single log eventually becomes a bottleneck.

Partition by key:

```text
Orders

↓

Hash(OrderID)

↓

Partition 1

Partition 2

Partition 3
```

Each partition maintains its own ordered append-only log, enabling parallelism.

---

# Capacity / Sizing

Assume:

- **1 million writes/sec**
- Average record size = **1 KB**

Write throughput:

```text
1,000,000 × 1 KB ≈ 1 GB/sec
```

Daily storage:

```text
1 GB/sec × 86,400 sec

≈ 86 TB/day
```

With a replication factor of **3**:

```text
≈ 258 TB/day
```

A 1 GB segment rotates roughly every second at this ingest rate, so larger segment sizes (e.g., 4–10 GB) are often chosen in high-throughput deployments to reduce management overhead.

---

# Security / Observability

## Security

- TLS between clients and servers
- Authentication and authorization
- Encryption at rest
- Immutable audit logs
- Per-tenant access control

## Observability

Monitor:

- Append latency (P50/P95/P99)
- Disk throughput
- Replication lag
- Consumer lag
- Segment count
- Disk utilization
- Checksum failures
- Write error rate
- Recovery time

---

# Trade-offs

| Decision                 | Pros                                          | Cons                                         |
| ------------------------ | --------------------------------------------- | -------------------------------------------- |
| Append-only storage      | Excellent write throughput, immutable history | Requires compaction and retention management |
| Sparse indexes           | Low memory usage                              | Small sequential scan after lookup           |
| Segment-based storage    | Easy rotation and cleanup                     | More metadata to manage                      |
| Leader-based replication | Strong ordering and consistency               | Leader can become a bottleneck               |
| Log compaction           | Reduces storage while preserving latest state | Background processing overhead               |
| Partitioned logs         | Horizontal scalability                        | Ordering guaranteed only within a partition  |

---

# Interview-ready summary

> "An append-only log store is optimized for high-throughput sequential writes by treating data as immutable and appending every new record to the end of the log. The log is divided into immutable segments, indexed by sparse offset indexes, and replicated for durability. Reads use offsets to locate records efficiently, while background services handle compaction, retention, and recovery. This architecture underpins systems like Kafka and database write-ahead logs because it offers excellent write performance, simple crash recovery, efficient replication, and the ability to replay history."

## Question 5. How do you implement deduplication at scale (like Gmail threads)?

# Direct answer

Deduplication at scale involves identifying records that represent the **same logical entity** and storing or displaying only one canonical copy while preserving references to duplicates. At systems like Gmail, this is achieved using a combination of **deterministic identifiers**, **content hashing**, **indexes**, **similarity algorithms**, and **background reconciliation**.

For Gmail-like email threading, it's important to distinguish two related concepts:

- **Message deduplication**: Prevent storing the same email multiple times.
- **Conversation threading**: Group different emails belonging to the same conversation.

These are implemented using different techniques.

---

# Requirements / Problem Framing

## Functional Requirements

- Detect duplicate records during writes
- Support billions of records
- Low-latency inserts and lookups
- Avoid duplicate storage
- Preserve references to original records
- Handle concurrent writes
- Recover from missed duplicates

## Non-Functional Requirements

- Horizontal scalability
- High availability
- Low false positives
- Idempotent operations
- Efficient storage utilization

---

# High-Level Architecture

```text
              Clients
                  |
             API Gateway
                  |
          Deduplication Service
                  |
      ------------------------------
      |            |               |
 Content Hash   Metadata Index   Similarity Engine
      |            |               |
      ------------------------------
                  |
          Canonical Record Store
                  |
        Object / Blob Storage

---------------------------------------

Background Pipeline

New Records
      |
 Message Queue
      |
 Similarity Matching
      |
 Merge Candidates
      |
 Index Updates
```

---

# Approach 1: Exact Deduplication (Hash-Based)

For exact duplicates, compute a cryptographic hash of the content.

```text
Email Body

↓

SHA-256

↓

8F4A92...
```

Workflow:

1. Compute content hash.
2. Check if the hash already exists.
3. If yes, reuse the existing object.
4. Otherwise, store the new object and index the hash.

### Advantages

- O(1) lookup
- Very accurate
- Easy to scale

### Limitations

Any modification (even a single character) produces a different hash.

---

# Approach 2: Metadata-Based Deduplication

Sometimes records differ slightly but represent the same logical object.

For emails, compare metadata such as:

- Message-ID
- Sender
- Recipient
- Subject
- Timestamp
- Attachment IDs

Example:

```text
Message-ID:
abc123@example.com
```

If the same `Message-ID` is received multiple times, treat it as the same message.

This is much cheaper than comparing full content.

---

# Approach 3: Similarity-Based Deduplication

For near-duplicates, use similarity algorithms instead of exact hashes.

Examples:

- SimHash
- MinHash
- Locality Sensitive Hashing (LSH)

Example:

```text
Document A

Hello World!

Document B

Hello World!!
```

SHA-256:

```text
Different
```

SimHash:

```text
Very Similar
```

Useful for:

- Emails
- Documents
- Images
- News articles

---

# Gmail Threading

Gmail threading is **not simply deduplication**.

Instead, it groups related messages into a conversation using metadata such as:

- `Message-ID`
- `In-Reply-To`
- `References`
- Subject normalization (e.g., removing prefixes like "Re:" and "Fwd:")

Example:

```text
Message A

↓

Reply B

↓

Reply C
```

All three are distinct emails but belong to one conversation.

```text
Conversation
    |
-------------
|     |      |
A     B      C
```

The conversation maintains an ordered list of message IDs rather than merging messages into one.

---

# Data Model

## Message Table

```text
MessageID
ConversationID
Hash
Sender
Timestamp
StoragePointer
```

## Conversation Table

```text
ConversationID

↓

List<MessageIDs>
```

This separation allows efficient retrieval of entire threads.

---

# Indexes

Maintain indexes for:

```text
Content Hash

↓

Message ID
```

```text
Message-ID

↓

Storage Pointer
```

```text
Conversation ID

↓

Message List
```

These indexes enable constant-time lookups.

---

# Handling Concurrent Inserts

Two identical messages may arrive simultaneously.

Without coordination:

```text
Server A

Store

Server B

Store
```

Duplicate storage occurs.

Solution:

Use atomic operations.

Example:

```text
SETNX(hash)
```

or

```text
INSERT ... ON CONFLICT DO NOTHING
```

Only one request creates the canonical record; others reuse it.

---

# Background Deduplication

Some duplicates are difficult to detect in the request path.

A background job can:

1. Scan recent records.
2. Group by similarity.
3. Merge duplicates.
4. Update indexes.
5. Remove redundant storage.

This reduces write latency while improving long-term storage efficiency.

---

# Deep Design Considerations

## 1. Content-Addressable Storage

Instead of naming objects by filename:

```text
Object Name

↓

SHA-256(Content)
```

If identical content already exists, simply create another reference.

This is common in backup systems and version control.

---

## 2. Reference Counting

When multiple logical records point to the same stored object:

```text
Blob X

↓

Reference Count = 4
```

Deleting one message decrements the count.

Only delete the blob when the count reaches zero.

---

## 3. Bloom Filters

At very high scale, checking the primary index for every insert can be expensive.

Use a Bloom filter as a fast pre-check:

```text
Incoming Hash

↓

Bloom Filter

↓

Probably Exists

↓

Verify in Index
```

Advantages:

- Reduces database lookups
- Small memory footprint

Limitation:

- False positives are possible.
- False negatives are not.

---

## 4. Partitioning

Shard by hash:

```text
Hash

↓

Shard 1

Shard 2

Shard 3
```

Benefits:

- Even load distribution
- Parallel processing
- Horizontal scaling

---

## 5. Eventual Consistency

Cross-region deployments may temporarily store duplicates.

Periodic reconciliation jobs compare indexes and merge duplicates later.

This favors availability and low write latency over immediate global deduplication.

---

# Capacity / Sizing

Assume:

- **500 million new emails/day**
- Average size: **100 KB**
- 15% exact duplicates

Without deduplication:

```text
500M × 100 KB

≈ 50 TB/day
```

With 15% duplicate elimination:

```text
≈ 42.5 TB/day
```

Saving roughly **7.5 TB/day**, in addition to reduced replication and backup costs.

---

# Trade-offs

| Approach                    | Pros                              | Cons                             |
| --------------------------- | --------------------------------- | -------------------------------- |
| Content hashing             | Fast, accurate for identical data | Misses near-duplicates           |
| Metadata matching           | Very inexpensive                  | Depends on reliable metadata     |
| Similarity algorithms       | Detects modified copies           | Higher CPU cost                  |
| Background reconciliation   | Doesn't slow writes               | Duplicates may exist temporarily |
| Content-addressable storage | Excellent storage efficiency      | Requires reference management    |

---

# Interview-ready summary

> "At scale, deduplication is usually implemented in layers. Exact duplicates are detected using content hashes and content-addressable storage, while metadata indexes provide fast identity checks. Near-duplicates require similarity algorithms such as SimHash or MinHash, often executed asynchronously. For Gmail-like systems, message deduplication and conversation threading are separate concerns: duplicate messages are eliminated using identifiers like `Message-ID` and content hashes, while threads are built using `Message-ID`, `In-Reply-To`, and `References` headers, allowing distinct messages to be grouped into a single conversation without losing their individual identities.

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
