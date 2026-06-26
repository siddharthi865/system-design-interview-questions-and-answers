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

## Question 3. What is the difference between at-most-once, at-least-once, and exactly-once delivery?

## Question 4. How do you design a real-time typing indicator system for a chat app?

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
