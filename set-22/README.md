# Set 22

| S.No. | Question                                                                                                                                                                                  |
| ----- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How would you design a distributed cache invalidation strategy?](#question-1-how-would-you-design-a-distributed-cache-invalidation-strategy)                                             |
| 2.    | [How do you ensure idempotency in payment processing APIs?](#question-2-how-do-you-ensure-idempotency-in-payment-processing-apis)                                                         |
| 3.    | [What is the difference between at-most-once, at-least-once, and exactly-once delivery?](#question-3-what-is-the-difference-between-at-most-once-at-least-once-and-exactly-once-delivery) |
| 4.    | [How do you design a real-time typing indicator system for a chat app?](#question-4-how-do-you-design-a-real-time-typing-indicator-system-for-a-chat-app)                                 |
| 5.    | [What is a vector clock and how is it used in conflict resolution?](#question-5-what-is-a-vector-clock-and-how-is-it-used-in-conflict-resolution)                                         |
| 6.    | [How do you design a low-latency leaderboard system?](#question-6-how-do-you-design-a-low-latency-leaderboard-system)                                                                     |
| 7.    | [Explain how DNS load balancing works under the hood](#question-7-explain-how-dns-load-balancing-works-under-the-hood)                                                                    |
| 8.    | [How would you store billions of IoT device telemetry logs efficiently?](#question-8-how-would-you-store-billions-of-iot-device-telemetry-logs-efficiently)                               |
| 9.    | [What is hot partitioning in distributed systems, and how do you solve it?](#question-9-what-is-hot-partitioning-in-distributed-systems-and-how-do-you-solve-it)                          |
| 10.   | [How do you implement distributed tracing in microservices?](#question-10-how-do-you-implement-distributed-tracing-in-microservices)                                                      |
| 11.   | [What are quorum reads and quorum writes in distributed databases?](#question-11-what-are-quorum-reads-and-quorum-writes-in-distributed-databases)                                        |
| 12.   | [How would you design a feature flagging system?](#question-12-how-would-you-design-a-feature-flagging-system)                                                                            |
| 13.   | [What is a tombstone in Cassandra, and why is it important?](#question-13-what-is-a-tombstone-in-cassandra-and-why-is-it-important)                                                       |
| 14.   | [How do you design a system to support multiple time zones efficiently?](#question-14-how-do-you-design-a-system-to-support-multiple-time-zones-efficiently)                              |
| 15.   | [How would you implement pagination for large datasets?](#question-15-how-would-you-implement-pagination-for-large-datasets)                                                              |
| 16.   | [What is the difference between logical and physical data models?](#question-16-what-is-the-difference-between-logical-and-physical-data-models)                                          |
| 17.   | [How would you design a video transcoding pipeline at scale?](#question-17-how-would-you-design-a-video-transcoding-pipeline-at-scale)                                                    |
| 18.   | [How do you ensure GDPR compliance in data storage design?](#question-18-how-do-you-ensure-gdpr-compliance-in-data-storage-design)                                                        |
| 19.   | [What is a CRDT, and how does it solve real-time sync problems?](#question-19-what-is-a-crdt-and-how-does-it-solve-real-time-sync-problems)                                               |
| 20.   | [How would you design an alerting system for server downtime?](#question-20-how-would-you-design-an-alerting-system-for-server-downtime)                                                  |

## Question 1. How would you design a distributed cache invalidation strategy?

# Direct answer

A **distributed cache invalidation strategy** ensures that when the source of truth (typically a database) changes, all cached copies across multiple application instances or cache nodes are updated or removed consistently.

There is no one-size-fits-all solution. The best strategy depends on consistency requirements:

- **Cache-aside + event-driven invalidation** is the most common and scalable approach.
- **TTL-based expiration** provides eventual consistency with simplicity.
- **Write-through/write-behind** can reduce stale data for specific workloads.
- **Versioning or cache generations** help avoid race conditions.
- Large systems often combine multiple techniques.

---

# Requirements / Problem Framing

### Functional Requirements

- Cache frequently accessed data
- Invalidate stale entries after updates
- Support multiple application servers
- Keep cache reasonably consistent with the database

### Non-functional Requirements

- Low read latency
- High throughput
- High availability
- Minimal stale reads
- Horizontal scalability

---

# High-Level Architecture

```text
                Read
Client -----------------> Application
                              |
                     Cache (Redis/Memcached)
                     |        ^
      Cache Miss     |        |
                     v        |
                  Database    |
                     |        |
              Data Updated    |
                     |        |
             Publish Event ---+
                     |
          Message Broker (Kafka/Pub/Sub)
                     |
        +------------+-------------+
        |                          |
 Service A                  Service B
Invalidate Cache         Invalidate Cache
```

Flow:

1. Client requests data.
2. Application checks cache.
3. Cache miss → read database.
4. Store value in cache.
5. Database update occurs.
6. Publish cache invalidation event.
7. Every service removes or refreshes its cache entry.

---

# Common Cache Invalidation Strategies

## 1. TTL (Time-To-Live)

Every cached object expires automatically.

```
User:123
TTL = 5 minutes
```

### Advantages

- Extremely simple
- No coordination needed
- Works well for rarely changing data

### Disadvantages

- Data may remain stale until expiration
- Choosing TTL is difficult

Good for:

- Product catalogs
- Static configurations
- Public content

---

## 2. Explicit Delete (Cache Aside)

After updating the database:

```
Update DB

↓

Delete cache key

↓

Next read reloads cache
```

Example

```
UPDATE users SET name='Alice'

↓

DEL user:123
```

Next request:

```
Cache Miss

↓

Read DB

↓

Cache New Value
```

Advantages

- Very common
- Easy to implement
- Database remains source of truth

Challenge

Race conditions if concurrent updates occur.

---

## 3. Event-Based Invalidation

Every write publishes an event.

```
User Updated

↓

Kafka

↓

All services receive event

↓

Delete local cache
```

Advantages

- Works across many services
- Supports microservices
- Scales well

Used by:

- Large e-commerce systems
- Social networks
- Distributed microservices

---

## 4. Write Through

Every write updates:

```
Application

↓

Cache

↓

Database
```

Advantages

- Cache always updated
- No stale cache after writes

Disadvantages

- Higher write latency
- Every write touches cache

Good for

- Frequently read hot objects

---

## 5. Write Around

```
Write DB

↓

Do not cache immediately

↓

Cache on future read
```

Advantages

- Avoids caching cold data
- Lower cache pollution

---

## 6. Write Behind (Write Back)

```
Application

↓

Cache Updated

↓

Database Updated Later
```

Advantages

- Very fast writes
- Good for heavy write workloads

Risks

- Possible data loss
- Complex recovery
- Eventual consistency only

---

# Distributed Cache Invalidation Techniques

## Pub/Sub

```
DB Updated

↓

Redis Pub/Sub

↓

Every Application Server

↓

Delete Local Cache
```

Very common for:

- Redis
- Hazelcast
- Ignite

---

## Kafka/Event Bus

Instead of Redis Pub/Sub:

```
Update

↓

Kafka Topic

↓

Multiple Consumers

↓

Invalidate Cache
```

Benefits

- Durable
- Replayable
- Better for large systems

---

## Version Numbers

Instead of deleting immediately:

```
user:123:v8
```

After update

```
user:123:v9
```

Old version becomes unused.

Advantages

- Eliminates stale overwrite races
- Easier concurrent updates

---

## Generational Keys

Instead of

```
product:10
```

Use

```
product:v17:10
```

Changing version invalidates an entire namespace without deleting millions of keys.

---

## Tag-Based Invalidation

Associate cache entries with tags.

Example

```
Product:101

Tags:
electronics
phones
```

Updating the category invalidates all tagged entries.

Useful for:

- CMS
- Search indexes
- Catalog systems

---

# Race Conditions

Consider:

```
Thread A:
Read DB

↓

Slow

↓

Write Cache
```

Meanwhile

```
Thread B

Update DB

↓

Delete Cache
```

Thread A finally writes the old value into cache.

Result:

```
Database:
New Value

Cache:
Old Value
```

Solutions

- Version numbers
- Compare-and-set (CAS)
- Distributed locks (used sparingly)
- Cache population with timestamps
- Short TTLs as a safety net

---

# Deep Design Considerations

### Hot Keys

Frequently accessed keys may create contention.

Solutions:

- Replicate hot keys
- Local in-memory caches
- Request coalescing
- Read-through caching

---

### Cache Stampede

Many requests hit an expired key simultaneously.

Solutions:

- Single-flight loading
- Distributed locking
- Early refresh
- Probabilistic expiration

---

### Partial Failure

Some services may miss invalidation events.

Mitigations:

- TTL fallback
- Durable event queues
- Periodic cache reconciliation

---

### Ordering

Updates:

```
Version 5

↓

Version 6

↓

Version 4 arrives late
```

Need:

- Ordered events
- Version checks before applying invalidation

---

# Trade-offs

| Strategy      | Read Speed | Write Cost | Consistency | Complexity |
| ------------- | ---------- | ---------- | ----------- | ---------- |
| TTL           | High       | Low        | Eventual    | Low        |
| Cache Aside   | High       | Medium     | Good        | Low        |
| Event Driven  | High       | Medium     | Very Good   | Medium     |
| Write Through | High       | High       | Stronger    | Medium     |
| Write Behind  | Very High  | Low        | Eventual    | High       |
| Versioning    | High       | Medium     | Excellent   | Medium     |

---

# Security / Observability

Monitor key cache metrics such as:

- Cache hit ratio
- Cache miss ratio
- Invalidation event latency
- Event processing failures
- Stale read rate
- Cache eviction rate
- Cache node memory utilization

Implement structured logging and distributed tracing around cache reads, writes, and invalidation events to quickly diagnose consistency issues.

---

# Interview-ready summary

> "In a distributed system, I typically use a **cache-aside pattern** with **event-driven invalidation**. After updating the database, the service publishes an invalidation event through a message broker such as Kafka, and all application instances remove or refresh the affected cache entries. I combine this with **TTLs** as a safety net, **versioned cache keys** to prevent stale-write race conditions, and techniques like request coalescing and early refresh to avoid cache stampedes. This approach provides a good balance between scalability, performance, and consistency for most production systems."

## Question 2. How do you ensure idempotency in payment processing APIs?

# Direct answer

**Idempotency** in payment processing means that **multiple identical requests produce the same result as a single request**, preventing duplicate charges caused by retries, network failures, or client timeouts.

The standard approach is to require clients to send a unique **Idempotency-Key** with every payment request. The server stores the key and the result of the first successful execution. Any subsequent request with the same key returns the previously stored response instead of processing the payment again.

---

# Requirements / Problem Framing

### Functional Requirements

- Prevent duplicate payment processing.
- Support safe retries after network failures or timeouts.
- Return the same response for duplicate requests.
- Handle concurrent duplicate requests correctly.

### Non-functional Requirements

- High reliability and consistency.
- Low latency for retries.
- Scalable across multiple application instances.
- Fault tolerant.

---

# High-Level Architecture

```text
                POST /payments
      +------------------------------+
      | Idempotency-Key: abc123      |
      +--------------+---------------+
                     |
                     v
              API Gateway / LB
                     |
                     v
             Payment Service
                     |
         Check Idempotency Store
             /                \
      Key Exists?          Key Missing?
        |                      |
 Return Stored           Create "Processing"
 Response                Record (Atomic)
                               |
                               v
                     Call Payment Gateway
                               |
                        Payment Successful
                               |
                               v
                  Store Response + Status
                               |
                               v
                       Return Response
```

---

# Request Flow

### First Request

```text
POST /payments

Idempotency-Key: payment-123
Amount: $100
```

Steps:

1. Check if `payment-123` exists.
2. It doesn't.
3. Create an idempotency record atomically.
4. Process payment.
5. Save the response.
6. Return success.

---

### Retry Request

The client retries because it didn't receive the response.

```text
POST /payments

Idempotency-Key: payment-123
```

Server:

1. Finds existing record.
2. Does **not** charge again.
3. Returns the stored response.

---

# Idempotency Store

A dedicated table (or Redis with persistence) stores request metadata.

| Idempotency Key | Request Hash | Status  | Response   | Created At |
| --------------- | ------------ | ------- | ---------- | ---------- |
| payment-123     | hash(req)    | SUCCESS | Payment ID | Timestamp  |

### Why store the request hash?

If someone reuses the same key with different request data:

```text
Key: payment-123

Amount = $100
```

Later:

```text
Key: payment-123

Amount = $500
```

The server compares the request hash and returns an error because the key is being reused for a different operation.

---

# Handling Concurrent Requests

Two identical requests may arrive simultaneously.

Without protection:

```text
Request A → Charge Card

Request B → Charge Card
```

Result:

- Customer charged twice.

### Solution

Use an atomic insert or unique constraint.

```sql
INSERT INTO idempotency_keys(key)
VALUES('payment-123');
```

Only one insert succeeds.

The other request:

- Waits for completion, or
- Returns the stored result once available.

This guarantees only one payment execution.

---

# State Machine

```text
NEW

↓

PROCESSING

↓

SUCCESS
```

or

```text
NEW

↓

PROCESSING

↓

FAILED
```

If another request arrives while the status is `PROCESSING`, it can:

- Wait briefly and poll.
- Return `202 Accepted`.
- Return a "Request in progress" response.

This prevents duplicate execution during in-flight processing.

---

# Deep Design Considerations

## Atomicity

Creating the idempotency record and marking it as `PROCESSING` should be atomic to prevent race conditions.

Possible implementations:

- Database transaction.
- `INSERT ... ON CONFLICT DO NOTHING`.
- Redis `SETNX`.

---

## Storage Duration

Idempotency records shouldn't live forever.

Typical retention:

- 24 hours
- 48 hours
- 7 days (depending on business requirements)

Expired records are cleaned up periodically.

---

## Exactly-Once vs At-Least-Once

True **exactly-once delivery** is nearly impossible in distributed systems.

The practical goal is:

- Requests may be delivered **at least once**.
- The API processes the payment **exactly once** using idempotency.

---

## External Payment Gateway

Many payment gateways also support idempotency keys.

Flow:

```text
Client
    |
Payment Service
    |
Stripe/Adyen/etc.
    |
Idempotency-Key forwarded
```

Even if your service retries the gateway call, the gateway prevents duplicate charges.

---

## Failure Scenarios

### Response Lost

```text
Charge succeeds

↓

Network timeout

↓

Client retries
```

The stored response is returned without charging again.

---

### Server Crash

```text
Status = PROCESSING

↓

Server crashes
```

Recovery options:

- Background worker reconciles with the payment gateway.
- Check whether the payment was actually completed.
- Update the record to `SUCCESS` or `FAILED` before allowing retries.

---

# Trade-offs

| Approach                    | Pros                                | Cons                                                                                    |
| --------------------------- | ----------------------------------- | --------------------------------------------------------------------------------------- |
| Idempotency Key             | Industry standard, simple, scalable | Requires client cooperation                                                             |
| Database Unique Constraint  | Prevents duplicate inserts          | Doesn't handle response replay by itself                                                |
| Distributed Lock            | Prevents concurrent execution       | Higher latency, lock management complexity                                              |
| Request Fingerprinting Only | No client-generated key needed      | Harder to distinguish intentional duplicate payments from legitimate repeated purchases |

---

# Security / Observability

**Security**

- Generate high-entropy idempotency keys (typically UUIDs).
- Validate that the same key isn't reused with different payloads.
- Scope keys to the authenticated customer or merchant to prevent cross-user collisions.
- Encrypt sensitive payment data; never store raw card details in the idempotency store.

**Observability**

Track metrics such as:

- Duplicate request rate.
- Idempotency cache hit rate.
- Concurrent duplicate requests.
- Requests stuck in `PROCESSING`.
- Payment success/failure rates.
- Retry frequency and latency.

Log the idempotency key, request ID, and payment ID together to simplify debugging and distributed tracing.

---

# Interview-ready summary

> "For payment APIs, I ensure idempotency by requiring clients to send a unique **Idempotency-Key** with every payment request. The server atomically creates an idempotency record before processing the payment, stores the final response, and returns that same response for any retries with the same key. I protect against concurrent duplicate requests using a unique constraint or `SETNX`, verify that the same key isn't reused with a different payload by storing a request hash, and periodically expire old idempotency records. This approach prevents duplicate charges while allowing clients to safely retry requests."

## Question 3. What is the difference between at-most-once, at-least-once, and exactly-once delivery?

# Direct answer

These terms describe **message delivery guarantees** in distributed systems.

| Delivery Guarantee | Message Lost?                    | Duplicate Delivery? | Typical Use Cases                                            |
| ------------------ | -------------------------------- | ------------------- | ------------------------------------------------------------ |
| **At-most-once**   | Yes                              | No                  | Logging, metrics, non-critical notifications                 |
| **At-least-once**  | No (unless catastrophic failure) | Yes                 | Payment events, order processing, email jobs                 |
| **Exactly-once**   | No                               | No                  | Financial transactions, stream processing, inventory updates |

The key trade-off is between **reliability, complexity, and performance**. As guarantees become stronger, implementation becomes more complex and expensive.

---

# Understanding the Guarantees

## 1. At-most-once Delivery

A message is delivered **zero or one time**.

- It may be lost.
- It is never delivered twice.

### Flow

```text
Producer

↓

Broker

↓

Consumer
```

If the consumer crashes before processing or acknowledging:

```text
Producer

↓

Broker

↓

Consumer crashes

↓

Message Lost
```

The broker does **not** retry.

### Advantages

- Lowest latency
- Simplest implementation
- No duplicate handling

### Disadvantages

- Possible message loss
- Unsuitable for critical data

### Examples

- Debug logs
- Analytics events
- Monitoring metrics
- Presence updates

---

## 2. At-least-once Delivery

A message is delivered **one or more times**.

- No intentional message loss.
- Duplicates are possible.

### Flow

```text
Producer

↓

Broker

↓

Consumer

↓

ACK
```

Suppose:

```text
Consumer processes message

↓

ACK is lost

↓

Broker retries

↓

Consumer processes again
```

The message is processed twice.

### Advantages

- High reliability
- No message loss under normal conditions

### Disadvantages

- Duplicate processing
- Consumers must be idempotent

### Examples

- Payment events
- Order creation
- Inventory updates
- Email processing

---

## 3. Exactly-once Delivery

A message is processed **exactly one time**.

- No loss
- No duplicates

This is the strongest guarantee.

### Ideal Flow

```text
Producer

↓

Broker

↓

Consumer

↓

Processed Once

↓

ACK
```

Even if retries occur internally, the **effect** of processing happens only once.

---

# Why Exactly-Once Is Difficult

Consider:

```text
Consumer processes payment

↓

Payment succeeds

↓

ACK lost

↓

Broker retries
```

Without safeguards:

```text
Customer Charged Twice
```

The broker cannot determine whether:

- Processing failed, or
- Only the acknowledgment was lost.

This uncertainty is a classic distributed systems challenge.

---

# How Systems Achieve Exactly-Once Semantics

True network-level exactly-once delivery is generally **not achievable** in distributed systems due to failures and partitions.

Instead, systems aim for **exactly-once processing semantics** using techniques such as:

### Idempotency

```
Payment ID = P123

Retry

↓

Already Processed

↓

Return Previous Result
```

---

### Deduplication

Maintain processed message IDs.

```
Processed IDs

A101
A102
A103
```

Ignore repeated IDs.

---

### Transactions

Atomically:

- Consume message
- Update database
- Commit offset

Either all steps succeed or none do.

---

### Atomic Offset Commit

```
Read Message

↓

Update DB

↓

Commit Offset

↓

Done
```

This prevents reprocessing after successful commits.

---

# Comparison

| Property                  | At-most-once | At-least-once       | Exactly-once                             |
| ------------------------- | ------------ | ------------------- | ---------------------------------------- |
| Message loss              | Possible     | Rare                | No                                       |
| Duplicate delivery        | No           | Yes                 | No                                       |
| Retry                     | No           | Yes                 | Internal retries hidden from application |
| Consumer complexity       | Low          | Medium (idempotent) | High                                     |
| Performance               | Highest      | High                | Lowest                                   |
| Implementation complexity | Low          | Medium              | High                                     |

---

# Real-World Examples

| System                                                  | Delivery Guarantee                         |
| ------------------------------------------------------- | ------------------------------------------ |
| UDP                                                     | At-most-once                               |
| HTTP without retries                                    | At-most-once                               |
| RabbitMQ (manual ACK)                                   | At-least-once                              |
| Apache Kafka (default consumer behavior)                | At-least-once                              |
| Apache Kafka with idempotent producers and transactions | Exactly-once processing semantics          |
| Payment APIs using idempotency keys                     | Exactly-once effect for payment operations |

---

# Trade-offs

| Guarantee         | Pros                                     | Cons                                                          |
| ----------------- | ---------------------------------------- | ------------------------------------------------------------- |
| **At-most-once**  | Fastest, simplest                        | Risk of data loss                                             |
| **At-least-once** | Reliable delivery                        | Requires idempotent consumers to handle duplicates            |
| **Exactly-once**  | Prevents both loss and duplicate effects | Highest complexity, additional coordination, lower throughput |

---

# Interview-ready summary

> "At-most-once delivery means a message is delivered zero or one time, so messages can be lost but never duplicated. At-least-once delivery guarantees messages are eventually delivered, but duplicates can occur, so consumers must be idempotent. Exactly-once aims to ensure each message's effect occurs only once, typically through idempotency, deduplication, and transactional processing rather than relying on the network alone. In practice, most distributed systems use at-least-once delivery with idempotent consumers because it offers a good balance between reliability and complexity."

## Question 4. How do you design a real-time typing indicator system for a chat app?

# Direct answer

A **real-time typing indicator system** lets users know when another participant is actively typing without persisting this information. Unlike chat messages, typing events are **ephemeral**, **latency-sensitive**, and **best-effort**—it's acceptable to occasionally miss an event, but the indicator should appear and disappear quickly.

The typical design uses **WebSockets**, a **presence service**, and **short-lived typing events** with automatic expiration.

---

# Requirements / Problem Framing

### Functional Requirements

- Show "User X is typing..."
- Support one-to-one and group chats
- Automatically stop showing the indicator after inactivity
- Notify only participants in the conversation
- Do not permanently store typing events

### Non-functional Requirements

- Very low latency (<100 ms preferred)
- High scalability
- Minimal bandwidth usage
- High availability
- Best-effort delivery (occasional misses are acceptable)

---

# High-Level Architecture

```text
                +----------------------+
                |      Chat Client     |
                +----------+-----------+
                           |
                     WebSocket
                           |
                 +---------v---------+
                 | Realtime Gateway  |
                 +---------+---------+
                           |
                  Typing Event
                           |
                 +---------v---------+
                 | Presence Service  |
                 +---------+---------+
                           |
                 Publish Typing Event
                           |
                  Pub/Sub (Redis/Kafka)
                           |
         +-----------------+-----------------+
         |                                   |
 +-------v-------+                   +-------v-------+
 | Chat Server A |                   | Chat Server B |
 +-------+-------+                   +-------+-------+
         |                                   |
   Notify Receiver                     Notify Receiver
```

---

# Event Flow

## User Starts Typing

```
Alice types

↓

Client sends

typing_start(chatId, userId)

↓

Server

↓

Broadcast to chat participants

↓

Bob sees

"Alice is typing..."
```

---

## User Stops Typing

Either:

```
typing_stop
```

or

```
No keystrokes for 3–5 seconds

↓

Server expires indicator

↓

Remove "typing..."
```

Using inactivity timeouts avoids relying solely on explicit `typing_stop`, which may never arrive if the user closes the app or loses connectivity.

---

# API Design

### WebSocket Events

**Start Typing**

```json
{
  "type": "typing_start",
  "chatId": "c101",
  "userId": "u12"
}
```

**Stop Typing**

```json
{
  "type": "typing_stop",
  "chatId": "c101",
  "userId": "u12"
}
```

Broadcast:

```json
{
  "type": "user_typing",
  "chatId": "c101",
  "userId": "u12"
}
```

---

# Client Optimization

Sending an event for every keystroke would overwhelm the server.

Instead:

```
Key Press

↓

If not already typing

↓

Send typing_start

↓

Suppress further events

↓

Reset inactivity timer
```

Example:

```
Alice types:

H
He
Hel
Hell
Hello
```

Instead of five events:

```
typing_start

...

typing_stop
```

Only two events are sent.

---

# Server-Side State

Maintain temporary typing state.

```text
chatId

↓

typingUsers

↓

{
   Alice,
   Charlie
}
```

Each entry has a short expiration.

Example:

```
TTL = 5 seconds
```

No database persistence is needed.

---

# Scaling Strategy

## Multiple WebSocket Servers

```
           LB
        /      \
 WS-1          WS-2
```

Alice and Bob may connect to different servers.

Without coordination:

- Alice's server doesn't know where Bob is connected.

Solution:

```
Alice

↓

WS-1

↓

Redis Pub/Sub

↓

WS-2

↓

Bob
```

Each WebSocket server subscribes to typing events and forwards them to locally connected users.

---

## Presence Service

Track:

```
User

↓

Current Connection

↓

Current Chats

↓

Online Status
```

This allows routing typing events only to participants currently connected.

---

# Group Chats

If several users type simultaneously:

```
Alice typing

Bob typing

Charlie typing
```

Display:

```
Alice is typing...

Alice and Bob are typing...

3 people are typing...
```

The server maintains a set of active typists per chat and broadcasts updates.

---

# Deep Design Considerations

## Event Throttling

Without throttling:

```
10 keys/sec

×

1 million users

=

10 million events/sec
```

Instead:

```
Send at most once every 2–3 seconds
```

This dramatically reduces network traffic while keeping the UI responsive.

---

## Automatic Expiration

Suppose:

```
typing_start

↓

Phone battery dies
```

No `typing_stop` arrives.

Solution:

```
Every typing event

↓

TTL = 5 seconds

↓

Auto-remove
```

This prevents stale indicators.

---

## Best-Effort Delivery

Typing indicators are **non-critical**.

If one event is lost:

```
User briefly doesn't see typing
```

This is acceptable, unlike losing a chat message.

Therefore:

- No retries
- No persistence
- No durable queues

---

## Ordering

Possible sequence:

```
typing_start

↓

typing_stop

↓

typing_start
```

Use timestamps or sequence numbers to ignore stale events that arrive out of order.

---

# Capacity Considerations

Assume:

- 10 million daily active users
- 1 million concurrent users
- Average typing session: 10 seconds
- One `typing_start` and one `typing_stop` per session

If each active user starts typing once per minute:

```
1M / 60

≈ 16,700 typing starts/sec

≈ 16,700 typing stops/sec

≈ 33,400 events/sec
```

With throttling and lightweight payloads (tens of bytes), this is well within the capabilities of horizontally scaled WebSocket gateways and an in-memory Pub/Sub system like Redis.

---

# Security / Observability

### Security

- Authenticate WebSocket connections (JWT or session token).
- Verify the sender belongs to the chat before broadcasting.
- Apply rate limits to prevent event flooding.
- Ignore malformed or spoofed typing events.

### Observability

Monitor:

- Active WebSocket connections
- Typing events/sec
- Broadcast latency
- Pub/Sub latency
- Event drop rate
- Average typing session duration
- WebSocket disconnects and reconnects

---

# Trade-offs

| Approach              | Advantages             | Disadvantages                                                                  |
| --------------------- | ---------------------- | ------------------------------------------------------------------------------ |
| Polling               | Simple                 | High latency and inefficient                                                   |
| WebSockets            | Real-time, low latency | Persistent connection management                                               |
| Durable Queue (Kafka) | Reliable               | Overkill for ephemeral events                                                  |
| Redis Pub/Sub         | Fast, lightweight      | Messages are lost if subscribers are temporarily unavailable (acceptable here) |
| TTL-based expiration  | Simple and resilient   | Indicator may linger briefly after typing stops                                |

---

# Interview-ready summary

> "A typing indicator is an ephemeral, best-effort feature, so I'd implement it using WebSockets. When a user begins typing, the client sends a throttled `typing_start` event, and the server broadcasts it only to participants in that chat. The server stores typing state in memory with a short TTL, typically 3–5 seconds, so indicators automatically disappear even if a `typing_stop` event is never received. In a multi-server deployment, WebSocket servers synchronize through Redis Pub/Sub. Since typing events are transient, I wouldn't persist them or use durable messaging, keeping the design low-latency, scalable, and bandwidth-efficient."

## Question 5. What is a vector clock and how is it used in conflict resolution?

## Question 6. How do you design a low-latency leaderboard system?

## Question 7. Explain how DNS load balancing works under the hood

## Question 8. How would you store billions of IoT device telemetry logs efficiently?

## Question 9. What is hot partitioning in distributed systems, and how do you solve it?

## Question 10. How do you implement distributed tracing in microservices?

## Question 11. What are quorum reads and quorum writes in distributed databases?

## Question 12. How would you design a feature flagging system?

## Question 13. What is a tombstone in Cassandra, and why is it important?

## Question 14. How do you design a system to support multiple time zones efficiently?

## Question 15. How would you implement pagination for large datasets?

## Question 16. What is the difference between logical and physical data models?

## Question 17. How would you design a video transcoding pipeline at scale?

## Question 18. How do you ensure GDPR compliance in data storage design?

## Question 19. What is a CRDT, and how does it solve real-time sync problems?

## Question 20. How would you design an alerting system for server downtime?
