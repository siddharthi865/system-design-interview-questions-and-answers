# Set 13

| S.No. | Question                                                                                                                                                                  |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you design a system for real-time collaboration?](#question-1-how-do-you-design-a-system-for-real-time-collaboration)                                             |
| 2.    | [What is a service mesh?](#question-2-what-is-a-service-mesh)                                                                                                             |
| 3.    | [What is an RPC (Remote Procedure Call)?](#question-3-what-is-an-rpc-remote-procedure-call)                                                                               |
| 4.    | [What is the API Gateway pattern?](#question-4-what-is-the-api-gateway-pattern)                                                                                           |
| 5.    | [What is the difference between synchronous API calls and asynchronous events?](#question-5-what-is-the-difference-between-synchronous-api-calls-and-asynchronous-events) |
| 6.    | [How do you design a streaming API system?](#question-6-how-do-you-design-a-streaming-api-system)                                                                         |
| 7.    | [What is the role of gRPC in microservices?](#question-7-what-is-the-role-of-grpc-in-microservices)                                                                       |
| 8.    | [What is circuit breaking in API communication?](#question-8-what-is-circuit-breaking-in-api-communication)                                                               |
| 9.    | [How do you handle cross-region replication for APIs?](#question-9-how-do-you-handle-cross-region-replication-for-apis)                                                   |
| 10.   | [How do you design a chat server with message ordering guarantees?](#question-10-how-do-you-design-a-chat-server-with-message-ordering-guarantees)                        |
| 11.   | [What is horizontal partitioning vs vertical partitioning?](#question-11-what-is-horizontal-partitioning-vs-vertical-partitioning)                                        |
| 12.   | [What is database federation?](#question-12-what-is-database-federation)                                                                                                  |
| 13.   | [How do you reduce tail latency in distributed systems?](#question-13-how-do-you-reduce-tail-latency-in-distributed-systems)                                              |
| 14.   | [How do you handle read-heavy workloads?](#question-14-how-do-you-handle-read-heavy-workloads)                                                                            |
| 15.   | [How do you handle write-heavy workloads?](#question-15-how-do-you-handle-write-heavy-workloads)                                                                          |
| 16.   | [What is connection pooling in databases?](#question-16-what-is-connection-pooling-in-databases)                                                                          |
| 17.   | [What is multi-tenancy in system design?](#question-17-what-is-multi-tenancy-in-system-design)                                                                            |
| 18.   | [How do you design a system to handle sudden traffic spikes?](#question-18-how-do-you-design-a-system-to-handle-sudden-traffic-spikes)                                    |
| 19.   | [What is graceful degradation in system design?](#question-19-what-is-graceful-degradation-in-system-design)                                                              |
| 20.   | [What is batching in performance optimization?](#question-20-what-is-batching-in-performance-optimization)                                                                |

## Question 1. How do you design a system for real-time collaboration?

# Direct answer

A **real-time collaboration system** allows multiple users to simultaneously edit the same document, whiteboard, spreadsheet, or code file while seeing each other's changes almost instantly.

The biggest challenge is ensuring that:

- Everyone eventually sees the same document.
- Concurrent edits don't overwrite each other.
- Updates propagate with very low latency (typically <100 ms).
- The system remains available even with network delays or temporary disconnects.

Popular examples include:

- Google Docs
- Figma
- Visual Studio Code

---

# Requirements / Problem framing

## Functional Requirements

- Create/open collaborative documents
- Multiple users edit simultaneously
- Show edits in real time
- Presence indicators (who is online)
- Cursor sharing
- Comments/chat
- Version history
- Undo/Redo
- Conflict resolution

## Non-functional Requirements

- Very low latency (<100 ms desirable)
- High availability
- Strong durability
- Scalable to millions of users
- Support temporary offline users
- Secure access control

---

# High-Level Architecture

```
                Client A
                    │
             WebSocket Connection
                    │
                Load Balancer
                    │
        -------------------------
        │                       │
 Collaboration Server      Collaboration Server
        │                       │
        --------Document Service---------
                    │
          Operational Engine
         (OT or CRDT Algorithm)
                    │
          Event Log / Message Queue
                    │
            Persistent Database
                    │
             Object Storage
```

---

# Components

## 1. Client

Maintains:

- Local document copy
- Cursor position
- Pending operations
- WebSocket connection
- Version number

Local edits are immediately applied for a responsive experience.

---

## 2. WebSocket Gateway

Real-time collaboration requires persistent bidirectional communication.

Why WebSocket?

- Low latency
- Push updates instantly
- No polling
- Efficient network usage

Flow:

```
User types

↓

WebSocket

↓

Collaboration Server

↓

Broadcast to others
```

---

## 3. Collaboration Server

Responsible for:

- Receiving operations
- Validating permissions
- Ordering operations
- Resolving conflicts
- Broadcasting updates

It is stateless, enabling horizontal scaling.

---

## 4. Collaboration Engine

This is the core of the system.

Two popular approaches:

### Option 1: Operational Transformation (OT)

Used by early collaborative editors.

Example

Original:

```
Hello
```

User A inserts:

```
Hello World
```

User B inserts:

```
Hello Everyone
```

OT transforms operations so both edits are preserved.

Advantages

- Mature
- Efficient
- Works well for text

Disadvantages

- Complex transformation logic
- Difficult to implement correctly

---

### Option 2: CRDT (Conflict-Free Replicated Data Type)

Modern systems increasingly use CRDTs.

Each operation is designed to merge automatically.

Example:

```
Replica A
Replica B

↓

Independent edits

↓

Merge

↓

Same final document
```

Advantages

- Naturally supports offline editing
- No central coordinator required
- Eventual consistency

Disadvantages

- Higher metadata overhead
- More memory usage

Interview answer:

> Google Docs originally used OT, while many newer collaborative systems use CRDTs because they simplify offline synchronization.

---

# Document Flow

Suppose User A types:

```
A
```

Flow:

```
Keyboard

↓

Client

↓

Apply locally

↓

Send operation

↓

Server

↓

Assign sequence number

↓

Store event

↓

Broadcast

↓

Other clients

↓

Apply operation
```

Latency is typically:

```
40–100 ms
```

---

# Data Model

Document

```
Document
---------
documentId
owner
title
createdAt
updatedAt
```

Operation

```
Operation
---------
operationId
documentId
userId
type
position
content
version
timestamp
```

Example operation

```json
{
  "type": "insert",
  "position": 15,
  "text": "hello"
}
```

---

# Scaling Strategy

## Partition documents

Each document belongs to one collaboration server.

```
Document A

↓

Server 1

Document B

↓

Server 2
```

This avoids coordination for every edit.

---

## Sticky Routing

All users editing the same document should reach the same collaboration server.

Use

```
hash(documentId)
```

to consistently route users.

---

## Horizontal Scaling

```
Client

↓

Load Balancer

↓

Server 1

Server 2

Server 3

Server 4
```

Servers remain stateless except for active document sessions.

---

## Event Streaming

Persist every edit using a durable log (e.g., a distributed event stream).

Benefits:

- Recovery
- Replay
- Audit trail
- Version history

---

# Caching

Frequently edited documents remain in memory.

```
RAM

↓

Current document state

↓

Periodic snapshot

↓

Database
```

Benefits:

- Faster reads
- Lower database load

---

# Persistence Strategy

Avoid writing the full document after every keystroke.

Instead:

```
Document

+

Operation Log
```

Every few minutes:

```
Snapshot

+

Clear old operations
```

This minimizes storage and improves recovery.

---

# Presence Service

Tracks:

- Active users
- Cursor positions
- Typing indicators
- Selection ranges

Typically stored in memory rather than a database.

---

# Reliability

## Durable Event Log

Never lose edits.

```
Edit

↓

Event Log

↓

Ack client
```

---

## Replication

Replicate operations across multiple nodes.

```
Primary

↓

Replica A

Replica B
```

Protects against node failures.

---

## Recovery

If a collaboration server crashes:

```
Latest Snapshot

+

Replay Operations

↓

Current Document
```

This reconstructs the latest state.

---

# Consistency

Real-time editors often use **eventual consistency**.

Reason:

Waiting for global consensus on every keystroke would introduce noticeable latency.

Conflict resolution algorithms (OT or CRDT) ensure all clients eventually converge to the same document state.

---

# Security / Observability

### Security

- Authentication (JWT/OAuth)
- Authorization (document ACLs)
- TLS for WebSocket connections
- Input validation
- Rate limiting to prevent abuse

### Observability

Monitor:

- Edit latency
- WebSocket connection count
- Operations/sec
- Synchronization failures
- Conflict resolution rate
- Reconnection frequency
- Server CPU and memory
- Event queue lag

---

# Trade-offs

| Decision             | Pros                              | Cons                         |
| -------------------- | --------------------------------- | ---------------------------- |
| WebSocket vs Polling | Low latency, efficient            | Persistent connections       |
| OT                   | Proven for text editing           | Complex implementation       |
| CRDT                 | Offline-friendly, simpler merging | Higher metadata overhead     |
| Event sourcing       | Full history, replay              | More storage                 |
| Sticky routing       | Simple coordination               | Rebalancing challenges       |
| Snapshots + logs     | Fast recovery                     | Snapshot management required |

---

# Interview-ready summary

> "For a real-time collaboration system, I'd use persistent WebSocket connections to deliver low-latency updates. Clients optimistically apply local edits and send operations to a collaboration server responsible for ordering and broadcasting them. Conflicts are resolved using Operational Transformation (OT) or CRDTs. Active document state is kept in memory for speed, while all edits are durably persisted in an append-only event log with periodic snapshots for efficient recovery. Documents are partitioned across collaboration servers using consistent hashing on the document ID, enabling horizontal scaling while maintaining low latency and eventual consistency across all collaborators."

## Question 2. What is a service mesh?

# Direct answer

A **service mesh** is an infrastructure layer that manages **service-to-service communication** in a microservices architecture. Instead of each service implementing networking features like retries, load balancing, encryption, authentication, and observability, the service mesh provides these capabilities transparently through **sidecar proxies** or similar data-plane components.

In simple terms:

> **Application code handles business logic, while the service mesh handles communication between services.**

---

# Why do we need a service mesh?

As the number of microservices grows, each service must communicate reliably and securely with many others.

Without a service mesh, every service needs to implement:

- Service discovery
- Load balancing
- Retry logic
- Timeouts
- Circuit breakers
- Mutual TLS (mTLS)
- Metrics and tracing
- Traffic routing

This leads to duplicated code and inconsistent behavior.

A service mesh centralizes these networking concerns.

---

# High-Level Architecture

```text
               Control Plane
      (Configuration & Policy)
                │
        --------------------
        │                  │
     Proxy A            Proxy B
        │                  │
   Service A  ─────────► Service B
        ▲                  ▲
   All traffic flows through proxies
```

There are two main components:

### 1. Data Plane

The data plane consists of **sidecar proxies** deployed alongside each service.

Responsibilities:

- Route requests
- Load balance traffic
- Encrypt communication (mTLS)
- Retry failed requests
- Collect metrics
- Generate distributed traces
- Enforce policies

The application communicates with its local proxy, which then communicates with the destination proxy.

---

### 2. Control Plane

The control plane manages all proxies.

Responsibilities:

- Configure routing rules
- Push security policies
- Manage certificates
- Monitor proxy health
- Control traffic distribution

The control plane does **not** handle application traffic directly.

---

# Request Flow

```text
Client

↓

Service A

↓

Sidecar Proxy A

↓

Network

↓

Sidecar Proxy B

↓

Service B
```

Neither service needs to know about:

- TLS
- Retries
- Load balancing
- Observability

The proxies handle these concerns automatically.

---

# Features Provided by a Service Mesh

| Feature           | Description                                 |
| ----------------- | ------------------------------------------- |
| Service discovery | Finds healthy service instances             |
| Load balancing    | Distributes requests across instances       |
| Retries           | Retries transient failures automatically    |
| Timeouts          | Prevents hanging requests                   |
| Circuit breakers  | Stops sending traffic to unhealthy services |
| mTLS              | Encrypts service-to-service communication   |
| Authentication    | Verifies service identity                   |
| Authorization     | Enforces communication policies             |
| Traffic routing   | Canary, blue-green, A/B deployments         |
| Observability     | Metrics, logs, and distributed tracing      |

---

# Example: Retry Without a Service Mesh

Without a service mesh:

```javascript
async function callPayment() {
  for (let i = 0; i < 3; i++) {
    try {
      return await paymentService();
    } catch (e) {}
  }
}
```

Every service may implement retries differently.

With a service mesh:

- The application makes a single request.
- The proxy performs retries according to centralized policy.
- Retry behavior is consistent across all services.

---

# Traffic Management Example

Suppose you're deploying a new version of a service.

Instead of sending all traffic to the new version immediately:

```text
90% → Version 1
10% → Version 2
```

If Version 2 performs well:

```text
50% → Version 2
```

Finally:

```text
100% → Version 2
```

This enables safer canary deployments without modifying application code.

---

# Security

A service mesh commonly provides **mutual TLS (mTLS)**:

```text
Service A

⇄ Encrypted + Authenticated ⇄

Service B
```

Benefits:

- Encryption in transit
- Strong service identity
- Automatic certificate rotation
- Protection against impersonation

---

# Observability

Because every request passes through proxies, the mesh can automatically collect:

- Request latency
- Error rates
- Request volume
- Retries
- Timeouts
- Distributed traces
- Service dependency graphs

This provides visibility without adding instrumentation for every networking concern.

---

# Trade-offs

| Advantages                            | Disadvantages                      |
| ------------------------------------- | ---------------------------------- |
| Removes networking code from services | Additional operational complexity  |
| Consistent retries and timeouts       | Extra CPU and memory for proxies   |
| Automatic mTLS                        | Slight increase in request latency |
| Rich observability                    | Learning curve for teams           |
| Advanced traffic management           | More infrastructure to manage      |

---

# When should you use a service mesh?

A service mesh is most beneficial when:

- You have **dozens or hundreds of microservices**.
- Security between services is critical.
- You need consistent traffic policies and observability.
- You perform frequent canary or blue-green deployments.

It is often **not** worth the complexity for:

- Small systems with only a few services.
- Monolithic applications.
- Simple microservice deployments where basic API gateway and load balancing are sufficient.

---

# Interview-ready summary

> "A service mesh is an infrastructure layer that manages service-to-service communication in a microservices architecture. It uses sidecar proxies (data plane) controlled by a centralized control plane to provide capabilities such as service discovery, load balancing, retries, circuit breaking, mTLS, traffic routing, and observability without requiring changes to application code. This lets developers focus on business logic while the mesh consistently handles networking and security concerns across the system."

## Question 3. What is an RPC (Remote Procedure Call)?

# Direct answer

A **Remote Procedure Call (RPC)** is a communication mechanism that allows a program to invoke a function or method running on another machine **as if it were a local function call**. The RPC framework hides the complexities of network communication, serialization, and transport from the developer.

In simple terms:

> **RPC makes calling a remote service feel like calling a local method.**

Examples include gRPC, Apache Thrift, and JSON-RPC.

---

# How RPC works

Suppose Service A needs user information from Service B.

Instead of manually constructing HTTP requests:

```text
GET /users/123
```

You simply call:

```javascript
const user = userService.getUser(123);
```

Behind the scenes, the RPC framework:

1. Serializes the request into bytes.
2. Sends it over the network.
3. Executes the method on the remote server.
4. Serializes the response.
5. Returns the result to the caller.

The network communication is abstracted away.

---

# Architecture

```text
Service A

getUser(123)

↓

Client Stub (Proxy)

↓

Serialize Request

↓

Network (HTTP/2, TCP, etc.)

↓

Server Stub

↓

User Service

↓

Response

↓

Client receives result
```

The **client stub** acts as a local proxy, while the **server stub** unmarshals the request, invokes the actual method, and returns the response.

---

# Components

### 1. Client

Calls a method that appears local.

```javascript
const user = userService.getUser(123);
```

---

### 2. Client Stub (Proxy)

Responsible for:

- Serializing parameters
- Sending the request
- Receiving the response
- Deserializing the result

---

### 3. Network Transport

Carries the request.

Common transports:

- HTTP/2 (used by gRPC)
- TCP
- QUIC (in some modern implementations)

---

### 4. Server Stub

Responsible for:

- Deserializing the request
- Calling the appropriate method
- Serializing the response

---

### 5. Server

Executes the business logic.

```javascript
function getUser(id) {
  return database.find(id);
}
```

---

# Serialization

Since objects cannot be sent directly over the network, RPC frameworks serialize them.

Common formats:

| Format           | Characteristics                 |
| ---------------- | ------------------------------- |
| Protocol Buffers | Compact, fast, strongly typed   |
| JSON             | Human-readable, larger payloads |
| Avro             | Common in data pipelines        |
| MessagePack      | Binary and compact              |

For example, gRPC typically uses **Protocol Buffers**, which are smaller and faster than JSON.

---

# RPC vs REST

| Feature              | RPC                   | REST                   |
| -------------------- | --------------------- | ---------------------- |
| Communication style  | Method/function calls | Resource-oriented APIs |
| Example              | `getUser(123)`        | `GET /users/123`       |
| Payload              | Usually binary        | Usually JSON           |
| Performance          | High                  | Moderate               |
| Type safety          | Strong                | Often weaker           |
| Human readability    | Lower                 | Higher                 |
| Browser friendliness | Limited               | Excellent              |

---

# When to use RPC

RPC is a good choice for:

- Internal microservice communication
- Low-latency systems
- High-throughput services
- Strongly typed APIs
- Polyglot environments where multiple programming languages interact

REST is often preferred for:

- Public APIs
- Browser-based clients
- APIs that benefit from HTTP semantics and caching
- Simpler integrations

---

# Design considerations

Even though RPC feels like a local call, it's still a **network call**, so you must account for:

- Network latency
- Timeouts
- Retries
- Partial failures
- Idempotency for retried operations
- Circuit breakers
- Load balancing

A common interview point is:

> **Never treat an RPC like an in-process function call. Network failures are inevitable and must be handled explicitly.**

---

# Trade-offs

| Advantages                 | Disadvantages                                     |
| -------------------------- | ------------------------------------------------- |
| Simple programming model   | Can hide network complexity from developers       |
| High performance           | Harder to debug than plain HTTP                   |
| Strong typing              | Less human-readable                               |
| Efficient binary protocols | Browser support may require additional proxies    |
| Automatic code generation  | Tighter coupling through shared service contracts |

---

# Interview-ready summary

> "RPC allows one service to invoke a method on another machine as though it were a local function call. The RPC framework handles serialization, transport, request routing, and response deserialization. Frameworks like gRPC use Protocol Buffers and HTTP/2 to provide efficient, strongly typed communication, making RPC a popular choice for internal microservice communication where performance and type safety are important."

## Question 4. What is the API Gateway pattern?

## Question 5. What is the difference between synchronous API calls and asynchronous events?

## Question 6. How do you design a streaming API system?

## Question 7. What is the role of gRPC in microservices?

## Question 8. What is circuit breaking in API communication?

## Question 9. How do you handle cross-region replication for APIs?

## Question 10. How do you design a chat server with message ordering guarantees?

## Question 11. What is horizontal partitioning vs vertical partitioning?

## Question 12. What is database federation?

## Question 13. How do you reduce tail latency in distributed systems?

## Question 14. How do you handle read-heavy workloads?

## Question 15. How do you handle write-heavy workloads?

## Question 16. What is connection pooling in databases?

## Question 17. What is multi-tenancy in system design?

## Question 18. How do you design a system to handle sudden traffic spikes?

## Question 19. What is graceful degradation in system design?

## Question 20. What is batching in performance optimization?
