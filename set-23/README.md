# Set 23

| S.No. | Question                                                                                                                                                          |
| ----- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [Explain the difference between CQRS and Event Sourcing](#question-1-explain-the-difference-between-cqrs-and-event-sourcing)                                      |
| 2.    | [How do you prevent cache avalanche, cache stampede, and cache penetration?](#question-2-how-do-you-prevent-cache-avalanche-cache-stampede-and-cache-penetration) |
| 3.    | [How would you design a financial transaction ledger system?](#question-3-how-would-you-design-a-financial-transaction-ledger-system)                             |
| 4.    | [What are the trade-offs of push vs. pull models in notifications?](#question-4-what-are-the-trade-offs-of-push-vs-pull-models-in-notifications)                  |
| 5.    | [How would you design a rate limiter (like token bucket or leaky bucket)?](#question-5-how-would-you-design-a-rate-limiter-like-token-bucket-or-leaky-bucket)     |
| 6.    | [What is a sticky session, and when should you avoid it?](#question-6-what-is-a-sticky-session-and-when-should-you-avoid-it)                                      |
| 7.    | [How do you design multi-region support for an e-commerce system?](#question-7-how-do-you-design-multi-region-support-for-an-e-commerce-system)                   |
| 8.    | [What is a Bloom filter, and how would you use it in caching?](#question-8-what-is-a-bloom-filter-and-how-would-you-use-it-in-caching)                            |
| 9.    | [How do you handle schema evolution in NoSQL databases?](#question-9-how-do-you-handle-schema-evolution-in-nosql-databases)                                       |
| 10.   | [How would you implement two-factor authentication in system design?](#question-10-how-would-you-implement-two-factor-authentication-in-system-design)            |
| 11.   | [How do you secure inter-service communication in microservices?](#question-11-how-do-you-secure-inter-service-communication-in-microservices)                    |
| 12.   | [How would you implement an "undo" feature in collaborative systems?](#question-12-how-would-you-implement-an-undo-feature-in-collaborative-systems)              |
| 13.   | [How do you design a system that provides real-time analytics dashboards?](#question-13-how-do-you-design-a-system-that-provides-real-time-analytics-dashboards)  |
| 14.   | [What is Raft consensus, and how does it differ from Paxos?](#question-14-what-is-raft-consensus-and-how-does-it-differ-from-paxos)                               |
| 15.   | [How do you handle versioning in REST APIs vs. GraphQL APIs?](#question-15-how-do-you-handle-versioning-in-rest-apis-vs-graphql-apis)                             |
| 16.   | [How would you design a distributed cron job scheduler?](#question-16-how-would-you-design-a-distributed-cron-job-scheduler)                                      |
| 17.   | [What's the role of gossip protocols in distributed systems?](#question-17-whats-the-role-of-gossip-protocols-in-distributed-systems)                             |
| 18.   | [How do you optimize cold-start latency in serverless systems?](#question-18-how-do-you-optimize-cold-start-latency-in-serverless-systems)                        |
| 19.   | [How would you design a resilient order management system?](#question-19-how-would-you-design-a-resilient-order-management-system)                                |
| 20.   | [What is Canary Deployment, and how does it help in system design?](#question-20-what-is-canary-deployment-and-how-does-it-help-in-system-design)                 |

## Question 1. Explain the difference between CQRS and Event Sourcing

# CQRS vs Event Sourcing

## Direct answer

**CQRS (Command Query Responsibility Segregation)** and **Event Sourcing** are two distinct architectural patterns that are often used together but solve different problems.

- **CQRS** separates the **write (commands)** and **read (queries)** models to optimize each independently.
- **Event Sourcing** stores every change to an application's state as a sequence of **immutable events** instead of storing only the latest state.

**Key point:** You can use **CQRS without Event Sourcing**, **Event Sourcing without CQRS**, or **both together**.

---

## Core difference

| CQRS                                     | Event Sourcing                             |
| ---------------------------------------- | ------------------------------------------ |
| Separates read and write models          | Stores state as a sequence of events       |
| Focuses on application architecture      | Focuses on data persistence                |
| Optimizes reads and writes independently | Provides complete history of state changes |
| Current state is usually stored directly | Current state is reconstructed from events |
| Doesn't require event storage            | Requires event storage                     |
| Can use traditional databases            | Uses an event store                        |

---

# CQRS Explained

In traditional CRUD systems:

```
          Client
             |
        User Service
             |
         User Table
```

The same model handles both:

- Creating users
- Updating users
- Fetching users
- Searching users

CQRS separates these responsibilities.

```
            Client
          /        \
    Commands      Queries
        |             |
 Write Service    Read Service
        |             |
 Write DB       Read Database
```

### Command Side

Responsible for:

- Create
- Update
- Delete
- Validation
- Business rules

Example:

```
POST /orders
PUT /orders/123
DELETE /orders/123
```

---

### Query Side

Responsible only for:

- Reading
- Searching
- Aggregation
- Reporting

Example:

```
GET /orders
GET /dashboard
GET /analytics
```

The read database may be optimized differently:

- Denormalized
- Indexed for search
- Cached
- Materialized views

---

## Benefits of CQRS

- Independent scaling
- Faster reads
- Simpler queries
- Better separation of concerns
- Different storage technologies for reads/writes

Example:

```
Writes -> PostgreSQL

Reads -> Elasticsearch
```

---

## Drawbacks

- More services
- Eventual consistency
- Synchronization complexity
- More infrastructure

---

# Event Sourcing Explained

Instead of storing:

```
Account
-------
Balance = 150
```

Store every event:

```
AccountCreated
MoneyDeposited(100)
MoneyDeposited(80)
MoneyWithdrawn(30)
```

Current balance is calculated by replaying:

```
0
+100
+80
-30
-----
150
```

The events become the source of truth.

---

## Traditional Database

```
Account

ID    Balance

1      150
```

Only current value exists.

Past history is lost.

---

## Event Store

```
Event 1:
AccountCreated

Event 2:
Deposit 100

Event 3:
Deposit 80

Event 4:
Withdraw 30
```

Nothing is deleted.

Nothing is updated.

Events are append-only.

---

## Rebuilding State

```
Events

↓

Replay

↓

Current State
```

If projection data is lost:

```
Replay all events

↓

Recreate database
```

---

# Why Event Sourcing?

### Complete audit trail

Know exactly:

- Who changed what
- When
- Why

Perfect for:

- Banking
- Healthcare
- Financial systems

---

### Time travel

View system state at:

```
Yesterday

Last week

Last month
```

Simply replay events up to that point.

---

### Debugging

Instead of asking:

> "What happened?"

You replay the exact sequence.

---

### Recovery

If read databases fail:

```
Replay events

↓

Rebuild projections
```

---

# CQRS + Event Sourcing Together

These patterns complement each other well.

```
          Commands
              |
        Command Handler
              |
        Business Logic
              |
         Store Events
              |
        Event Store
              |
       Publish Events
        /           \
 Projection      Analytics
 Builder          Service
      |                |
 Read Database     Other Systems
```

Flow:

1. User sends command
2. Validate command
3. Generate event
4. Save event
5. Publish event
6. Update read model
7. Queries use read model

---

## Example: Order System

Traditional CRUD

```
Order

Status = Delivered
```

Previous states are lost.

---

With Event Sourcing

```
OrderCreated

↓

PaymentReceived

↓

OrderPacked

↓

OrderShipped

↓

OrderDelivered
```

Current status:

```
Replay events

↓

Delivered
```

Read model:

```
Orders Table

ID
Customer
Status
Delivery Date
```

Users query this table instead of replaying events on every request.

---

# When to use CQRS

Use CQRS when:

- Read traffic greatly exceeds write traffic
- Read and write models differ significantly
- Complex reporting is required
- Independent scaling of reads and writes is beneficial

Avoid CQRS for:

- Small CRUD applications
- Simple internal tools
- Low-traffic systems

---

# When to use Event Sourcing

Use Event Sourcing when:

- Complete audit history is required
- Financial transactions must be traceable
- Regulatory compliance is important
- You need to reconstruct or replay system state
- Domain events are valuable to other services

Avoid Event Sourcing when:

- History is unimportant
- Data changes are simple
- Teams are unfamiliar with event-driven systems
- Operational simplicity is a priority

---

# Can they be used independently?

### CQRS without Event Sourcing ✅

```
Commands

↓

Write Database

↓

Replicate

↓

Read Database
```

This is common in many production systems.

---

### Event Sourcing without CQRS ✅

```
Commands

↓

Event Store

↓

Replay Events

↓

Current State
```

Reads can replay events or use simple projections without maintaining separate read/write models.

---

### CQRS + Event Sourcing ✅

Most common in event-driven architectures.

```
Commands

↓

Event Store

↓

Events

↓

Read Projections

↓

Queries
```

---

# Trade-offs

| Aspect           | CQRS                                         | Event Sourcing                                    |
| ---------------- | -------------------------------------------- | ------------------------------------------------- |
| Primary goal     | Optimize reads and writes                    | Preserve complete history                         |
| Data model       | Separate read/write models                   | Immutable event log                               |
| Complexity       | Medium                                       | High                                              |
| Audit history    | Limited unless implemented separately        | Built in                                          |
| Read performance | Excellent with optimized projections         | Requires projections or replay                    |
| Recovery         | Restore from backups                         | Replay events to rebuild state                    |
| Scalability      | Independent scaling of read/write paths      | Scales well with append-only writes               |
| Consistency      | Often eventual between read and write models | Often paired with eventual-consistent projections |

# Interview-ready summary

> **CQRS and Event Sourcing solve different problems. CQRS separates the command and query paths so each can be optimized independently, while Event Sourcing persists every state change as an immutable event rather than only the latest state. CQRS is an architectural pattern, whereas Event Sourcing is a persistence pattern. They are frequently combined: commands generate events stored in an event store, and those events are used to build read-optimized projections. This provides scalable reads, a complete audit trail, and the ability to rebuild application state by replaying events, at the cost of increased architectural and operational complexity.**

## Question 2. How do you prevent cache avalanche, cache stampede, and cache penetration?

# Direct answer

**Cache avalanche**, **cache stampede**, and **cache penetration** are three different cache failure scenarios:

- **Cache avalanche:** Many cache entries expire or become unavailable at the same time, causing a flood of requests to the database.
- **Cache stampede (dogpile effect):** Multiple requests simultaneously regenerate the same expired cache entry.
- **Cache penetration:** Requests repeatedly ask for data that doesn't exist, bypassing the cache and hitting the database every time.

Each problem requires different mitigation strategies.

---

# 1. Cache Avalanche

## What is it?

A cache avalanche occurs when **a large number of cache entries become invalid simultaneously**, or the entire cache cluster becomes unavailable.

Example:

```
1 million keys
TTL = 1 hour

↓

Exactly after 1 hour

↓

All keys expire

↓

Millions of DB requests

↓

Database overload
```

## Causes

- Same TTL for all keys
- Redis cluster failure
- Cache restart
- Bulk cache invalidation

---

## Prevention Techniques

### 1. Randomized TTL (Most Common)

Instead of:

```
TTL = 1 hour
```

Use:

```
TTL = 1 hour ± random(0–10 minutes)
```

Example:

```
User1 → 61 min
User2 → 58 min
User3 → 66 min
```

Keys expire gradually instead of all at once.

---

### 2. High Availability Cache

Deploy Redis with replication and failover.

```
           Client
              |
        Load Balancer
              |
     -------------------
     |                 |
 Redis Primary    Redis Replica
```

This reduces the chance of a complete cache outage.

---

### 3. Warm the Cache

Preload frequently accessed data after:

- Restart
- Deployment
- Disaster recovery

Instead of waiting for user traffic to refill the cache.

---

### 4. Multi-Level Cache

```
Application

↓

Local Cache (L1)

↓

Redis (L2)

↓

Database
```

If Redis is unavailable, the application may still serve many requests from its local cache.

---

### 5. Rate Limiting

When cache misses spike:

```
Too many DB requests

↓

Throttle

↓

Protect database
```

---

# 2. Cache Stampede (Dogpile Effect)

## What is it?

Only **one hot key** expires, but thousands of clients request it simultaneously.

```
Cache:

Product 100 → expired

↓

10,000 requests

↓

10,000 DB queries
```

Instead of:

```
1 DB query
```

---

## Prevention Techniques

### 1. Distributed Lock (Most Common)

Only one request regenerates the cache.

```
Request A

↓

Acquire Lock

↓

DB Query

↓

Update Cache

↓

Release Lock
```

Other requests:

```
Wait

↓

Read newly populated cache
```

---

### 2. Single Flight / Request Coalescing

Merge identical concurrent requests.

```
100 requests

↓

One DB fetch

↓

Shared response

↓

100 clients
```

Popular libraries provide this pattern.

---

### 3. Stale-While-Revalidate

Serve slightly stale data while refreshing in the background.

```
Expired

↓

Serve old value

↓

Background refresh

↓

Replace cache
```

Users get fast responses and the database avoids spikes.

---

### 4. Proactive Refresh

Refresh hot keys before expiration.

```
TTL remaining

↓

Background worker refreshes

↓

Key never expires during peak traffic
```

Ideal for dashboards, trending content, and popular products.

---

### 5. Never Expire Hot Keys

For very hot data:

```
No TTL

+

Explicit invalidation
```

Useful when updates are event-driven.

---

# 3. Cache Penetration

## What is it?

Clients repeatedly request **data that doesn't exist**.

Example:

```
GET /user/999999999

↓

Cache miss

↓

DB miss

↓

Return 404
```

Next request repeats the same expensive path.

---

## Prevention Techniques

### 1. Cache Null Results (Most Common)

Store:

```
User 999999999

↓

NULL

TTL = 1 minute
```

Next request:

```
Cache hit

↓

NULL

↓

Return 404

(No DB call)
```

Use a short TTL to handle newly created records.

---

### 2. Bloom Filter

Maintain a probabilistic set of valid IDs.

```
Request

↓

Bloom Filter

↓

Probably Exists?

No

↓

Reject immediately
```

Only likely-valid keys reach Redis or the database.

**Trade-off:**

- No false negatives
- Possible false positives

---

### 3. Input Validation

Reject obviously invalid requests.

Examples:

```
Negative IDs

Invalid UUIDs

Malformed keys

Unsupported formats
```

These never reach the database.

---

### 4. Rate Limiting

Limit abusive clients making repeated invalid requests.

```
1000 invalid requests/sec

↓

Throttle

↓

Protect backend
```

---

# Comparison

| Problem           | Cause                                     | Impact                             | Common Solution                                              |
| ----------------- | ----------------------------------------- | ---------------------------------- | ------------------------------------------------------------ |
| Cache Avalanche   | Many keys expire together or cache outage | Massive DB traffic spike           | Random TTL, HA cache, cache warming                          |
| Cache Stampede    | One hot key expires under heavy load      | Duplicate DB work for the same key | Distributed lock, request coalescing, stale-while-revalidate |
| Cache Penetration | Requests for non-existent data            | Continuous DB misses               | Cache null values, Bloom filter, input validation            |

---

# Combined Architecture

```
                Client Requests
                      |
                API Gateway
                      |
              Input Validation
                      |
              Bloom Filter Check
                      |
                  Cache (Redis)
                 /             \
            Cache Hit      Cache Miss
                               |
                    Distributed Lock
                               |
                         Database Query
                               |
          +--------------------+------------------+
          |                                       |
   Cache Valid Result                    Cache NULL (short TTL)
          |
   Randomized TTL
          |
 Background Refresh for Hot Keys
```

---

# Trade-offs

| Technique              | Pros                                         | Cons                                            |
| ---------------------- | -------------------------------------------- | ----------------------------------------------- |
| Randomized TTL         | Prevents synchronized expirations            | Doesn't help individual hot keys                |
| Distributed Lock       | Eliminates duplicate regeneration            | Lock management adds latency and complexity     |
| Stale-While-Revalidate | Low latency, good user experience            | Clients may see slightly stale data             |
| Cache NULL Values      | Stops repeated DB misses                     | Short TTL required if data may be created later |
| Bloom Filter           | Blocks most invalid requests before cache/DB | False positives and additional maintenance      |
| Multi-Level Cache      | Reduces dependence on centralized cache      | Cache invalidation becomes more complex         |

---

# Interview-ready summary

> **These three cache issues have different root causes and solutions. Cache avalanche occurs when many keys expire simultaneously or the cache becomes unavailable, and is mitigated with randomized TTLs, high availability, and cache warming. Cache stampede happens when many requests regenerate the same hot key, which is addressed using distributed locks, request coalescing, stale-while-revalidate, or proactive refresh. Cache penetration involves repeated requests for non-existent data and is prevented by caching null results, using Bloom filters to reject invalid keys, validating inputs, and rate limiting abusive clients. Together, these techniques protect both the cache and the database under high load.**

## Question 3. How would you design a financial transaction ledger system?

## Question 4. What are the trade-offs of push vs. pull models in notifications?

## Question 5. How would you design a rate limiter (like token bucket or leaky bucket)?

## Question 6. What is a sticky session, and when should you avoid it?

## Question 7. How do you design multi-region support for an e-commerce system?

## Question 8. What is a Bloom filter, and how would you use it in caching?

## Question 9. How do you handle schema evolution in NoSQL databases?

## Question 10. How would you implement two-factor authentication in system design?

## Question 11. How do you secure inter-service communication in microservices?

## Question 12. How would you implement an "undo" feature in collaborative systems?

## Question 13. How do you design a system that provides real-time analytics dashboards?

## Question 14. What is Raft consensus, and how does it differ from Paxos?

## Question 15. How do you handle versioning in REST APIs vs. GraphQL APIs?

## Question 16. How would you design a distributed cron job scheduler?

## Question 17. What's the role of gossip protocols in distributed systems?

## Question 18. How do you optimize cold-start latency in serverless systems?

## Question 19. How would you design a resilient order management system?

## Question 20. What is Canary Deployment, and how does it help in system design?
