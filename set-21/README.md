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

## **Direct answer**

You ensure **API compatibility across versions** by following **backward-compatible evolution practices**: primarily using **versioning strategies, additive changes (not breaking changes), strict contract design, deprecation policies, and comprehensive testing** to guarantee old clients keep working while new functionality is introduced.

---

## **Requirements / problem framing**

In distributed systems, API compatibility matters because:

- Clients (mobile apps, services, partners) upgrade at different times
- Old and new versions often run **simultaneously**
- Breaking changes can cause system-wide outages

So the goal is:

> Allow APIs to evolve without breaking existing consumers.

---

## **Core strategies for API compatibility**

### 1. **Use explicit API versioning**

#### Common approaches:

| Strategy            | Example                               | Notes                       |
| ------------------- | ------------------------------------- | --------------------------- |
| URI versioning      | `/v1/users`, `/v2/users`              | Most explicit, widely used  |
| Header versioning   | `Accept: application/vnd.api.v2+json` | Cleaner URLs, more flexible |
| Query param         | `/users?version=2`                    | Simple but less preferred   |
| Content negotiation | MIME-based versioning                 | Advanced REST approach      |

👉 Most real-world systems use **URI versioning for simplicity**.

---

### 2. **Prefer backward-compatible (additive) changes**

Safe changes:

- Add new fields (optional)
- Add new endpoints
- Add new response attributes
- Extend enums (carefully)

Unsafe changes (breaking):

- Removing fields
- Renaming fields
- Changing field meaning/type
- Changing required parameters

👉 Rule of thumb:

> “Never break existing contracts—only extend them.”

---

### 3. **Strict contract design (schema-first APIs)**

Define APIs using:

- OpenAPI / Swagger
- Protobuf (gRPC)
- JSON Schema

This ensures:

- Strong typing
- Validation at compile/runtime
- Clear expectations for consumers

---

### 4. **Support graceful deprecation lifecycle**

Instead of removing APIs immediately:

#### Lifecycle:

1. **Introduce new version (v2)**
2. **Mark v1 as deprecated**
3. Add warnings in headers/logs
4. Maintain both versions for a grace period
5. Eventually sunset v1

Example:

```
Deprecation: true
Sunset: Sat, 01 Aug 2026 00:00:00 GMT
```

---

### 5. **Use backward-compatible serialization rules**

For JSON / Protobuf:

#### JSON rules:

- New fields must be optional
- Ignore unknown fields in clients
- Never reorder or rename fields blindly

#### Protobuf rules (very important in interviews):

- Never reuse field numbers
- Only add new optional fields
- Mark fields as deprecated, don’t delete immediately

---

### 6. **Strong client-server decoupling**

Ensure clients are resilient to change:

- Clients should ignore unknown fields
- Servers should handle missing optional fields
- Avoid tight coupling between internal models and API models

👉 Always use **DTOs (Data Transfer Objects)** instead of exposing DB models.

---

### 7. **Contract testing & CI enforcement**

Prevent accidental breaking changes:

- Consumer-driven contract testing (e.g., Pact)
- API schema diff tools
- CI checks for:
  - Removed fields
  - Type changes
  - Endpoint removals

---

### 8. **Feature flags for gradual rollout**

Instead of changing API behavior globally:

- Use feature flags to enable new behavior per:
  - user
  - region
  - percentage rollout

This helps:

- Safe migration
- A/B testing
- rollback capability

---

## **High-level evolution model**

```id="api_versioning_flow"
Client (v1) ---> API Gateway ---> Service (v1 + v2 logic)
                                 |
Client (v2) ---> API Gateway --->|
```

Both versions co-exist during migration.

---

## **Deep design considerations**

### 1. Backward vs forward compatibility

| Type                | Meaning                            |
| ------------------- | ---------------------------------- |
| Backward compatible | New system supports old clients    |
| Forward compatible  | Old clients tolerate new responses |

Good systems aim for **both**.

---

### 2. Avoiding "version explosion"

Bad design:

- v1, v2, v3, v4...

Better:

- Keep **2–3 active versions max**
- Prefer evolution over duplication

---

### 3. Data consistency across versions

Sometimes v1 and v2:

- write to same DB schema
- but expose different API views

This requires:

- internal canonical data model
- API-specific mapping layer

---

## **Trade-offs**

| Approach                      | Pros             | Cons                 |
| ----------------------------- | ---------------- | -------------------- |
| URI versioning                | Simple, explicit | URL duplication      |
| Header versioning             | Clean URLs       | Harder debugging     |
| Aggressive versioning         | Fast evolution   | Client fragmentation |
| Strict backward compatibility | Stable system    | Slower innovation    |

---

## **Common real-world pattern**

Most large systems combine:

- `/v1`, `/v2` API versioning
- Additive schema evolution
- Deprecation headers
- Contract testing in CI
- Feature flags for rollout

---

## **Interview-ready summary**

API compatibility is ensured by designing APIs to evolve safely through versioning strategies, backward-compatible (additive) changes, strict schema contracts, and controlled deprecation cycles. Combined with contract testing and feature flags, this allows distributed systems to evolve without breaking existing clients while supporting long-term scalability and maintainability.

## Question 5. What is the CAP theorem trade-off for DynamoDB?

## **Direct answer**

Amazon DynamoDB is designed to prioritize **Availability and Partition Tolerance (AP)** under the CAP theorem for most operations, while offering **tunable consistency** (eventual consistency by default, with optional strong consistency for reads within a region). It does **not provide full global strong consistency across partitions/regions in the CAP sense**, but it can offer **strongly consistent reads in a single region** with trade-offs in latency and availability.

---

## **CAP theorem framing (quick recap)**

CAP theorem states that in the presence of a **network partition (P)**, a distributed system must choose between:

| Property                | Meaning                                           |
| ----------------------- | ------------------------------------------------- |
| Consistency (C)         | All nodes see the same latest data                |
| Availability (A)        | Every request gets a response (even if stale)     |
| Partition Tolerance (P) | System continues operating despite network splits |

👉 In real distributed systems, **P is non-negotiable**, so the real trade-off is:

> **Consistency vs Availability during partitions**

---

## **DynamoDB’s CAP positioning**

### 1. **Primary mode: AP (Availability + Partition Tolerance)**

DynamoDB is built for:

- Always responding to requests
- Surviving node, AZ, and partial network failures
- Scaling horizontally without coordination bottlenecks

So for most reads:

- Uses **eventual consistency**
- Replicates data asynchronously across partitions/AZs

👉 This ensures:

> High availability even under failures or network partitions

---

### 2. **Optional mode: Strong consistency (within a region)**

DynamoDB supports:

- **Strongly consistent reads**
- But only within a **single region**

This changes behavior to:

- Reads wait for latest committed write
- Higher latency
- Reduced availability under certain failure scenarios

👉 So for strongly consistent reads, DynamoDB temporarily leans toward:

> **CP-like behavior (Consistency + Partition tolerance)** within region boundaries

---

## **How DynamoDB achieves this under the hood**

### Storage model:

- Data is partitioned by **partition key**
- Each partition is replicated across multiple nodes/AZs
- Writes are propagated asynchronously (eventual consistency)

### Read paths:

| Type                       | Behavior                                      |
| -------------------------- | --------------------------------------------- |
| Eventually consistent read | May return stale replica                      |
| Strongly consistent read   | Reads from quorum/leader ensuring latest data |

---

## **Important nuance (interview-critical)**

DynamoDB is **not purely AP or CP**.

Instead it is:

> **AP by default with configurable consistency trade-offs**

Because:

- In partitions → it prioritizes availability
- But allows consistency tuning per request

---

## **Failure scenario analysis**

### Network partition between replicas:

#### Eventual consistency (default):

- System continues serving reads/writes
- Some nodes may return stale data
- ✔ Available
- ✘ Not immediately consistent

#### Strong consistency:

- Must ensure latest write is visible
- May block or fail reads during partition
- ✔ Consistent
- ✘ Reduced availability

---

## **Trade-off summary**

| Dimension           | DynamoDB behavior                         |
| ------------------- | ----------------------------------------- |
| Availability        | Very high (core design goal)              |
| Consistency         | Eventual by default, optional strong      |
| Partition tolerance | Fully supported                           |
| Latency             | Low for eventual, higher for strong reads |
| Global consistency  | Not truly CP globally                     |

---

## **Why AWS chose this model**

DynamoDB is optimized for:

- Massive scale
- Always-on applications (shopping carts, gaming, IoT)
- Low operational overhead

So AWS prioritizes:

> “The system must always respond” over “every read must be perfectly up-to-date”

---

## **Comparison with CP systems (e.g., traditional databases)**

| System                            | CAP Bias     | Behavior under partition                    |
| --------------------------------- | ------------ | ------------------------------------------- |
| DynamoDB                          | AP (default) | Keeps serving data, may be stale            |
| Zookeeper / etcd                  | CP           | May reject requests to maintain correctness |
| Traditional RDBMS (single leader) | CP-ish       | May become unavailable on failover          |

---

## **Interview-ready summary**

DynamoDB is primarily an AP system in CAP terms, designed for high availability and partition tolerance with eventual consistency as the default. It provides optional strongly consistent reads within a region, which temporarily shifts behavior toward consistency at the cost of higher latency and reduced availability during failures. Overall, it trades strict global consistency for scalability, resilience, and always-on availability.

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
