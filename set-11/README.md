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

## Question 3. What is data warehousing and how does it differ from OLTP?

## Question 4. What is a distributed transaction?

## Question 5. Explain the concept of ACID in databases

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
