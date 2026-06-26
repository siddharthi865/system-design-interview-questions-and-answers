# Set 12

| S.No. | Question                                                                                                                     |
| ----- | ---------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you design a notification retry mechanism?](#question-1-how-do-you-design-a-notification-retry-mechanism)            |
| 2.    | [How do you design a spell checker system?](#question-2-how-do-you-design-a-spell-checker-system)                            |
| 3.    | [How do you design a plagiarism checker?](#question-3-how-do-you-design-a-plagiarism-checker)                                |
| 4.    | [How do you design a news feed ranking system?](#question-4-how-do-you-design-a-news-feed-ranking-system)                    |
| 5.    | [How do you design a comment threading system?](#question-5-how-do-you-design-a-comment-threading-system)                    |
| 6.    | [How do you design a shopping cart system?](#question-6-how-do-you-design-a-shopping-cart-system)                            |
| 7.    | [How do you design a dynamic pricing engine?](#question-7-how-do-you-design-a-dynamic-pricing-engine)                        |
| 8.    | [How do you design a taxi fare calculator system?](#question-8-how-do-you-design-a-taxi-fare-calculator-system)              |
| 9.    | [How do you design a leaderboard system for games?](#question-9-how-do-you-design-a-leaderboard-system-for-games)            |
| 10.   | [How do you design a coupon/voucher redemption system?](#question-10-how-do-you-design-a-couponvoucher-redemption-system)    |
| 11.   | [What is write amplification in storage systems?](#question-11-what-is-write-amplification-in-storage-systems)               |
| 12.   | [What are secondary indexes in NoSQL?](#question-12-what-are-secondary-indexes-in-nosql)                                     |
| 13.   | [What is a time-series database (TSDB)?](#question-13-what-is-a-time-series-database-tsdb)                                   |
| 14.   | [How do you design a database for IoT data?](#question-14-how-do-you-design-a-database-for-iot-data)                         |
| 15.   | [What is cold storage vs hot storage?](#question-15-what-is-cold-storage-vs-hot-storage)                                     |
| 16.   | [What is a columnar database? Give examples](#question-16-what-is-a-columnar-database-give-examples)                         |
| 17.   | [What is a key-value store?](#question-17-what-is-a-key-value-store)                                                         |
| 18.   | [How do you design a database for full-text search?](#question-18-how-do-you-design-a-database-for-full-text-search)         |
| 19.   | [What is an LSM tree (Log-Structured Merge Tree)?](#question-19-what-is-an-lsm-tree-log-structured-merge-tree)               |
| 20.   | [How do you prevent data skew in distributed databases?](#question-20-how-do-you-prevent-data-skew-in-distributed-databases) |

## Question 1. How do you design a notification retry mechanism?

# How do you design a notification retry mechanism?

## Direct answer

A **notification retry mechanism** ensures that notifications (email, SMS, push, webhook, etc.) are eventually delivered despite temporary failures. The design should:

- Retry only **retryable failures** (timeouts, 5xx, rate limits)
- Use **exponential backoff with jitter**
- Store retry state persistently
- Limit retries with a maximum retry count or time window
- Support **dead-letter queues (DLQ)** for permanently failed notifications
- Ensure **idempotency** to avoid duplicate deliveries
- Continuously monitor retry success and failure metrics

The goal is to maximize delivery while avoiding overwhelming downstream services.

---

# Requirements / Problem Framing

### Functional Requirements

- Send notifications asynchronously
- Retry failed deliveries
- Support multiple channels (Email, SMS, Push)
- Prevent duplicate notifications
- Handle temporary and permanent failures
- Allow manual replay of failed notifications

### Non-functional Requirements

- High reliability
- Scalable to millions of notifications
- Fault tolerant
- Observable
- Configurable retry policies

---

# High-Level Architecture

```text
              User Action
                    │
                    ▼
          Notification Service
                    │
                    ▼
             Message Queue
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
    Notification Worker   Retry Worker
          │                   │
          ▼                   │
   Email/SMS/Push Provider    │
          │                   │
      Success?                │
      │      │                │
     Yes     No───────────────┘
      │
      ▼
Mark Delivered

If retries exhausted
        │
        ▼
 Dead Letter Queue (DLQ)
```

---

# Retry Flow

### Step 1: Notification created

```text
Notification
Status = PENDING
Retry Count = 0
```

Worker picks it up.

---

### Step 2: Delivery succeeds

```text
Status = DELIVERED
```

Done.

---

### Step 3: Temporary failure

Examples:

- Network timeout
- HTTP 500
- SMTP unavailable
- Push provider unavailable

Worker schedules retry.

```text
Retry Count = Retry Count + 1

Next Retry Time =
Current Time + Backoff Delay
```

---

### Step 4: Permanent failure

Examples:

- Invalid email
- Invalid phone number
- User blocked notifications
- HTTP 400 validation error

No retry.

```text
Status = FAILED
```

or

```text
Move to DLQ
```

---

# Retry Policy

Typical retry schedule:

| Retry | Delay      |
| ----- | ---------- |
| 1     | 1 minute   |
| 2     | 5 minutes  |
| 3     | 15 minutes |
| 4     | 1 hour     |
| 5     | 6 hours    |
| 6     | 24 hours   |

After maximum retries:

```text
FAILED
```

or

```text
Dead Letter Queue
```

---

# Exponential Backoff

Instead of retrying immediately:

```text
1 sec
2 sec
4 sec
8 sec
16 sec
32 sec
```

Formula:

```text
delay = base × 2^retryCount
```

Example:

```
Base = 5 sec

Retry 1 → 5 sec
Retry 2 → 10 sec
Retry 3 → 20 sec
Retry 4 → 40 sec
```

Benefits:

- Reduces load
- Gives providers time to recover
- Prevents retry storms

---

# Why Add Jitter?

Without jitter:

```text
100,000 failed notifications

↓

All retry after exactly 60 seconds

↓

Huge traffic spike
```

With jitter:

```text
Retry between

55–65 sec

or

60–90 sec
```

Traffic becomes evenly distributed.

Example:

```text
delay = exponential_backoff + random(0–10 sec)
```

---

# Retryable vs Non-Retryable Errors

| Error                  | Retry?        |
| ---------------------- | ------------- |
| Network timeout        | ✅ Yes        |
| HTTP 500               | ✅ Yes        |
| Rate limit (429)       | ✅ Yes        |
| SMTP unavailable       | ✅ Yes        |
| Invalid email          | ❌ No         |
| Invalid phone          | ❌ No         |
| Authentication failure | ❌ Usually No |
| Bad request (400)      | ❌ No         |

A key design decision is classifying errors correctly so resources aren't wasted retrying requests that can never succeed.

---

# Data Model

```text
Notification

id
userId
channel
payload
status
retryCount
maxRetries
nextRetryAt
lastError
createdAt
updatedAt
```

Indexes:

```
(status, nextRetryAt)
```

Allows efficient lookup:

```sql
WHERE status = 'RETRY'
AND nextRetryAt <= NOW()
```

---

# Retry Queue

Instead of scanning the database constantly:

```text
Notification

↓

Retry Queue

↓

Retry Worker

↓

Provider
```

Possible implementations:

- Delayed queues
- Scheduled jobs
- Priority queues

This scales better than polling large tables.

---

# Dead Letter Queue (DLQ)

When retries are exhausted:

```text
Notification

↓

DLQ
```

Reasons:

- Permanent failure
- Too many retries
- Corrupted payload
- Unexpected exceptions

Operators can:

- Inspect failures
- Replay messages
- Fix bugs
- Alert support teams

---

# Idempotency

A worker may crash after the provider accepts the notification but before the success is recorded:

```text
Worker

↓

Provider accepted

↓

Worker crashes

↓

Retry happens

↓

Duplicate notification
```

Solutions:

- Generate an **idempotency key** per notification.
- Include it in requests to providers that support idempotent operations.
- Record completed notification IDs locally and ignore duplicate processing.

---

# Scaling Strategy

### Queue-based architecture

```text
Producer

↓

Kafka / RabbitMQ / SQS

↓

Multiple Notification Workers
```

Workers scale horizontally.

---

### Partitioning

Partition by:

- User ID
- Notification ID
- Channel

Benefits:

- Parallel processing
- Better throughput
- Reduced contention

---

### Separate workers by channel

```text
Email Worker

SMS Worker

Push Worker
```

Advantages:

- Independent scaling
- Different retry policies
- Isolation from channel-specific outages

---

# Observability

Track metrics such as:

- Notifications sent
- Retry count
- Retry success rate
- Average retries per notification
- Time to delivery
- DLQ size
- Provider error rate
- Queue length
- Queue processing latency

Set alerts when:

- Retry rate spikes
- DLQ grows rapidly
- Queue backlog increases
- Delivery latency exceeds thresholds

---

# Trade-offs

| Approach                     | Advantages                                       | Disadvantages                              |
| ---------------------------- | ------------------------------------------------ | ------------------------------------------ |
| Immediate retry              | Simple                                           | Can overwhelm providers during outages     |
| Fixed delay                  | Easy to implement                                | Less adaptive to varying failure durations |
| Exponential backoff          | Reduces load and improves recovery               | Higher latency for eventual delivery       |
| Exponential backoff + jitter | Best scalability and avoids synchronized retries | Slightly more complex                      |
| Database polling             | Simple                                           | Inefficient at scale                       |
| Delayed message queues       | Efficient and scalable                           | Requires messaging infrastructure          |

---

# Interview-ready summary

"I would implement notifications asynchronously using a message queue and worker processes. Workers classify failures into retryable and non-retryable categories. Retryable failures are rescheduled using exponential backoff with jitter, while permanent failures are marked as failed immediately. Retry state, counts, and the next retry time are stored persistently, and notifications exceeding retry limits are moved to a Dead Letter Queue for investigation or replay. The system uses idempotency keys to prevent duplicate deliveries, scales horizontally with multiple workers, and is monitored through metrics such as retry rate, delivery latency, queue depth, and DLQ size."

## Question 2. How do you design a spell checker system?

## Question 3. How do you design a plagiarism checker?

## Question 4. How do you design a news feed ranking system?

## Question 5. How do you design a comment threading system?

## Question 6. How do you design a shopping cart system?

## Question 7. How do you design a dynamic pricing engine?

## Question 8. How do you design a taxi fare calculator system?

## Question 9. How do you design a leaderboard system for games?

## Question 10. How do you design a coupon/voucher redemption system?

## Question 11. What is write amplification in storage systems?

## Question 12. What are secondary indexes in NoSQL?

## Question 13. What is a time-series database (TSDB)?

## Question 14. How do you design a database for IoT data?

## Question 15. What is cold storage vs hot storage?

## Question 16. What is a columnar database? Give examples

## Question 17. What is a key-value store?

## Question 18. How do you design a database for full-text search?

## Question 19. What is an LSM tree (Log-Structured Merge Tree)?

## Question 20. How do you prevent data skew in distributed databases?
