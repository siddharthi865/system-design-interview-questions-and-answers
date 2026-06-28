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

## **Direct answer**

A **service mesh** is an infrastructure layer that manages **service-to-service communication** in a distributed system, handling concerns like **traffic routing, security, observability, and reliability** without requiring changes to application code.

It typically works by deploying a **sidecar proxy alongside each service instance**, which intercepts all network traffic.

---

## **What problem does a service mesh solve?**

In microservices, every service must handle:

- Retry logic
- Timeouts
- mTLS encryption
- Load balancing
- Circuit breaking
- Observability (metrics, tracing)

Without a service mesh:

> Each service re-implements these concerns → duplicated logic + inconsistent behavior.

A service mesh centralizes these concerns at the infrastructure level.

---

## **High-level architecture**

```
        Control Plane
   (config, policies, routing)
            |
--------------------------------
|              |               |
Sidecar     Sidecar        Sidecar
Proxy       Proxy          Proxy
|             |              |
Service A   Service B    Service C
```

### Key idea:

Each service talks through a **sidecar proxy**, not directly to other services.

---

## **Core components**

### 1. **Data Plane (Proxies)**

- Runs next to each service instance
- Intercepts all inbound/outbound traffic
- Examples: Envoy proxy

Responsibilities:

- Load balancing
- Retry, timeout handling
- Traffic routing
- mTLS encryption/decryption

---

### 2. **Control Plane**

- Central management layer
- Configures all proxies
- Pushes policies and routing rules

Responsibilities:

- Service discovery rules
- Traffic policies (A/B testing, canary releases)
- Security policies (mTLS, auth rules)

Examples:

- Istio control plane
- Linkerd control plane

---

## **Key capabilities of a service mesh**

### 1. **Traffic management**

- Load balancing
- Canary deployments
- Blue/green deployments
- Traffic splitting (e.g., 90% v1, 10% v2)

---

### 2. **Security (Zero Trust)**

- Automatic **mutual TLS (mTLS)** between services
- Service identity management
- Fine-grained authorization policies

---

### 3. **Observability**

- Distributed tracing (request path across services)
- Metrics (latency, error rate, throughput)
- Logs at proxy level

---

### 4. **Resilience**

- Retries with backoff
- Circuit breaking
- Fault injection (for testing failures)

---

## **Example flow**

Request from Service A → Service B:

1. A sends request
2. Sidecar proxy (A) intercepts it
3. Proxy encrypts request (mTLS)
4. Routes it based on policy (e.g., version v2 of Service B)
5. Sidecar proxy (B) receives request
6. Decrypts and forwards to Service B

---

## **Service mesh vs API Gateway**

| Feature  | Service Mesh                | API Gateway                  |
| -------- | --------------------------- | ---------------------------- |
| Scope    | Internal service-to-service | External client-to-service   |
| Layer    | Infrastructure sidecars     | Edge layer                   |
| Purpose  | Microservice communication  | Client access control        |
| Examples | Istio, Linkerd              | Kong, NGINX, AWS API Gateway |

👉 Simple way to remember:

- API Gateway = entry door to system
- Service Mesh = internal plumbing

---

## **Trade-offs**

| Benefit                           | Cost                                               |
| --------------------------------- | -------------------------------------------------- |
| Centralized traffic control       | Added infrastructure complexity                    |
| Strong security (mTLS everywhere) | Performance overhead (proxy hop)                   |
| Better observability              | Operational overhead                               |
| No code changes needed            | Debugging becomes harder (extra abstraction layer) |

---

## **When to use a service mesh**

Use it when:

- You have **large microservice architecture**
- Need **zero-trust security**
- Require **advanced traffic control**
- Need **deep observability across services**

Avoid it when:

- System is small (monolith or few services)
- Team cannot handle operational complexity

---

## **Interview-ready summary**

A service mesh is an infrastructure layer that manages inter-service communication in microservices architectures. It uses sidecar proxies and a control plane to handle networking concerns like routing, security (mTLS), retries, and observability in a centralized way, allowing application services to remain focused purely on business logic while improving consistency and reliability across the system.

## Question 3. What is a monolithic kernel vs. microkernel in system design?

## **Direct answer**

A **monolithic kernel** is an operating system architecture where most core services (process management, memory management, file systems, device drivers) run in **kernel space**.

A **microkernel** keeps only the **minimal essential functions** in kernel space (like IPC, scheduling, and basic memory management), while other services run in **user space** as separate processes.

The key difference is:

> **Monolithic = everything in kernel space**
> **Microkernel = minimal kernel + user-space services**

---

## **Core idea comparison**

| Aspect             | Monolithic Kernel                | Microkernel                |
| ------------------ | -------------------------------- | -------------------------- |
| Kernel size        | Large                            | Small                      |
| Where services run | Kernel space                     | User space                 |
| Performance        | Fast (fewer context switches)    | Slower (IPC overhead)      |
| Modularity         | Low                              | High                       |
| Fault isolation    | Poor (driver crash can crash OS) | Strong (service isolation) |
| Maintainability    | Harder                           | Easier                     |
| Security           | Larger attack surface            | Smaller attack surface     |

---

## **Monolithic kernel design**

### **Direct idea**

All major OS services run inside the kernel.

### **Typical components inside kernel space**

- Process scheduler
- Memory management
- File system
- Device drivers
- Networking stack

### **Architecture**

```id="monolithic"
+--------------------------------------+
|            User Applications         |
+--------------------------------------+
|           System Calls              |
+--------------------------------------+
|   Monolithic Kernel (Kernel Space)  |
|  - File system                     |
|  - Drivers                         |
|  - Memory manager                 |
|  - Scheduler                     |
|  - Networking stack             |
+--------------------------------------+
|               Hardware              |
+--------------------------------------+
```

### **Pros**

- Very fast (direct function calls in kernel space)
- Lower overhead (no IPC between components)
- Simpler runtime performance model

### **Cons**

- A bug in a driver can crash the entire system
- Hard to maintain and evolve
- Large trusted computing base

### **Examples**

- Linux kernel (modular monolithic)
- Traditional UNIX

---

## **Microkernel design**

### **Direct idea**

Only the most essential functions remain in kernel; everything else runs as user-space services.

### **Kernel typically includes**

- Inter-process communication (IPC)
- Basic scheduling
- Minimal memory management

### **User-space services**

- File system
- Device drivers
- Network stack
- GUI system

### **Architecture**

```id="microkernel"
+--------------------------------------+
|        User Space Services          |
|  - File system                     |
|  - Drivers                        |
|  - Network stack                 |
|  - UI services                  |
+------------------▲-------------------+
                   | IPC
+------------------|-------------------+
|        Microkernel (Kernel Space)   |
|  - IPC mechanism                  |
|  - Scheduler                    |
|  - Basic memory mgmt          |
+--------------------------------------+
|              Hardware              |
+--------------------------------------+
```

---

## **Key design difference: communication model**

### Monolithic kernel

- Function calls inside kernel space
- Direct execution
- Very low latency

### Microkernel

- Services communicate via **message passing (IPC)**
- More context switches
- Higher latency but better isolation

---

## **Trade-offs (important for interviews)**

| Dimension     | Monolithic          | Microkernel                |
| ------------- | ------------------- | -------------------------- |
| Performance   | Better              | Worse (IPC overhead)       |
| Reliability   | Lower               | Higher                     |
| Security      | Weaker              | Stronger                   |
| Debugging     | Hard (kernel crash) | Easier (isolated services) |
| Extensibility | Hard                | Easy                       |

---

## **Real-world intuition**

### Monolithic kernel analogy

> A single large factory where all workers are inside one building.

- Fast communication (walk across room)
- If one department fails, whole factory may stop

---

### Microkernel analogy

> A campus with separate buildings communicating via couriers.

- Safer isolation
- Communication overhead between departments

---

## **Hybrid reality (important insight)**

Most modern systems are not pure:

- **Linux** → monolithic but modular (loadable kernel modules)
- **Windows NT** → hybrid kernel
- **macOS (XNU)** → hybrid (Mach microkernel + BSD layers)

👉 Real systems borrow from both models for balance.

---

## **When to choose what (system design mindset)**

### Monolithic is preferred when:

- Performance is critical
- Tight hardware integration is needed
- Simplicity of execution matters more than isolation

### Microkernel is preferred when:

- Security and isolation are critical
- Systems need high reliability (e.g., aerospace, embedded systems)
- You want modular, upgradable components

---

## **Interview-ready summary**

A monolithic kernel runs all core OS services in kernel space, making it fast but less fault-isolated. A microkernel keeps only minimal functionality in kernel space and moves other services like drivers and file systems into user space, improving modularity and reliability at the cost of performance due to IPC overhead. Modern OSs often use hybrid approaches to balance these trade-offs.

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
