# Set 18

| S.No. | Question                                                                                                                                                                           |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you design a system with API rate quotas?](#question-1-how-do-you-design-a-system-with-api-rate-quotas)                                                                    |
| 2.    | [How do you design a reliable multicast system?](#question-2-how-do-you-design-a-reliable-multicast-system)                                                                        |
| 3.    | [How do you design an email spam filtering system?](#question-3-how-do-you-design-an-email-spam-filtering-system)                                                                  |
| 4.    | [What is a relay server?](#question-4-what-is-a-relay-server)                                                                                                                      |
| 5.    | [How do you design a web crawler like Googlebot?](#question-5-how-do-you-design-a-web-crawler-like-googlebot)                                                                      |
| 6.    | [How do you design a system with DNS load balancing?](#question-6-how-do-you-design-a-system-with-dns-load-balancing)                                                              |
| 7.    | [What is a TCP vs UDP tradeoff in system design?](#question-7-what-is-a-tcp-vs-udp-tradeoff-in-system-design)                                                                      |
| 8.    | [How do you design a peer-to-peer file sharing system?](#question-8-how-do-you-design-a-peer-to-peer-file-sharing-system)                                                          |
| 9.    | [How do you design a push-to-talk (PTT) system?](#question-9-how-do-you-design-a-push-to-talk-ptt-system)                                                                          |
| 10.   | [How do you design a video conferencing platform like Zoom?](#question-10-how-do-you-design-a-video-conferencing-platform-like-zoom)                                               |
| 11.   | [What is backpressure in system design?](#question-11-what-is-backpressure-in-system-design)                                                                                       |
| 12.   | [What are hotspots in load balancing?](#question-12-what-are-hotspots-in-load-balancing)                                                                                           |
| 13.   | [How do you design a system to handle batch jobs at scale?](#question-13-how-do-you-design-a-system-to-handle-batch-jobs-at-scale)                                                 |
| 14.   | [How do you design a high-QPS (queries per second) read system?](#question-14-how-do-you-design-a-high-qps-queries-per-second-read-system)                                         |
| 15.   | [How do you design a low-latency trading system?](#question-15-how-do-you-design-a-low-latency-trading-system)                                                                     |
| 16.   | [What is speculative execution in distributed systems?](#question-16-what-is-speculative-execution-in-distributed-systems)                                                         |
| 17.   | [How do you optimize for P99 latency?](#question-17-how-do-you-optimize-for-p99-latency)                                                                                           |
| 18.   | [How do you design a multi-region database?](#question-18-how-do-you-design-a-multi-region-database)                                                                               |
| 19.   | [How do you scale search indexing systems?](#question-19-how-do-you-scale-search-indexing-systems)                                                                                 |
| 20.   | [What is the difference between vertical partitioning and horizontal partitioning?](#question-20-what-is-the-difference-between-vertical-partitioning-and-horizontal-partitioning) |

## Question 1. How do you design a system with API rate quotas?

## Direct answer

A system with API rate quotas is designed using a **centralized or distributed rate-limiting layer** that tracks each client’s usage (requests per time window) and enforces limits before requests hit backend services. The core idea is: **identify user → track usage → decide allow/deny → enforce consistently across all instances**.

---

## Requirements / problem framing

### Functional requirements

- Enforce API usage limits per:
  - User / API key / tenant
  - Endpoint (optional)
  - Time window (e.g., 1000 requests/minute)

- Reject or throttle requests exceeding quota
- Support different quota plans (free, pro, enterprise)
- Return meaningful error responses (e.g., `429 Too Many Requests`)

### Non-functional requirements

- Low latency overhead (rate check must be fast)
- Highly available (cannot block API if rate limiter fails)
- Horizontally scalable
- Consistent enforcement across distributed servers
- Configurable and dynamic quota updates

---

## High-level architecture

A typical scalable design uses a **Rate Limiting Service backed by a fast in-memory store (Redis)**.

```
Client
  |
  v
API Gateway / Edge Proxy
  |
  v
Rate Limiter Service (middleware)
  |
  v
Redis / In-memory store (quota counters)
  |
  v
Backend Microservices
```

### Key flow

1. Request arrives at API Gateway
2. Gateway extracts identity (API key / user ID)
3. Rate limiter checks Redis for usage
4. If allowed → forward request
5. If exceeded → return 429 immediately

---

## Core rate-limiting strategies

### 1. Fixed Window Counter

- Count requests in fixed intervals (e.g., per minute)

**Example**

- User limit: 100 req/min
- Key: `user123:20260629T10:00`

**Pros**

- Simple
- Low storage overhead

**Cons**

- Burst problem at window edges

---

### 2. Sliding Window Log

- Store timestamps of requests
- Count only last X seconds

**Pros**

- Accurate

**Cons**

- High memory usage

---

### 3. Sliding Window Counter (hybrid)

- Approximate sliding behavior using two buckets

**Pros**

- Balanced accuracy + performance

**Cons**

- Slight approximation error

---

### 4. Token Bucket (most common in real systems)

- Each user has a “bucket” of tokens
- Each request consumes a token
- Tokens refill at a fixed rate

**Pros**

- Handles bursts gracefully
- Industry standard (AWS, NGINX-style)

**Cons**

- Slightly more complex

---

## Deep design considerations

### 1. Distributed consistency problem

In multi-instance systems, each API server must agree on usage.

**Solution: Redis (central counter store)**

- Atomic operations using `INCR`, `EXPIRE`
- Lua scripts for atomic check + update

---

### 2. High throughput optimization

Redis can become bottleneck → solutions:

- Local caching (soft limits)
- Sharded Redis (by user hash)
- Edge rate limiting (CDN / API Gateway level)

---

### 3. Multi-tier quotas

Real systems often have:

- Global quota (per API key)
- Endpoint-specific quota (e.g., `/search` stricter)
- Burst vs sustained limits

---

### 4. Retry behavior

When limit is exceeded:

- Return `429 Too Many Requests`
- Include headers:
  - `Retry-After`
  - `X-RateLimit-Remaining`
  - `X-RateLimit-Reset`

---

### 5. Fault tolerance

If Redis fails:

- Fail open (allow traffic) OR
- Fail closed (block traffic) depending on risk tolerance

Most production APIs:

- Critical systems → fail closed
- Consumer APIs → fail open with degraded enforcement

---

### 6. Multi-region systems

Options:

- Global centralized Redis (simpler, higher latency)
- Regional rate limiters + eventual sync
- Edge enforcement at CDN layer (best for latency)

---

## Trade-offs

| Approach           | Pros                     | Cons             | When to use          |
| ------------------ | ------------------------ | ---------------- | -------------------- |
| Fixed window       | Simple, fast             | Burst issues     | Basic APIs           |
| Sliding log        | Accurate                 | Expensive        | High precision needs |
| Token bucket       | Flexible, burst-friendly | Complex          | Production systems   |
| Edge-only limiting | Very fast                | Hard consistency | CDN/API gateways     |

---

## Capacity / sizing (example)

Assume:

- 1M users
- 100 req/sec peak per user group

Redis load:

- ~100K–500K ops/sec

Scaling strategies:

- Redis cluster (sharding by user_id hash)
- Local caching for short bursts
- Batch token refill instead of per-request writes

---

## Security / observability

### Security

- Rate limit by API key (not IP alone)
- Prevent key sharing abuse
- Detect anomalies (spikes, bot traffic)

### Observability

- Metrics:
  - Allowed requests
  - Rejected requests (429s)
  - Per-user usage distribution

- Alerts:
  - sudden spikes in 429s
  - Redis latency increase

- Tracing:
  - track request → rate limiter → backend path

---

## Interview-ready summary

A scalable API rate quota system is typically implemented at the API gateway level using a distributed rate limiter backed by a fast store like Redis. The system tracks per-user request usage using algorithms like token bucket or sliding window counters. It enforces quotas consistently across multiple service instances while balancing accuracy, performance, and availability. Trade-offs revolve around precision vs cost, and centralized vs edge-based enforcement.

## Question 2. How do you design a reliable multicast system?

## Direct answer

A reliable multicast system ensures that a message sent by one producer is delivered to **all intended receivers correctly, exactly once (or at least once depending on semantics), in a consistent order, despite failures, network loss, or partitions**. The core challenge is combining **fan-out scalability (multicast)** with **reliability guarantees (ACK/NAK, retransmission, ordering, and fault tolerance)**.

The most common real-world approach is:

> **Use a brokered or tree-based multicast + sequence numbering + receiver-driven or sender-driven retransmission + membership management.**

---

## Requirements / problem framing

### Functional requirements

- One-to-many message delivery (publisher → many subscribers)
- Guaranteed delivery semantics:
  - At-least-once (common)
  - Exactly-once (hard, usually application-level dedupe)

- Optional ordering:
  - FIFO (per sender)
  - Total order (stronger, more expensive)

- Dynamic group membership (join/leave)
- Retransmission of lost messages

### Non-functional requirements

- High scalability (thousands/millions of receivers)
- Low latency fan-out
- Fault tolerance (node + network failure)
- Backpressure handling (slow consumers)
- Efficient bandwidth usage (avoid N copies per receiver)

---

## High-level architecture

There are three common architectural patterns:

### 1. Broker-based multicast (most practical)

```text
        Publisher
            |
            v
     +----------------+
     |  Message Broker|
     | (Kafka/NATS)   |
     +----------------+
       /     |      \
      v      v       v
  Consumer Consumer Consumer
```

- Broker handles replication and delivery guarantees
- Consumers pull messages (simplifies reliability)
- Used in Kafka-like systems

---

### 2. Tree-based multicast (network efficient)

```text
            Root
           /    \
        Node     Node
       /   \       \
   Leaf   Leaf     Leaf
```

- Messages forwarded down a spanning tree
- Reduces sender load from O(N) → O(log N)

---

### 3. Overlay multicast (hybrid peer-assisted)

- Nodes help forward messages (P2P-style)
- Used in WebRTC / live streaming systems

---

## Core design components

### 1. Message identification

Each message must include:

- `groupId`
- `sequenceNumber`
- `timestamp`
- `producerId`

This enables:

- Deduplication
- Ordering
- Gap detection

---

### 2. Group membership service

We need to know:

- Who is in the multicast group?
- How to update membership safely?

Options:

- Central registry (simple, but bottleneck)
- Distributed coordination (ZooKeeper / etcd-style)

---

### 3. Reliable delivery mechanism

#### Option A: ACK-based (sender-driven reliability)

- Sender tracks all receivers
- Receivers ACK received messages
- Sender retransmits missing messages

**Pros**

- Strong guarantees

**Cons**

- Doesn’t scale well (O(N) ACK overhead)

---

#### Option B: NAK-based (receiver-driven reliability) ⭐ preferred

- Sender does NOT track every receiver
- Receiver detects missing sequence numbers
- Receiver requests retransmission

```text
Sender → multicast message(seq=10)
Receiver A gets 10
Receiver B misses 10 → sends NAK(10)
Sender retransmits 10
```

**Pros**

- Much more scalable
- Lower overhead

---

### 4. Ordering guarantees

#### FIFO ordering

- Sequence numbers per sender
- Receiver buffers out-of-order messages

#### Total ordering (harder)

Options:

- Central sequencer
- Distributed consensus (Raft-like ordering service)

Used in:

- financial systems
- replicated state machines

---

### 5. Message buffering & retransmission store

Sender or broker must retain:

- last N messages (sliding window buffer)
- configurable retention time

Used for:

- replay on NAK
- slow consumers
- recovery after failure

---

### 6. Flow control / backpressure

Without control:

- slow consumer causes memory blowup

Solutions:

- consumer lag tracking
- drop policies (for non-critical streams)
- rate limiting per subscriber
- window-based flow control

---

## Deep design considerations

### 1. Scalability bottleneck problem

If sender directly sends to all receivers:

- O(N) fanout cost
- network explosion

Fix:

- broker fanout OR tree-based forwarding

---

### 2. Failure handling

| Failure type   | Handling                              |
| -------------- | ------------------------------------- |
| Receiver crash | rejoin + replay from last offset      |
| Sender crash   | leader election + state recovery      |
| Network loss   | NAK + retransmission                  |
| Partition      | temporary divergence + reconciliation |

---

### 3. Exactly-once semantics (hard problem)

Typical approach:

- at-least-once delivery
- receiver-side dedup using message IDs

True exactly-once requires:

- transactional log + idempotent processing

---

### 4. Large-scale multicast optimization

Techniques:

- batching messages
- compression
- delta encoding (for state sync)
- hierarchical fanout (region → edge nodes → clients)

---

### 5. Hybrid real-world design (what companies use)

Most production systems combine:

- Kafka-like log for durability
- Pub/sub broker for fanout
- consumer offsets for reliability
- optional push layer for low latency

---

## Trade-offs

| Design         | Pros               | Cons                     | Use case                |
| -------------- | ------------------ | ------------------------ | ----------------------- |
| ACK-based      | Strong reliability | Not scalable             | small groups            |
| NAK-based      | scalable           | delayed detection        | large multicast         |
| Broker-based   | simple, robust     | latency overhead         | most production systems |
| Tree-based     | low latency        | complex failure recovery | streaming systems       |
| Total ordering | strict consistency | slow                     | financial systems       |

---

## Interview-ready summary

A reliable multicast system ensures correct and ordered delivery of messages to all group members despite failures. The key design challenge is scalable reliability. In practice, systems use a broker-based or tree-based architecture with sequence numbering, sliding-window buffering, and either ACK or NAK-based retransmission. Most scalable designs prefer NAK-based recovery with message retention, combined with deduplication and consumer-side ordering. Trade-offs revolve around scalability vs strict ordering guarantees vs complexity.

## Question 3. How do you design an email spam filtering system?

## Question 4. What is a relay server?

## Question 5. How do you design a web crawler like Googlebot?

## Question 6. How do you design a system with DNS load balancing?

## Question 7. What is a TCP vs UDP tradeoff in system design?

## Question 8. How do you design a peer-to-peer file sharing system?

## Question 9. How do you design a push-to-talk (PTT) system?

## Question 10. How do you design a video conferencing platform like Zoom?

## Question 11. What is backpressure in system design?

## Question 12. What are hotspots in load balancing?

## Question 13. How do you design a system to handle batch jobs at scale?

## Question 14. How do you design a high-QPS (queries per second) read system?

## Question 15. How do you design a low-latency trading system?

## Question 16. What is speculative execution in distributed systems?

## Question 17. How do you optimize for P99 latency?

## Question 18. How do you design a multi-region database?

## Question 19. How do you scale search indexing systems?

## Question 20. What is the difference between vertical partitioning and horizontal partitioning?
