# Set 16

| S.No. | Question                                                                                                                                                   |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What are trade-offs between consistency and availability?](#question-1-what-are-trade-offs-between-consistency-and-availability)                          |
| 2.    | [What is sticky caching?](#question-2-what-is-sticky-caching)                                                                                              |
| 3.    | [What is a cold cache vs a warm cache?](#question-3-what-is-a-cold-cache-vs-a-warm-cache)                                                                  |
| 4.    | [What is a dead letter queue (DLQ)?](#question-4-what-is-a-dead-letter-queue-dlq)                                                                          |
| 5.    | [What is an append-only log in distributed systems?](#question-5-what-is-an-append-only-log-in-distributed-systems)                                        |
| 6.    | [Explain optimistic vs pessimistic locking](#question-6-explain-optimistic-vs-pessimistic-locking)                                                         |
| 7.    | [What is eventual leader election?](#question-7-what-is-eventual-leader-election)                                                                          |
| 8.    | [What is quorum-based replication?](#question-8-what-is-quorum-based-replication)                                                                          |
| 9.    | [What is hard state vs soft state in distributed systems?](#question-9-what-is-hard-state-vs-soft-state-in-distributed-systems)                            |
| 10.   | [What are compensating transactions?](#question-10-what-are-compensating-transactions)                                                                     |
| 11.   | [How do you design a global flight tracking system like FlightRadar24?](#question-11-how-do-you-design-a-global-flight-tracking-system-like-flightradar24) |
| 12.   | [How do you design a fitness tracking app like Strava?](#question-12-how-do-you-design-a-fitness-tracking-app-like-strava)                                 |
| 13.   | [How do you design an online payment splitting app like Splitwise?](#question-13-how-do-you-design-an-online-payment-splitting-app-like-splitwise)         |
| 14.   | [How do you design an online photo gallery like Flickr?](#question-14-how-do-you-design-an-online-photo-gallery-like-flickr)                               |
| 15.   | [How do you design a ticketing system for large-scale events?](#question-15-how-do-you-design-a-ticketing-system-for-large-scale-events)                   |
| 16.   | [How do you design a podcast transcription system?](#question-16-how-do-you-design-a-podcast-transcription-system)                                         |
| 17.   | [How do you design a file diffing system like Git?](#question-17-how-do-you-design-a-file-diffing-system-like-git)                                         |
| 18.   | [How do you design a live sports commentary platform?](#question-18-how-do-you-design-a-live-sports-commentary-platform)                                   |
| 19.   | [How do you design a collaborative code editor like VSCode Live Share?](#question-19-how-do-you-design-a-collaborative-code-editor-like-vscode-live-share) |
| 20.   | [How do you design an app marketplace like Google Play Store?](#question-20-how-do-you-design-an-app-marketplace-like-google-play-store)                   |

## Question 1. What are trade-offs between consistency and availability?

# Trade-offs Between Consistency and Availability

## Direct answer

Consistency and availability are two fundamental goals in distributed systems, but during a network partition you generally cannot guarantee both simultaneously. This is the core idea behind the CAP theorem.

- **Consistency (C):** Every client sees the latest committed data, regardless of which server they access.
- **Availability (A):** Every request receives a response, even if some servers are down or disconnected.

When a **network partition (P)** occurs, a distributed system must choose between:

- **Consistency:** Reject or delay some requests until replicas synchronize.
- **Availability:** Continue serving requests, even if different replicas temporarily return different values.

---

## Intuition

Imagine a banking system replicated across two data centers.

```
           Network Partition

     DC-A  XXXXXXXXXXXXX  DC-B
```

A user deposits ₹100 into DC-A.

Another user immediately checks the balance from DC-B.

The system has two choices:

### Option 1: Prioritize Consistency

DC-B refuses or delays the read because it cannot verify the latest balance.

```
Write -> Success

Read -> Wait / Error
```

**Result**

- Everyone sees the same data.
- Some requests fail or time out.

---

### Option 2: Prioritize Availability

DC-B immediately returns the locally stored balance.

```
Write -> Success

Read -> Old balance
```

**Result**

- System always responds.
- Users may temporarily see stale data.

---

## Comparison

| Aspect                    | Consistency     | Availability         |
| ------------------------- | --------------- | -------------------- |
| User always gets response | ❌ Not always   | ✅ Yes               |
| Latest data guaranteed    | ✅ Yes          | ❌ Not always        |
| Stale reads possible      | ❌ No           | ✅ Yes               |
| Writes during partition   | May be rejected | Usually accepted     |
| Latency                   | Higher          | Lower                |
| Data correctness          | Highest         | Eventually converges |

---

## Real-world examples

### Consistency-first systems

Examples:

- Distributed SQL databases
- Banking systems
- Payment processing
- Inventory management

Why?

Incorrect data is more harmful than temporary downtime.

Example:

```
Account Balance

Actual: ₹15,000

Every user sees:
₹15,000
```

Even if some requests are temporarily unavailable.

---

### Availability-first systems

Examples:

- Social media feeds
- News feeds
- Product catalogs
- DNS

Why?

Users prefer getting slightly stale data over receiving an error.

Example:

```
Like Count

Replica A:
100

Replica B:
98
```

Both responses are acceptable because the replicas will eventually synchronize.

---

## Eventual consistency

Many modern distributed systems choose **availability** and rely on **eventual consistency**.

```
Time 0

Replica A = 100
Replica B = 95

↓

Replication

↓

Replica B = 100
```

Eventually, all replicas converge to the same state.

---

## When to prioritize each

### Prioritize consistency when

- Banking
- Payments
- Financial ledgers
- Seat reservations
- Stock trading
- Inventory with limited quantities

Incorrect data can cause financial loss or double-booking.

---

### Prioritize availability when

- Social media
- Chat presence indicators
- Analytics dashboards
- News feeds
- Product recommendations
- Video streaming metadata

Temporary inconsistencies are acceptable if the service remains responsive.

---

## Trade-offs

| Choose Consistency                 | Choose Availability                         |
| ---------------------------------- | ------------------------------------------- |
| Correct data                       | Faster responses                            |
| May reject requests                | Always responds                             |
| Higher latency                     | Lower latency                               |
| Better for financial systems       | Better for user-facing web apps             |
| Reduced user-visible inconsistency | Temporary stale reads or conflicting writes |

---

## Interview-ready summary

> Consistency ensures every client sees the latest data, while availability ensures every request receives a response. According to the CAP theorem, when a network partition occurs, a distributed system must choose between these two properties. Consistency-first systems may reject or delay requests to preserve correctness, making them suitable for domains like banking and payments. Availability-first systems continue serving requests despite temporary inconsistencies, making them ideal for applications such as social media and content delivery. The right choice depends on whether correctness or uninterrupted service is more critical for the application's requirements.

## Question 2. What is sticky caching?

# Sticky Caching

## Direct answer

**Sticky caching** is a caching strategy where requests from the same user, client, or session are consistently routed to the same server, allowing that server's **local cache** to be reused effectively. This improves cache hit rates and reduces latency because the cached data is already present on that server.

Sticky caching is commonly implemented using **sticky sessions (session affinity)** at the load balancer.

---

## How it works

Suppose you have three application servers, each with its own in-memory cache.

```text
                Load Balancer
                     |
      +--------------+--------------+
      |              |              |
   Server A       Server B       Server C
   Local Cache    Local Cache    Local Cache
```

If User A always gets routed to **Server A**, then:

1. First request:
   - Cache miss
   - Server A fetches data from the database.
   - Stores it in its local cache.

2. Subsequent requests:
   - User A continues hitting Server A.
   - Data is served directly from the local cache.

```text
User A
   │
   ▼
Load Balancer
   │
   ▼
Server A
(Cache Hit)
```

Without sticky routing, User A's requests might go to different servers, each experiencing cache misses until their local caches are populated.

---

## Benefits

- **Higher cache hit rate** for repeated requests from the same user.
- **Lower latency** because data is served from local memory.
- **Reduced database load** due to fewer repeated queries.
- Enables use of fast in-memory caches without requiring every cache lookup to go to a distributed cache.

---

## Drawbacks

- **Uneven load distribution:** Some servers may become hotspots if many active users are pinned to them.
- **Poor fault tolerance:** If a server fails, its local cache is lost and users are redirected elsewhere, causing cache misses.
- **Harder to scale:** Adding or removing servers can redistribute users, reducing cache effectiveness.
- **Cache duplication:** The same data may be stored independently on multiple servers, increasing memory usage.

---

## Sticky caching vs. distributed caching

| Aspect          | Sticky Caching                              | Distributed Cache             |
| --------------- | ------------------------------------------- | ----------------------------- |
| Cache location  | Local to each application server            | Shared cache cluster          |
| Routing         | Requires session affinity                   | Any server can serve requests |
| Cache hit rate  | High for repeat requests to the same server | High across all servers       |
| Scalability     | Limited by sticky routing                   | Better horizontal scalability |
| Fault tolerance | Lower                                       | Higher                        |
| Examples        | Local in-memory caches                      | Redis, Memcached              |

---

## When to use sticky caching

Sticky caching is a good fit when:

- Users make many repeated requests during a session.
- Cached data is relatively small and frequently reused.
- Very low latency is important.
- The application already uses sticky sessions.

Examples include:

- User profile pages
- Shopping carts stored in memory
- Dashboard data
- Frequently accessed session-specific information

---

## When not to use it

Avoid sticky caching when:

- The system must scale across many application servers.
- High availability is critical.
- Requests should be evenly distributed.
- Multiple servers need to share the same cached data.

In these cases, a distributed cache such as Redis is typically a better choice.

---

## Interview-ready summary

> Sticky caching keeps requests from the same user on the same application server so that the server's local cache can be reused. This improves cache hit rates, reduces latency, and lowers database load. However, it introduces uneven load distribution, reduces fault tolerance, and makes scaling more difficult. For large-scale distributed systems, a shared distributed cache is generally preferred over sticky caching.

## Question 3. What is a cold cache vs a warm cache?

# Cold Cache vs. Warm Cache

## Direct answer

A **cold cache** is a cache that is empty or contains very little useful data, resulting in many **cache misses**. A **warm cache** has already been populated with frequently accessed data, resulting in a high **cache hit rate** and lower latency.

The difference mainly affects **performance**, not correctness.

---

## Cold Cache

A cold cache occurs when:

- A cache server has just started.
- The cache has been cleared or expired.
- A new node has been added to the cluster.
- Data has never been requested before.

Example:

```text
User Request
      │
      ▼
   Cache (Empty)
      │
 Cache Miss
      │
      ▼
   Database
      │
      ▼
 Cache Updated
      │
      ▼
 User Response
```

### Characteristics

- Many cache misses
- Higher database load
- Higher latency
- Slower response times
- Cache gradually fills with frequently accessed data

---

## Warm Cache

A warm cache contains data that has already been loaded from previous requests.

Example:

```text
User Request
      │
      ▼
 Cache
      │
 Cache Hit
      │
      ▼
 User Response
```

### Characteristics

- High cache hit rate
- Lower latency
- Reduced database traffic
- Better throughput
- Faster user experience

---

## Comparison

| Aspect              | Cold Cache            | Warm Cache                              |
| ------------------- | --------------------- | --------------------------------------- |
| Cache contents      | Empty or mostly empty | Frequently accessed data already stored |
| Cache hit rate      | Low                   | High                                    |
| Database load       | High                  | Low                                     |
| Response time       | Slower                | Faster                                  |
| Startup performance | Poor                  | Good                                    |
| User experience     | Initial slowdown      | Smooth and responsive                   |

---

## Real-world example

Imagine an e-commerce website.

### Cold cache

The cache is empty after a deployment.

```text
Request:
Product #123

Cache → Miss
Database → Read
Cache ← Store
Response
```

The first user experiences higher latency.

---

### Warm cache

Later requests for the same product:

```text
Request:
Product #123

Cache → Hit
Response
```

No database query is needed, making the response much faster.

---

## How to warm a cache

Many production systems avoid cold-start performance by proactively populating caches.

Common techniques include:

- **Preloading popular data** during application startup
- **Background cache warming** jobs
- **Scheduled jobs** to refresh frequently accessed data
- **Lazy loading**, where the first request populates the cache
- **Predictive warming** based on historical traffic patterns

Example:

```text
Application Starts
        │
        ▼
Background Job
        │
        ▼
Load Top 10,000 Products
        │
        ▼
Cache Ready
```

---

## Trade-offs

| Cold Cache             | Warm Cache                             |
| ---------------------- | -------------------------------------- |
| Faster startup         | Requires time or resources to populate |
| Higher initial latency | Lower request latency                  |
| More database traffic  | Less database traffic                  |
| Simpler deployment     | May require cache warming strategies   |

---

## Interview-ready summary

> A cold cache is empty or sparsely populated, so requests experience frequent cache misses, higher latency, and increased database load. A warm cache already contains commonly accessed data, resulting in high cache hit rates, lower latency, and better throughput. Large-scale systems often use cache warming techniques—such as preloading popular data or background refresh jobs—to minimize the performance impact of cold starts after deployments or restarts.

## Question 4. What is a dead letter queue (DLQ)?

# Dead Letter Queue (DLQ)

## Direct answer

A **Dead Letter Queue (DLQ)** is a special queue that stores messages that **cannot be successfully processed** after a configured number of retry attempts. Instead of repeatedly failing and blocking the main queue, these messages are isolated for later inspection, debugging, or manual reprocessing.

DLQs improve the **reliability** and **resilience** of message-driven systems.

---

## Why do we need a DLQ?

In distributed systems, message processing can fail for various reasons:

- Invalid message format
- Corrupted data
- Missing required fields
- Downstream service failures
- Business logic errors
- Unexpected exceptions

Without a DLQ:

```text
Producer
    │
    ▼
Main Queue
    │
    ▼
Consumer
    │
 Processing Fails
    │
    └──────────► Retry Forever
```

Problems:

- Infinite retry loops
- Wasted compute resources
- Queue backlog grows
- Valid messages are delayed (head-of-line blocking)
- Difficult to identify problematic messages

---

## How a DLQ works

A message is retried a limited number of times.

If it still fails, it is moved to the DLQ.

```text
Producer
    │
    ▼
Main Queue
    │
    ▼
Consumer
    │
Success ─────────► Done

Failure
    │
Retry 1
    │
Retry 2
    │
Retry 3
    │
Still Fails
    ▼
Dead Letter Queue (DLQ)
```

The main queue continues processing other messages while failed ones are safely isolated.

---

## What is stored in a DLQ?

A DLQ typically stores:

- Original message payload
- Message ID
- Timestamp
- Retry count
- Error message or exception
- Stack trace (optional)
- Metadata (topic, partition, headers, etc.)

This information helps engineers diagnose the root cause.

---

## Common use cases

### 1. Invalid data

```json
{
  "userId": null,
  "amount": 500
}
```

The consumer requires a valid `userId`.

After multiple failed retries, the message is sent to the DLQ.

---

### 2. Payment processing

```text
Order Created
      │
      ▼
Payment Service
      │
Database Exception
      │
Retry
      │
Retry
      │
Retry
      ▼
DLQ
```

Operations teams can later inspect and replay the message after fixing the issue.

---

### 3. Email notifications

Suppose an email message contains an invalid recipient address.

Rather than blocking all other emails, only the invalid message is moved to the DLQ.

---

## Benefits

- Prevents infinite retry loops
- Keeps the main queue flowing
- Improves fault isolation
- Simplifies debugging
- Enables manual or automated replay after issues are resolved
- Preserves failed messages instead of losing them

---

## Best practices

### Set a retry limit

Example:

- Retry 1
- Retry 2
- Retry 3
- Move to DLQ

Avoid retrying indefinitely.

---

### Use exponential backoff

Instead of retrying immediately:

```text
Retry 1 → 1 sec

Retry 2 → 5 sec

Retry 3 → 30 sec
```

This reduces pressure on a failing downstream service.

---

### Monitor the DLQ

Track metrics such as:

- Number of messages in the DLQ
- Rate of new failed messages
- Oldest message age
- Failure reasons

A rapidly growing DLQ often indicates a systemic issue.

---

### Enable replay

After fixing the root cause:

```text
DLQ
   │
Replay
   │
Main Queue
   │
Consumer
```

Replay should be **idempotent** so that processing the same message again does not produce duplicate side effects.

---

## Trade-offs

| Advantages                          | Disadvantages                                  |
| ----------------------------------- | ---------------------------------------------- |
| Prevents blocking of the main queue | Requires operational monitoring                |
| Preserves failed messages           | Uses additional storage                        |
| Easier debugging                    | Replay logic adds complexity                   |
| Improves system reliability         | Doesn't fix the underlying cause automatically |

---

## Real-world systems

Many messaging systems support DLQs natively or through configuration, including:

- Apache Kafka (using dead-letter topics)
- RabbitMQ
- Amazon Simple Queue Service (Dead-Letter Queues)
- Azure Service Bus
- Google Cloud Pub/Sub

---

## Interview-ready summary

> A Dead Letter Queue (DLQ) is a dedicated queue for messages that cannot be processed successfully after a predefined number of retries. Instead of repeatedly retrying and blocking the main queue, failed messages are isolated for debugging, monitoring, and later replay. DLQs improve reliability, fault isolation, and operational visibility, and are a standard pattern in resilient event-driven architectures.

## Question 5. What is an append-only log in distributed systems?

# Append-Only Log in Distributed Systems

## Direct answer

An **append-only log** is a data structure where **new records are always added to the end of the log**, and existing records are **never modified or deleted**. Instead of updating data in place, every change is recorded as a new immutable event.

This design is fundamental to many distributed systems because it provides **durability, fault tolerance, replication, recovery, and event ordering**.

---

## How it works

Instead of updating records:

```text
Balance = 1000

UPDATE Balance = 1200
```

An append-only log records every change:

```text
Offset  Event
------  ---------------------
0       Account Created
1       Deposit ₹1000
2       Withdraw ₹200
3       Deposit ₹400
4       Withdraw ₹100
```

Nothing is overwritten.

The current balance is obtained by replaying the events:

```text
0 + 1000 - 200 + 400 - 100 = ₹1100
```

---

## Why use an append-only log?

### 1. Durability

Once an event is written, it remains unchanged.

```text
Append Event
      │
      ▼
Persist to Disk
      │
Never Modified
```

This reduces the risk of data corruption from in-place updates.

---

### 2. Easy replication

Followers simply copy new log entries in order.

```text
Leader
  │
  ├── Event 101
  ├── Event 102
  └── Event 103
        │
        ▼
Followers append the same events
```

This approach is used by consensus protocols like Raft consensus algorithm.

---

### 3. Crash recovery

If a server crashes:

```text
Read Log
   │
Replay Events
   │
Restore State
```

Since the log is durable, the application can rebuild its state by replaying events.

---

### 4. Event ordering

Every event has a unique position (often called an **offset**).

```text
Offset
0
1
2
3
4
```

Consumers process events sequentially, ensuring a deterministic order.

---

## Architecture

```text
Producers
     │
     ▼
Append-Only Log
+----------------------+
| Event 1              |
| Event 2              |
| Event 3              |
| Event 4              |
+----------------------+
     │
     ▼
Consumers
```

Multiple consumers can independently read the same log at their own pace.

---

## Real-world examples

Many distributed systems are built around append-only logs:

- Apache Kafka stores messages as immutable log entries.
- Apache Pulsar uses persistent append-only storage.
- Apache BookKeeper provides durable append-only logs.
- Consensus algorithms like Raft consensus algorithm and Paxos replicate state through append-only logs.

---

## Advantages

- High write throughput (sequential disk writes are efficient)
- Immutable history for auditing
- Simplifies replication
- Enables reliable recovery after crashes
- Preserves event ordering
- Supports replay and time-travel debugging

---

## Disadvantages

- Logs grow continuously and require retention or compaction policies.
- Reading the latest state may require replaying many events unless snapshots or materialized views are maintained.
- Additional storage is needed because old events are retained.

---

## Append-only log vs. traditional database updates

| Aspect            | Append-Only Log               | Traditional Updates                      |
| ----------------- | ----------------------------- | ---------------------------------------- |
| Updates           | Add new records               | Modify existing records                  |
| History           | Complete history retained     | Usually only current state               |
| Auditability      | Excellent                     | Limited unless audit tables exist        |
| Recovery          | Replay log                    | Restore from backups or transaction logs |
| Write performance | Excellent (sequential writes) | Can involve random writes                |
| Storage           | Higher                        | Lower                                    |

---

## Common use cases

- Event-driven architectures
- Distributed messaging systems
- Consensus algorithms
- Database transaction logs
- Event sourcing
- Change Data Capture (CDC)
- Audit logging

---

## Interview-ready summary

> An append-only log is an immutable sequence of records where new events are always appended and existing records are never modified. This design enables efficient sequential writes, reliable replication, deterministic event ordering, crash recovery through log replay, and complete audit history. It's a foundational concept in distributed systems and is widely used in technologies like Apache Kafka and in consensus protocols such as Raft consensus algorithm.

## Question 6. Explain optimistic vs pessimistic locking

## Question 7. What is eventual leader election?

## Question 8. What is quorum-based replication?

## Question 9. What is hard state vs soft state in distributed systems?

## Question 10. What are compensating transactions?

## Question 11. How do you design a global flight tracking system like FlightRadar24?

## Question 12. How do you design a fitness tracking app like Strava?

## Question 13. How do you design an online payment splitting app like Splitwise?

## Question 14. How do you design an online photo gallery like Flickr?

## Question 15. How do you design a ticketing system for large-scale events?

## Question 16. How do you design a podcast transcription system?

## Question 17. How do you design a file diffing system like Git?

## Question 18. How do you design a live sports commentary platform?

## Question 19. How do you design a collaborative code editor like VSCode Live Share?

## Question 20. How do you design an app marketplace like Google Play Store?
