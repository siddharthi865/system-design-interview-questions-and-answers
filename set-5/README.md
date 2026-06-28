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

# Direct answer

Designing distributed systems is challenging because components run on multiple machines connected over unreliable networks. Unlike a single-machine application, you must handle **network failures, partial failures, consistency issues, scalability, concurrency, latency, fault tolerance, and operational complexity** while still providing a reliable user experience.

---

# Key Challenges in Distributed Systems

## 1. Network Failures

In a distributed system, machines communicate over a network, and networks are inherently unreliable.

Possible issues:

- Packet loss
- Network partitions
- High latency
- Connection timeouts

Example:

```text
Service A ----X---- Service B
```

A doesn't know whether:

- B never received the request
- B processed it but response was lost
- B is down

This uncertainty makes distributed systems difficult.

---

## 2. Partial Failures

In a monolithic application:

```text
Server Down = System Down
```

In distributed systems:

```text
Service A -> Service B -> Service C
```

Only Service C may fail while A and B continue running.

Challenges:

- Detecting failures
- Retrying safely
- Preventing cascading failures

Common solutions:

- Timeouts
- Retries
- Circuit breakers
- Fallback mechanisms

---

## 3. Data Consistency

Multiple copies of data may exist across nodes.

Example:

```text
Primary DB
   |
Replication
   |
Replica DB
```

User updates profile:

```text
Write -> Primary
```

Immediately after:

```text
Read -> Replica
```

Replica may not have received the update yet.

Result:

```text
User sees stale data
```

Key challenge:

- Balancing consistency and availability

---

## 4. CAP Theorem

When a network partition occurs, a distributed system can choose only two of:

| Property            | Meaning                        |
| ------------------- | ------------------------------ |
| Consistency         | All nodes see same data        |
| Availability        | Every request gets a response  |
| Partition Tolerance | System survives network splits |

Since partition tolerance is mandatory in distributed systems, systems often trade off between:

- Consistency (CP)
- Availability (AP)

---

## 5. Concurrency and Race Conditions

Many users may update the same data simultaneously.

Example:

```text
Account Balance = $100
```

Two withdrawals:

```text
User A -> Withdraw $50
User B -> Withdraw $70
```

Without proper coordination:

```text
Final Balance = Incorrect
```

Solutions:

- Transactions
- Optimistic locking
- Distributed locks
- Versioning

---

## 6. Scalability

As traffic grows:

```text
100 Users
↓
10 Million Users
```

Challenges:

- Database bottlenecks
- Hot partitions
- Increased network traffic
- Uneven load distribution

Solutions:

- Load balancing
- Sharding
- Caching
- Horizontal scaling

---

## 7. Latency

Cross-machine communication is much slower than in-memory operations.

Approximate comparison:

| Operation    | Time                |
| ------------ | ------------------- |
| CPU Cache    | Nanoseconds         |
| RAM Access   | Tens of nanoseconds |
| SSD Access   | Microseconds        |
| Network Call | Milliseconds        |

A request touching multiple services:

```text
API
 ├─ User Service
 ├─ Order Service
 └─ Payment Service
```

can accumulate latency quickly.

---

## 8. Distributed Transactions

Maintaining ACID guarantees across multiple services is difficult.

Example:

```text
Order Service
Payment Service
Inventory Service
```

Scenario:

```text
Payment Success
Inventory Update Failed
```

System enters an inconsistent state.

Solutions:

- Saga Pattern
- Event-driven workflows
- Compensation transactions

---

## 9. Clock Synchronization

Each machine has its own clock.

```text
Server A: 10:00:01
Server B: 09:59:58
```

Problems:

- Event ordering
- Conflict resolution
- Distributed logging

Solutions:

- NTP
- Logical clocks
- Vector clocks
- Hybrid clocks

---

## 10. Service Discovery

In dynamic environments:

```text
Service Instances:
10 -> 100 -> 500
```

IP addresses change frequently.

Questions:

- How do services find each other?
- How do they know healthy instances?

Solutions:

- Service registry
- DNS-based discovery
- Health checks

Examples:

- Consul
- Eureka
- Kubernetes Service Discovery

---

## 11. Fault Tolerance

Machines fail regularly at scale.

Failures include:

- Disk crashes
- Server failures
- Datacenter outages
- Network failures

Need:

- Replication
- Automatic failover
- Redundancy
- Disaster recovery

---

## 12. Observability and Debugging

Debugging is harder when a request travels through many services.

Example:

```text
Client
  |
API Gateway
  |
User Service
  |
Order Service
  |
Payment Service
```

A single user request may generate dozens of logs.

Need:

- Centralized logging
- Metrics
- Distributed tracing

Tools:

- ELK Stack
- Prometheus
- Grafana
- OpenTelemetry
- Jaeger

---

# Trade-offs

| Challenge                         | Common Trade-off                 |
| --------------------------------- | -------------------------------- |
| Consistency vs Availability       | CAP theorem                      |
| Latency vs Accuracy               | Cache freshness                  |
| Throughput vs Durability          | Async processing                 |
| Simplicity vs Scalability         | Monolith vs Microservices        |
| Strong Consistency vs Performance | Synchronous vs Async replication |

---

# Interview-ready Summary

"The biggest challenges in distributed systems are handling unreliable networks, partial failures, data consistency, concurrency, scalability, latency, and fault tolerance. Since components run on different machines, communication can fail, data can become inconsistent, and services can experience independent failures. A good distributed system uses replication, caching, load balancing, retries, service discovery, observability, and carefully chosen consistency models to balance reliability, performance, and scalability."

## Question 3. What is fault tolerance in system design?

# Direct answer

**Fault tolerance** is the ability of a system to continue operating correctly even when some of its components fail. The goal is to minimize service disruption and avoid a single point of failure.

In large-scale distributed systems, failures are expected—not exceptional. A fault-tolerant system is designed to detect failures, isolate them, recover automatically, and continue serving users.

---

# Understanding with an Example

Without fault tolerance:

```text
Users
  |
Server A
```

If Server A crashes:

```text
Users
  |
Server A (DOWN)

System unavailable
```

With fault tolerance:

```text
           Load Balancer
          /             \
     Server A       Server B
```

If Server A fails:

```text
           Load Balancer
                  |
             Server B
```

Users can still access the service.

---

# Common Types of Failures

### Hardware Failures

- Disk crashes
- Memory failures
- Power outages
- Server failures

### Network Failures

- Packet loss
- Network partitions
- DNS failures
- High latency

### Software Failures

- Application crashes
- Memory leaks
- Deployment bugs
- Dependency failures

### Infrastructure Failures

- Datacenter outages
- Cloud region failures
- Load balancer failures

---

# Techniques for Achieving Fault Tolerance

## 1. Redundancy

Maintain multiple copies of critical components.

### Application Servers

```text
Load Balancer
    |
------------
|    |     |
A    B     C
```

If one server fails, others continue serving traffic.

### Databases

```text
Primary DB
   |
Replicas
```

If a replica fails, others remain available.

---

## 2. Replication

Store data on multiple nodes.

```text
User Data
   |
-------------
|     |     |
N1    N2    N3
```

Benefits:

- Data durability
- Faster recovery
- Higher availability

---

## 3. Automatic Failover

When a component fails, traffic is automatically redirected.

Example:

```text
Primary DB (DOWN)
        |
Automatic Failover
        |
Secondary DB becomes Primary
```

Users experience minimal downtime.

---

## 4. Health Checks

Continuously monitor components.

```text
Load Balancer
      |
 Health Checks
      |
 Server A
```

Unhealthy servers are removed from traffic routing.

---

## 5. Retry Mechanisms

Temporary failures often resolve themselves.

```text
Request
   |
Failure
   |
Retry
   |
Success
```

Use:

- Exponential backoff
- Retry limits
- Jitter

Avoid retry storms.

---

## 6. Circuit Breaker Pattern

Prevent repeated calls to a failing service.

Without circuit breaker:

```text
Service A -> Service B (failing)
```

Thousands of requests continue hitting B.

With circuit breaker:

```text
Service A
   |
Circuit Open
   |
Fallback Response
```

This prevents cascading failures.

---

## 7. Graceful Degradation

Provide reduced functionality instead of complete failure.

Example:

E-commerce site:

```text
Search Works
Recommendations Fail
```

Users can still purchase products.

Better than:

```text
Entire Website Down
```

---

## 8. Data Backups and Disaster Recovery

Protect against catastrophic failures.

Strategies:

- Periodic backups
- Point-in-time recovery
- Multi-region replication

Example:

```text
Region A Failure
      |
Traffic shifted
      |
Region B
```

---

# Fault Tolerance vs High Availability

| Aspect           | Fault Tolerance                     | High Availability               |
| ---------------- | ----------------------------------- | ------------------------------- |
| Goal             | Continue operating despite failures | Minimize downtime               |
| Failure Handling | System keeps functioning            | System recovers quickly         |
| Redundancy       | Required                            | Usually required                |
| Cost             | Higher                              | Lower than full fault tolerance |

### Example

**High Availability**

```text
Primary fails
↓
30 seconds downtime
↓
Backup starts
```

**Fault Tolerance**

```text
Primary fails
↓
Backup already active
↓
No visible interruption
```

---

# Trade-offs

| Benefit                 | Cost                     |
| ----------------------- | ------------------------ |
| Higher reliability      | More infrastructure      |
| Better uptime           | Increased complexity     |
| Faster recovery         | Higher operational cost  |
| Reduced business impact | More replication/storage |

There is no such thing as perfect fault tolerance. The goal is to achieve an acceptable level of reliability based on business requirements.

---

# Real-World Examples

### Netflix

Uses:

- Multi-region deployments
- Redundant services
- Chaos engineering (intentionally injecting failures)

### Amazon

Uses:

- Multiple Availability Zones
- Replicated storage
- Automatic failover

### Google

Uses:

- Distributed storage
- Replication across data centers
- Self-healing infrastructure

---

# Interview-ready Summary

"Fault tolerance is the ability of a system to continue functioning when components fail. It is achieved through redundancy, replication, failover mechanisms, health checks, retries, circuit breakers, and disaster recovery strategies. In distributed systems, failures are inevitable, so systems are designed to detect failures automatically and continue serving users with minimal or no interruption."

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
