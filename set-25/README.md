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

## Question 3. What is Lambda Architecture in data processing?

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
