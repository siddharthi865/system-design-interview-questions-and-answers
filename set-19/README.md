# Set 19

| S.No. | Question                                                                                                                                                    |
| ----- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you design a retry queue with exponential backoff?](#question-1-how-do-you-design-a-retry-queue-with-exponential-backoff)                           |
| 2.    | [How do you design a self-healing system?](#question-2-how-do-you-design-a-self-healing-system)                                                             |
| 3.    | [How do you handle zombie processes in distributed systems?](#question-3-how-do-you-handle-zombie-processes-in-distributed-systems)                         |
| 4.    | [What is replication lag and how do you mitigate it?](#question-4-what-is-replication-lag-and-how-do-you-mitigate-it)                                       |
| 5.    | [What is write-behind caching?](#question-5-what-is-write-behind-caching)                                                                                   |
| 6.    | [How do you design a strongly consistent distributed cache?](#question-6-how-do-you-design-a-strongly-consistent-distributed-cache)                         |
| 7.    | [What is the difference between sync checkpoints and async checkpoints?](#question-7-what-is-the-difference-between-sync-checkpoints-and-async-checkpoints) |
| 8.    | [How do you design a transactional outbox pattern?](#question-8-how-do-you-design-a-transactional-outbox-pattern)                                           |
| 9.    | [What is a saga pattern in distributed transactions?](#question-9-what-is-a-saga-pattern-in-distributed-transactions)                                       |
| 10.   | [How do you design dual writes handling in distributed systems?](#question-10-how-do-you-design-dual-writes-handling-in-distributed-systems)                |
| 11.   | [How do you design log ingestion pipelines at scale?](#question-11-how-do-you-design-log-ingestion-pipelines-at-scale)                                      |
| 12.   | [How do you store logs efficiently for 1 year?](#question-12-how-do-you-store-logs-efficiently-for-1-year)                                                  |
| 13.   | [What is histogram-based monitoring?](#question-13-what-is-histogram-based-monitoring)                                                                      |
| 14.   | [How do you monitor microservices dependency graphs?](#question-14-how-do-you-monitor-microservices-dependency-graphs)                                      |
| 15.   | [What is span sampling in tracing?](#question-15-what-is-span-sampling-in-tracing)                                                                          |
| 16.   | [How do you design an error reporting system like Sentry?](#question-16-how-do-you-design-an-error-reporting-system-like-sentry)                            |
| 17.   | [What is structured logging vs unstructured logging?](#question-17-what-is-structured-logging-vs-unstructured-logging)                                      |
| 18.   | [How do you implement health checks in microservices?](#question-18-how-do-you-implement-health-checks-in-microservices)                                    |
| 19.   | [What is blue/green monitoring?](#question-19-what-is-bluegreen-monitoring)                                                                                 |
| 20.   | [How do you design real-time metrics dashboards?](#question-20-how-do-you-design-real-time-metrics-dashboards)                                              |

## Question 1. How do you design a retry queue with exponential backoff?

## Direct answer

A **retry queue with exponential backoff** is a message-processing system that re-attempts failed tasks with increasing delays between retries (e.g., 1s → 2s → 4s → 8s), to prevent overwhelming downstream services and improve reliability under transient failures.

---

## Requirements / Problem Framing

### Functional requirements

- Accept failed tasks/messages for retry
- Retry automatically based on policy
- Apply exponential backoff with jitter
- Respect max retry limits
- Move permanently failing messages to a **dead-letter queue (DLQ)**

### Non-functional requirements

- High reliability (no message loss)
- Horizontal scalability
- Low contention under retry storms
- Idempotent processing support
- Observability (retry count, failure reasons)

---

## High-Level Architecture

```
          +-------------------+
Producer → | Main Queue        | → Worker Service
          +-------------------+         |
                                         | failure
                                         v
                                +-------------------+
                                | Retry Queue System|
                                +-------------------+
                                   |      |      |
                                   v      v      v
                         Delay Buckets / Scheduler / Timer Wheel
                                   |
                                   v
                          +------------------+
                          | Retry Worker     |
                          +------------------+
                                   |
                     success ------+------ failure
                                   |              |
                                   v              v
                              Success       Dead Letter Queue
```

---

## Core Design

### 1. Retry Queue Structure

Instead of a single queue, we typically use:

- **Main queue** → first attempt
- **Retry queues (delayed buckets)** → scheduled retries
- **DLQ** → permanent failures

Two common implementations:

| Approach                   | Description                                     | Trade-off                  |
| -------------------------- | ----------------------------------------------- | -------------------------- |
| Multiple delayed queues    | One queue per delay level (1s, 2s, 4s...)       | Simple but coarse-grained  |
| Timestamp-based scheduling | Single queue + scheduler checks visibility time | More scalable and flexible |

---

### 2. Exponential Backoff Logic

For each failed message:

```
delay = base * (2 ^ retry_count)
```

Example:

- retry 0 → 1s
- retry 1 → 2s
- retry 2 → 4s
- retry 3 → 8s

### Add jitter (critical in production)

To avoid thundering herd:

```
final_delay = random(0, delay)
or
final_delay = delay * (0.5 + random())
```

---

### 3. Message Schema

```json
{
  "id": "msg_123",
  "payload": {...},
  "retry_count": 2,
  "next_retry_at": 1710000000,
  "last_error": "timeout"
}
```

---

### 4. Retry Scheduler (Core Component)

Responsible for:

- Scanning retry queue
- Checking `next_retry_at <= now`
- Moving message to worker queue

Implementation options:

- Cron-based polling (simple)
- Priority queue (min-heap by timestamp)
- Redis sorted sets (common in production)

Example (Redis ZSET):

```
ZADD retry_queue <timestamp> <message_id>
```

---

### 5. Worker Flow

1. Consume message
2. Process task
3. If success → ACK
4. If failure:
   - increment retry_count
   - compute backoff delay
   - push to retry queue with updated timestamp
   - if retry_count > MAX → send to DLQ

---

## Deep Design Considerations

### 1. Avoiding Retry Storms

- Add jitter
- Spread retry buckets
- Rate limit retry workers

---

### 2. Idempotency (Critical)

Since retries can duplicate execution:

- Use idempotency keys per task
- Ensure downstream operations are safe to repeat

---

### 3. Retry Policies

Different strategies per failure type:

| Error type       | Strategy           |
| ---------------- | ------------------ |
| Network timeout  | retry with backoff |
| 4xx validation   | no retry → DLQ     |
| 5xx server error | retry              |
| Rate limit (429) | longer backoff     |

---

### 4. Priority Handling

You may want:

- High priority retries bypassing queue delays
- Separate queues per priority class

---

### 5. Storage Choice

| Storage                   | Pros                 | Cons                     |
| ------------------------- | -------------------- | ------------------------ |
| Redis ZSET                | fast, simple         | memory bound             |
| Kafka + delay topic       | durable, scalable    | complex scheduling       |
| RabbitMQ delayed exchange | native delay support | vendor-specific          |
| DB + scheduler            | simple               | not scalable at high QPS |

---

### 6. Failure Safety

- At-least-once delivery model
- Persistent retry state
- Crash-safe scheduler (checkpointing processed timestamps)

---

## Trade-offs

| Design Choice              | Pros              | Cons                            |
| -------------------------- | ----------------- | ------------------------------- |
| Fixed delay buckets        | simple            | inefficient backoff granularity |
| Timestamp scheduling       | precise, scalable | more complex                    |
| Immediate retries          | faster recovery   | risk of overload                |
| Strict exponential backoff | protects system   | slower recovery                 |

---

## Capacity / Scaling (Example)

Assume:

- 100K msgs/sec ingestion
- 5% failure rate → 5K/sec retries

Implications:

- Retry system must handle burst amplification
- Scheduler must be horizontally scalable
- Use partitioned retry queues (by hash of message_id)

---

## Security / Observability

### Observability

- retry_count histogram
- failure reason metrics
- DLQ rate alerts
- retry latency distribution

### Logging

- structured logs with message_id + attempt count

### Monitoring

- retry backlog size
- oldest retry age
- DLQ growth rate

---

## Interview-ready Summary

A retry queue with exponential backoff is a fault-tolerant messaging pattern where failed tasks are retried with increasing delays to avoid system overload. It typically consists of a main queue, a retry scheduling mechanism (often using timestamp-based delayed queues or Redis sorted sets), and a dead-letter queue for permanent failures. Exponential backoff combined with jitter prevents retry storms, while idempotency ensures safe reprocessing. The key design challenges are scalable scheduling, failure classification, and avoiding cascading overload during outages.

## Question 2. How do you design a self-healing system?

## Question 3. How do you handle zombie processes in distributed systems?

## Question 4. What is replication lag and how do you mitigate it?

## Question 5. What is write-behind caching?

## Question 6. How do you design a strongly consistent distributed cache?

## Question 7. What is the difference between sync checkpoints and async checkpoints?

## Question 8. How do you design a transactional outbox pattern?

## Question 9. What is a saga pattern in distributed transactions?

## Question 10. How do you design dual writes handling in distributed systems?

## Question 11. How do you design log ingestion pipelines at scale?

## Question 12. How do you store logs efficiently for 1 year?

## Question 13. What is histogram-based monitoring?

## Question 14. How do you monitor microservices dependency graphs?

## Question 15. What is span sampling in tracing?

## Question 16. How do you design an error reporting system like Sentry?

## Question 17. What is structured logging vs unstructured logging?

## Question 18. How do you implement health checks in microservices?

## Question 19. What is blue/green monitoring?

## Question 20. How do you design real-time metrics dashboards?
