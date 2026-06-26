# Set 21

| S.No. | Question                                                                                                                                                                    |
| ----- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What are the key components of a distributed system?](#question-1-what-are-the-key-components-of-a-distributed-system)                                                     |
| 2.    | [What is a service mesh?](#question-2-what-is-a-service-mesh)                                                                                                               |
| 3.    | [What is a monolithic kernel vs. microkernel in system design?](#question-3-what-is-a-monolithic-kernel-vs-microkernel-in-system-design)                                    |
| 4.    | [How do you ensure API compatibility across versions?](#question-4-how-do-you-ensure-api-compatibility-across-versions)                                                     |
| 5.    | [What is the CAP theorem trade-off for DynamoDB?](#question-5-what-is-the-cap-theorem-trade-off-for-dynamodb)                                                               |
| 6.    | [How would you design a read-heavy blog site?](#question-6-how-would-you-design-a-read-heavy-blog-site)                                                                     |
| 7.    | [What is the role of event loops in system performance?](#question-7-what-is-the-role-of-event-loops-in-system-performance)                                                 |
| 8.    | [How do you handle partial failures in distributed systems?](#question-8-how-do-you-handle-partial-failures-in-distributed-systems)                                         |
| 9.    | [What is write-ahead logging, and why is it important?](#question-9-what-is-write-ahead-logging-and-why-is-it-important)                                                    |
| 10.   | [How do you ensure high throughput in a messaging system?](#question-10-how-do-you-ensure-high-throughput-in-a-messaging-system)                                            |
| 11.   | [Explain the difference between pessimistic and optimistic concurrency control](#question-11-explain-the-difference-between-pessimistic-and-optimistic-concurrency-control) |
| 12.   | [What are data sharding strategies in SQL databases?](#question-12-what-are-data-sharding-strategies-in-sql-databases)                                                      |
| 13.   | [How do you ensure real-time updates in a collaborative editor?](#question-13-how-do-you-ensure-real-time-updates-in-a-collaborative-editor)                                |
| 14.   | [What's the difference between RPC and gRPC?](#question-14-whats-the-difference-between-rpc-and-grpc)                                                                       |
| 15.   | [How do you design APIs to handle rate limits gracefully?](#question-15-how-do-you-design-apis-to-handle-rate-limits-gracefully)                                            |
| 16.   | [What are circuit breakers in system design?](#question-16-what-are-circuit-breakers-in-system-design)                                                                      |
| 17.   | [What is separation of concerns in LLD?](#question-17-what-is-separation-of-concerns-in-lld)                                                                                |
| 18.   | [How do you reduce the impact of long-tail latency in distributed systems?](#question-18-how-do-you-reduce-the-impact-of-long-tail-latency-in-distributed-systems)          |
| 19.   | [What is dual-write inconsistency, and how do you avoid it?](#question-19-what-is-dual-write-inconsistency-and-how-do-you-avoid-it)                                         |
| 20.   | [How do you implement soft deletes in database design?](#question-20-how-do-you-implement-soft-deletes-in-database-design)                                                  |

## Question 1. What are the key components of a distributed system?

## **Direct answer**

A distributed system is built from multiple independent components that coordinate over a network to appear as a single coherent system. The key components typically include **clients, services/nodes, communication layer, data storage, coordination mechanisms, and reliability infrastructure**.

---

## **Key components of a distributed system**

### 1. **Nodes (Compute Layer)**

These are the machines or processes that do the actual work.

- **Client nodes**: Initiate requests (web apps, mobile apps, APIs)
- **Server nodes / service nodes**: Handle business logic
- **Worker nodes**: Run background jobs (queues, batch processing)
- **Stateful vs stateless nodes**
  - Stateless services scale horizontally easily
  - Stateful services require replication/sharding

👉 Example: In a ride-hailing system, trip matching service nodes handle incoming ride requests.

---

### 2. **Communication Layer**

Defines how nodes talk to each other.

- **Synchronous communication**
  - HTTP/REST, gRPC
  - Request/response model

- **Asynchronous communication**
  - Message queues (Kafka, RabbitMQ)
  - Event-driven systems

Key concerns:

- Serialization (JSON, Protobuf)
- Latency
- Retry and timeout handling
- Network failures

---

### 3. **Data Storage Layer**

Responsible for persistence and retrieval of data.

- **Databases**
  - SQL (strong consistency)
  - NoSQL (scalability, flexible schema)

- **Distributed storage systems**
  - Replication (availability)
  - Sharding (scalability)

- **Caching systems**
  - Redis, Memcached

Key challenges:

- Consistency vs availability
- Replication lag
- Data partitioning

---

### 4. **Coordination & Consensus Layer**

Ensures nodes agree on system state.

Used for:

- Leader election
- Distributed locking
- Configuration management

Common tools/algorithms:

- ZooKeeper, etcd, Consul
- Raft, Paxos

👉 Example: Ensuring only one node processes a job in a distributed queue.

---

### 5. **Load Balancing Layer**

Distributes traffic across nodes.

- **L4 load balancer** (TCP-level)
- **L7 load balancer** (HTTP-aware routing)
- Algorithms: round robin, least connections, consistent hashing

👉 Prevents overload and improves fault tolerance.

---

### 6. **Fault Tolerance & Reliability Mechanisms**

Handles failures gracefully.

- Replication (data and services)
- Retry mechanisms (with exponential backoff)
- Circuit breakers
- Dead letter queues (DLQ)
- Failover strategies (active-active / active-passive)

👉 Ensures system continues working despite node/network failures.

---

### 7. **Consistency & State Management**

Defines how data remains correct across nodes.

- Strong consistency (immediate sync)
- Eventual consistency (delayed sync)
- Quorum-based reads/writes

Key concepts:

- CAP theorem trade-offs
- Read-repair, anti-entropy

---

### 8. **Security Layer**

Protects communication and data.

- Authentication (OAuth, JWT, mTLS)
- Authorization (RBAC, ABAC)
- Encryption (in transit + at rest)
- Service-to-service trust boundaries

---

### 9. **Observability Layer**

Helps understand system health.

- Logging (centralized logs)
- Metrics (Prometheus, Grafana)
- Distributed tracing (Jaeger, OpenTelemetry)
- Alerting systems

👉 Critical for debugging distributed failures.

---

## **High-level architecture view**

```
Clients
   |
Load Balancer
   |
Service Nodes (Stateless/Stateful)
   |
-------------------------------
| Communication Layer (sync/async)
| Data Layer (DBs, Cache)
| Coordination (ZooKeeper/etcd)
-------------------------------
   |
Monitoring + Security + Observability
```

---

## **Trade-offs in distributed systems**

| Area                        | Trade-off                                               |
| --------------------------- | ------------------------------------------------------- |
| Consistency vs Availability | Strong consistency reduces availability under partition |
| Latency vs Reliability      | More retries improve reliability but increase latency   |
| Stateful vs Stateless       | Stateless scales better, stateful is more complex       |
| Sync vs Async communication | Sync is simpler, async is more scalable                 |

---

## **Interview-ready summary**

A distributed system is composed of multiple independent nodes connected via a communication layer, backed by distributed storage, and coordinated through consensus mechanisms. It relies heavily on load balancing, fault tolerance strategies, and observability systems to ensure scalability and reliability under failures. The core challenge is managing trade-offs between consistency, availability, and performance while ensuring all components work together as a single logical system.

## Question 2. What is a service mesh?

## Question 3. What is a monolithic kernel vs. microkernel in system design?

## Question 4. How do you ensure API compatibility across versions?

## Question 5. What is the CAP theorem trade-off for DynamoDB?

## Question 6. How would you design a read-heavy blog site?

## Question 7. What is the role of event loops in system performance?

## Question 8. How do you handle partial failures in distributed systems?

## Question 9. What is write-ahead logging, and why is it important?

## Question 10. How do you ensure high throughput in a messaging system?

## Question 11. Explain the difference between pessimistic and optimistic concurrency control

## Question 12. What are data sharding strategies in SQL databases?

## Question 13. How do you ensure real-time updates in a collaborative editor?

## Question 14. What's the difference between RPC and gRPC?

## Question 15. How do you design APIs to handle rate limits gracefully?

## Question 16. What are circuit breakers in system design?

## Question 17. What is separation of concerns in LLD?

## Question 18. How do you reduce the impact of long-tail latency in distributed systems?

## Question 19. What is dual-write inconsistency, and how do you avoid it?

## Question 20. How do you implement soft deletes in database design?
