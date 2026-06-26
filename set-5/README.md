# Set 5

| S.No. | Question                                                                                                                                                     |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1.    | [How do you design a system to handle millions of users?](#question-1-how-do-you-design-a-system-to-handle-millions-of-users)                                |
| 2.    | [What are the challenges in designing distributed systems?](#question-2-what-are-the-challenges-in-designing-distributed-systems)                            |
| 3.    | [What is fault tolerance in system design?](#question-3-what-is-fault-tolerance-in-system-design)                                                            |
| 4.    | [What are leader election algorithms (like Raft, Paxos)?](#question-4-what-are-leader-election-algorithms-like-raft-paxos)                                   |
| 5.    | [What is consensus in distributed systems?](#question-5-what-is-consensus-in-distributed-systems)                                                            |
| 6.    | [How do you design a highly available system?](#question-6-how-do-you-design-a-highly-available-system)                                                      |
| 7.    | [What are design considerations for disaster recovery?](#question-7-what-are-design-considerations-for-disaster-recovery)                                    |
| 8.    | [What is circuit breaker pattern in microservices?](#question-8-what-is-circuit-breaker-pattern-in-microservices)                                            |
| 9.    | [What is idempotency in system design?](#question-9-what-is-idempotency-in-system-design)                                                                    |
| 10.   | [How do you handle retries in distributed systems?](#question-10-how-do-you-handle-retries-in-distributed-systems)                                           |
| 11.   | [How do you monitor system performance?](#question-11-how-do-you-monitor-system-performance)                                                                 |
| 12.   | [What is observability in system design?](#question-12-what-is-observability-in-system-design)                                                               |
| 13.   | [What is centralized logging?](#question-13-what-is-centralized-logging)                                                                                     |
| 14.   | [How do you design a monitoring dashboard for microservices?](#question-14-how-do-you-design-a-monitoring-dashboard-for-microservices)                       |
| 15.   | [What is rate limiting and how do you implement it?](#question-15-what-is-rate-limiting-and-how-do-you-implement-it)                                         |
| 16.   | [How do you secure sensitive data in system design?](#question-16-how-do-you-secure-sensitive-data-in-system-design)                                         |
| 17.   | [What are common security vulnerabilities in system design?](#question-17-what-are-common-security-vulnerabilities-in-system-design)                         |
| 18.   | [What is encryption in transit vs encryption at rest?](#question-18-what-is-encryption-in-transit-vs-encryption-at-rest)                                     |
| 19.   | [How do you design an audit logging system?](#question-19-how-do-you-design-an-audit-logging-system)                                                         |
| 20.   | [How do you handle GDPR and data privacy requirements in system design?](#question-20-how-do-you-handle-gdpr-and-data-privacy-requirements-in-system-design) |

## Question 1. How do you design a system to handle millions of users?

# Direct answer

To design a system that handles millions of users, focus on **horizontal scalability, stateless services, caching, database scaling, load balancing, asynchronous processing, and fault tolerance**. The goal is to ensure the system can continue serving requests as traffic grows without becoming a bottleneck.

---

# Requirements / Problem Framing

Before designing, clarify:

### Functional Requirements

Depends on the product (social media, e-commerce, messaging, etc.), but generally:

- User authentication
- Read/write operations
- Data storage and retrieval
- Notifications/events

### Non-Functional Requirements

- Support millions of users
- Low latency (e.g., <100ms for common reads)
- High availability (99.9%+)
- Reliability and durability
- Fault tolerance
- Scalability

---

# High-Level Architecture

```text
                Users
                  |
            DNS Routing
                  |
           Load Balancer
                  |
      -----------------------
      |         |           |
   App-1     App-2      App-N
      |         |           |
      -----------------------
                  |
          Cache Layer
             (Redis)
                  |
      ----------------------
      |                    |
 Read Replicas      Message Queue
      |                    |
   Primary DB         Background Workers
      |
 Object Storage
(CDN for static content)
```

### Request Flow

1. User sends request.
2. Load balancer distributes traffic.
3. Stateless application servers process requests.
4. Frequently accessed data comes from cache.
5. Database stores persistent data.
6. Heavy tasks are pushed to queues.
7. CDN serves static assets globally.

---

# Deep Design Considerations

## 1. Load Balancing

Distribute traffic across many servers.

Examples:

- Round Robin
- Least Connections
- Weighted Routing

Benefits:

- Prevents server overload
- Enables horizontal scaling
- Improves availability

```text
1 Server -> 100K users

10 Servers -> 1M+ users
```

---

## 2. Stateless Application Servers

Store session data externally.

Bad:

```text
User -> Server A
Session stored on A
```

If A crashes, user logs out.

Good:

```text
User -> Any Server
Session -> Redis
```

Benefits:

- Easy scaling
- Easy failover
- Auto-scaling support

---

## 3. Caching

Database is usually the first bottleneck.

Use Redis/Memcached.

```text
Request
   |
Cache Hit --> Return
   |
Cache Miss
   |
Database
```

Cache:

- User profiles
- Product catalogs
- Feed data
- Session information

A cache can reduce DB load by 80–95%.

---

## 4. Database Scaling

### Vertical Scaling

```text
Bigger CPU
More RAM
Faster SSD
```

Easy but limited.

### Horizontal Scaling (Sharding)

```text
Shard 1 -> Users 1-10M
Shard 2 -> Users 10M-20M
Shard 3 -> Users 20M-30M
```

Benefits:

- Unlimited growth potential
- Higher throughput

Challenges:

- Cross-shard joins
- Rebalancing

---

## 5. Database Replication

```text
            Primary
           /       \
      Replica1   Replica2
```

### Writes

```text
Client -> Primary
```

### Reads

```text
Client -> Replica
```

Benefits:

- Higher read capacity
- Better availability
- Disaster recovery

---

## 6. Content Delivery Network (CDN)

Serve static content from edge locations.

```text
User India -> India Edge
User US    -> US Edge
```

Store:

- Images
- Videos
- CSS
- JS files

Benefits:

- Lower latency
- Reduced backend load

---

## 7. Asynchronous Processing

Don't perform expensive operations synchronously.

Example:

When a user uploads a photo:

Instead of:

```text
Upload
 -> Resize
 -> Generate Thumbnail
 -> Store
 -> Return
```

Use:

```text
Upload
 -> Save
 -> Queue Job
 -> Return Success
```

Workers process:

```text
Queue
  |
Workers
  |
Thumbnail Generation
```

Common tools:

- Kafka
- RabbitMQ
- Amazon SQS

---

## 8. Service Decomposition

As scale increases:

```text
User Service
Order Service
Payment Service
Notification Service
Search Service
```

Benefits:

- Independent scaling
- Independent deployments
- Fault isolation

---

## 9. Reliability and High Availability

### Multi-Instance Deployment

```text
App1
App2
App3
```

If App2 fails:

```text
Traffic -> App1 + App3
```

### Multi-AZ Deployment

```text
Region
 ├─ AZ1
 ├─ AZ2
 └─ AZ3
```

Protects against datacenter failures.

---

# Capacity / Sizing Example

Assume:

- 10 million daily active users
- 100 requests/day/user

Total requests/day:

```text
10M × 100
= 1 Billion requests/day
```

Average QPS:

```text
1,000,000,000 / 86,400
≈ 11,500 QPS
```

Peak traffic is usually 5–10× average:

```text
≈ 60K–120K QPS
```

System should be designed for peak load, not average load.

---

# Security / Observability

### Security

- HTTPS everywhere
- Authentication (JWT/OAuth)
- Rate limiting
- DDoS protection
- Encryption at rest

### Observability

Metrics:

- QPS
- Latency (P50/P95/P99)
- Error rates
- CPU/Memory

Tools:

- Prometheus
- Grafana
- ELK Stack
- OpenTelemetry

---

# Trade-offs

| Decision           | Pros                | Cons                          |
| ------------------ | ------------------- | ----------------------------- |
| Monolith           | Simple initially    | Hard to scale independently   |
| Microservices      | Independent scaling | Operational complexity        |
| SQL                | Strong consistency  | Harder horizontal scaling     |
| NoSQL              | Easy scaling        | Weaker consistency guarantees |
| Cache Aggressively | Lower latency       | Cache invalidation complexity |
| Async Processing   | Better throughput   | Eventual consistency          |

---

# Interview-ready Summary

"For a system serving millions of users, I would design stateless application servers behind load balancers, use Redis for caching, scale databases through replication and sharding, serve static content via a CDN, and offload heavy work to asynchronous queues. The system should be horizontally scalable, highly available across multiple zones, and monitored through comprehensive observability tooling. The key principle is eliminating single points of failure and ensuring every layer can scale independently."

## Question 2. What are the challenges in designing distributed systems?

## Question 3. What is fault tolerance in system design?

## Question 4. What are leader election algorithms (like Raft, Paxos)?

## Question 5. What is consensus in distributed systems?

## Question 6. How do you design a highly available system?

## Question 7. What are design considerations for disaster recovery?

## Question 8. What is circuit breaker pattern in microservices?

## Question 9. What is idempotency in system design?

## Question 10. How do you handle retries in distributed systems?

## Question 11. How do you monitor system performance?

## Question 12. What is observability in system design?

## Question 13. What is centralized logging?

## Question 14. How do you design a monitoring dashboard for microservices?

## Question 15. What is rate limiting and how do you implement it?

## Question 16. How do you secure sensitive data in system design?

## Question 17. What are common security vulnerabilities in system design?

## Question 18. What is encryption in transit vs encryption at rest?

## Question 19. How do you design an audit logging system?

## Question 20. How do you handle GDPR and data privacy requirements in system design?
