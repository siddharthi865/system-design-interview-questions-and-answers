# Set 1

| S.No. | Question                                                                                                                                                                       |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1.    | [What is system design? Why is it important in software engineering?](#question-1-what-is-system-design-why-is-it-important-in-software-engineering)                           |
| 2.    | [Explain the difference between High-Level Design (HLD) and Low-Level Design (LLD)](#question-2-explain-the-difference-between-high-level-design-hld-and-low-level-design-lld) |
| 3.    | [What are functional and non-functional requirements in system design?](#question-3-what-are-functional-and-non-functional-requirements-in-system-design)                      |
| 4.    | [What is scalability in system design?](#question-4-what-is-scalability-in-system-design)                                                                                      |
| 5.    | [What is the difference between vertical scaling and horizontal scaling?](#question-5-what-is-the-difference-between-vertical-scaling-and-horizontal-scaling)                  |
| 6.    | [What are the pros and cons of vertical scaling?](#question-6-what-are-the-pros-and-cons-of-vertical-scaling)                                                                  |
| 7.    | [What are the pros and cons of horizontal scaling?](#question-7-what-are-the-pros-and-cons-of-horizontal-scaling)                                                              |
| 8.    | [Explain latency vs throughput](#question-8-explain-latency-vs-throughput)                                                                                                     |
| 9.    | [What is availability? How is it measured?](#question-9-what-is-availability-how-is-it-measured)                                                                               |
| 10.   | [What is consistency in distributed systems?](#question-10-what-is-consistency-in-distributed-systems)                                                                         |
| 11.   | [What is reliability in a system?](#question-11-what-is-reliability-in-a-system)                                                                                               |
| 12.   | [Explain the CAP theorem in simple terms](#question-12-explain-the-cap-theorem-in-simple-terms)                                                                                |
| 13.   | [What is partition tolerance in the CAP theorem?](#question-13-what-is-partition-tolerance-in-the-cap-theorem)                                                                 |
| 14.   | [What is load balancing and why is it needed?](#question-14-what-is-load-balancing-and-why-is-it-needed)                                                                       |
| 15.   | [What are caching strategies in system design?](#question-15-what-are-caching-strategies-in-system-design)                                                                     |
| 16.   | [What are indexes in databases and why are they important?](#question-16-what-are-indexes-in-databases-and-why-are-they-important)                                             |
| 17.   | [What is sharding?](#question-17-what-is-sharding)                                                                                                                             |
| 18.   | [What is replication?](#question-18-what-is-replication)                                                                                                                       |
| 19.   | [Difference between synchronous and asynchronous replication](#question-19-difference-between-synchronous-and-asynchronous-replication)                                        |
| 20.   | [What is eventual consistency?](#question-20-what-is-eventual-consistency)                                                                                                     |

## Question 1. What is system design? Why is it important in software engineering?

# Direct answer

**System design** is the process of defining the architecture, components, data flow, interfaces, and infrastructure of a software system so that it meets both functional and non-functional requirements.

In simple terms, system design answers questions like:

- How will the system work end-to-end?
- How will different services communicate?
- How will data be stored and retrieved?
- How will the system scale as traffic grows?
- How will it remain reliable and available during failures?

It acts as the blueprint for building software systems.

---

# What does system design involve?

A system designer typically decides:

| Area          | Examples                                  |
| ------------- | ----------------------------------------- |
| Architecture  | Monolith vs Microservices                 |
| Data Storage  | SQL vs NoSQL                              |
| Scalability   | Load balancing, caching, sharding         |
| Reliability   | Replication, failover, redundancy         |
| Communication | REST, gRPC, messaging queues              |
| Consistency   | Strong vs eventual consistency            |
| Security      | Authentication, authorization, encryption |
| Observability | Logging, monitoring, tracing              |

---

# Why is system design important?

### 1. Helps systems scale

A design that works for 100 users may fail for 10 million users.

Example:

- A single database server might work initially.
- As traffic grows, you may need:
  - Read replicas
  - Database sharding
  - Caching
  - Load balancers

Good system design anticipates growth.

---

### 2. Improves reliability and availability

Failures are inevitable:

- Servers crash
- Databases fail
- Networks become unavailable

System design introduces:

- Replication
- Failover mechanisms
- Redundancy
- Disaster recovery

This ensures the system continues operating even when components fail.

---

### 3. Meets performance requirements

Users expect fast responses.

For example:

- Search results should appear in milliseconds.
- Video streaming should not buffer frequently.

System design helps reduce:

- Latency
- Network overhead
- Database bottlenecks

through techniques like caching, indexing, and distributed processing.

---

### 4. Controls cost

Poor architecture can become extremely expensive.

Example:

- Hitting the database for every request increases infrastructure costs.
- A cache can serve most requests at a fraction of the cost.

Good design balances:

- Performance
- Reliability
- Cost

---

### 5. Enables maintainability

Large systems are built and maintained by many engineers.

Good design provides:

- Clear service boundaries
- Modular components
- Well-defined APIs

This makes systems easier to:

- Understand
- Modify
- Debug
- Extend

---

# Real-world example

Consider designing a URL shortener like TinyURL.

A naive design:

```
User -> Application -> Database
```

Works for a small number of users.

At scale:

```
Users
   |
Load Balancer
   |
App Servers
   |
Cache (Redis)
   |
Database Cluster
   |
Replication + Backups
```

Now the system can:

- Handle millions of requests
- Survive server failures
- Respond with low latency

This evolution is the essence of system design.

---

# What system design focuses on beyond functionality

Writing code answers:

> "Can the feature work?"

System design answers:

> "Can the feature work reliably for millions of users under real-world conditions?"

Typical concerns include:

- Scalability
- Availability
- Reliability
- Consistency
- Fault tolerance
- Performance
- Security
- Cost efficiency

---

# Interview-ready summary

> System design is the process of defining a software system's architecture, components, data storage, communication patterns, and infrastructure to satisfy both functional and non-functional requirements. It is important because it ensures systems can scale, remain reliable, perform efficiently, tolerate failures, and be maintained as user traffic and business requirements grow.

## Question 2. Explain the difference between High-Level Design (HLD) and Low-Level Design (LLD)

# Direct answer

**High-Level Design (HLD)** describes the overall architecture of a system—major components, services, databases, and how they interact.

**Low-Level Design (LLD)** describes the internal implementation details of those components—classes, methods, data structures, design patterns, and object relationships.

A simple way to remember it:

- **HLD = What are we building and how do major pieces interact?**
- **LLD = How exactly is each piece implemented?**

---

# HLD vs LLD

| Aspect               | High-Level Design (HLD)                     | Low-Level Design (LLD)              |
| -------------------- | ------------------------------------------- | ----------------------------------- |
| Focus                | System architecture                         | Component implementation            |
| Audience             | Architects, senior engineers, stakeholders  | Developers                          |
| Abstraction Level    | High                                        | Detailed                            |
| Scope                | Entire system                               | Individual modules/services         |
| Diagrams             | Architecture diagrams, service interactions | Class diagrams, sequence diagrams   |
| Technology Decisions | Often included                              | Usually fixed already               |
| Main Question        | "How will the system be structured?"        | "How will this component be coded?" |

---

# Example: Designing a URL Shortener

## High-Level Design (HLD)

At HLD level, we discuss:

```text
Users
   |
Load Balancer
   |
Application Servers
   |
Redis Cache
   |
Database
```

Topics include:

- API design
- Database choice
- Caching strategy
- Scalability
- Replication
- Load balancing
- Availability

Questions answered:

- Should we use SQL or NoSQL?
- Do we need caching?
- How do we generate unique short URLs?
- How do we scale to millions of requests?

---

## Low-Level Design (LLD)

Inside the URL Shortener service, we discuss:

```java
class UrlService {
    ShortUrl createShortUrl(String longUrl);
    String getOriginalUrl(String shortCode);
}

class UrlRepository {
    save();
    findByShortCode();
}

class ShortCodeGenerator {
    generate();
}
```

Topics include:

- Classes
- Interfaces
- Object relationships
- Design patterns
- Method signatures
- Error handling

Questions answered:

- What classes are needed?
- What methods should each class expose?
- Which design pattern should be used?
- How should validation be implemented?

---

# Another Example: Food Delivery App

## HLD View

```text
Customer App
     |
API Gateway
     |
-------------------------
|      |       |        |
User  Order Restaurant Payment
Svc   Svc     Svc       Svc
     |
 Message Queue
```

Discussion focuses on:

- Microservices
- Service communication
- Databases
- Caching
- Event-driven architecture
- Scalability

---

## LLD View

Inside Order Service:

```text
Order
OrderItem
OrderManager
PaymentProcessor
OrderRepository
```

Discussion focuses on:

- Class design
- Relationships
- Interfaces
- Business rules
- State transitions

For example:

```text
CREATED
   ↓
PAID
   ↓
PREPARING
   ↓
DELIVERED
```

---

# When are they used?

### HLD is used when:

- Designing a new system
- Discussing architecture
- Planning scalability
- Conducting system design interviews

Examples:

- Design Instagram
- Design WhatsApp
- Design YouTube

---

### LLD is used when:

- Implementing features
- Writing code
- Creating reusable components
- Conducting object-oriented design interviews

Examples:

- Design a Parking Lot
- Design an Elevator System
- Design a Vending Machine

---

# Relationship between HLD and LLD

They are not separate activities.

```text
Requirements
     ↓
High-Level Design
     ↓
Low-Level Design
     ↓
Implementation
     ↓
Testing
```

HLD defines the architecture.

LLD defines how each architectural component is implemented.

---

# Common interview perspective

In a **System Design Interview**, interviewers usually expect mostly **HLD**:

- APIs
- Databases
- Caching
- Load balancing
- Replication
- Sharding
- Scalability trade-offs

In an **Object-Oriented Design (OOD/LLD) Interview**, interviewers expect:

- Classes
- Interfaces
- Design patterns
- SOLID principles
- Relationships
- Extensibility

---

# Interview-ready summary

> High-Level Design (HLD) focuses on the overall architecture of a system, including services, databases, communication patterns, and scalability. Low-Level Design (LLD) focuses on the detailed implementation of individual components, including classes, interfaces, methods, and design patterns. HLD answers "how the system is structured," while LLD answers "how each part is implemented."

## Question 3. What are functional and non-functional requirements in system design?

# Direct answer

**Functional requirements** define **what a system should do**—the features and behaviors users expect.

**Non-functional requirements (NFRs)** define **how well the system should perform those functions**—such as scalability, availability, latency, reliability, and security.

A simple way to remember:

- **Functional Requirements = Features**
- **Non-Functional Requirements = Quality Attributes**

---

# Functional Requirements

Functional requirements describe the business capabilities of the system.

They answer:

> "What should the system do?"

### Examples

For a URL Shortener:

- User can submit a long URL.
- System generates a short URL.
- User can access the original URL using the short URL.
- User can delete a shortened URL.
- User can view click statistics.

For a Food Delivery App:

- Browse restaurants.
- Search for food.
- Place orders.
- Make payments.
- Track deliveries.

---

# Non-Functional Requirements

Non-functional requirements describe system qualities and constraints.

They answer:

> "How should the system behave?"

### Common NFRs

| Category        | Example                               |
| --------------- | ------------------------------------- |
| Scalability     | Handle 10 million users               |
| Availability    | 99.99% uptime                         |
| Latency         | Response time < 200 ms                |
| Reliability     | No data loss during failures          |
| Durability      | Orders persist even after crashes     |
| Security        | Data encrypted in transit and at rest |
| Consistency     | Users see correct data                |
| Throughput      | Process 100,000 requests/sec          |
| Maintainability | Easy to update and debug              |
| Observability   | Metrics, logs, tracing                |

---

# Example: URL Shortener

## Functional Requirements

- Create short URLs.
- Redirect users to original URLs.
- Support custom aliases.
- Track click counts.

## Non-Functional Requirements

- Redirect latency < 50 ms.
- 99.99% availability.
- Support 100 million URLs.
- Handle 100,000 redirects/sec.
- No loss of URL mappings.

---

# Example: Instagram

## Functional Requirements

- Upload photos.
- Follow users.
- Like posts.
- View feeds.
- Send messages.

## Non-Functional Requirements

- Feed loads within 200 ms.
- Store billions of photos.
- Support millions of concurrent users.
- Highly available worldwide.
- Secure user data.

---

# Why are Non-Functional Requirements important?

Many systems can satisfy functional requirements.

The challenge is satisfying them **at scale**.

For example:

Two systems may both support "upload a photo."

| System A                | System B                   |
| ----------------------- | -------------------------- |
| Upload takes 10 seconds | Upload takes 500 ms        |
| Frequently crashes      | Highly available           |
| Doesn't scale           | Supports millions of users |

Both meet the functional requirement.

Only one meets the non-functional requirements.

This is why system design interviews focus heavily on NFRs.

---

# How system design interviews use them

A typical interview starts by gathering requirements.

Example:

**Design Twitter**

Functional requirements:

- Post tweets.
- Follow users.
- View timeline.
- Like tweets.

Non-functional requirements:

- Millions of daily active users.
- Timeline should load quickly.
- High availability.
- Eventual consistency is acceptable for likes.
- Global scale.

These NFRs heavily influence architectural decisions such as:

- Caching
- Database selection
- Replication strategy
- Sharding
- Load balancing

---

# Common mistake in interviews

Many candidates spend all their time discussing features:

❌ "Users can create posts, comment, and like."

Strong candidates quickly move to:

✅ "We expect 100M users, need 99.99% availability, feed latency under 200 ms, and eventual consistency is acceptable."

Those requirements drive the architecture.

---

# Interview-ready summary

> Functional requirements define what the system must do, such as creating posts, sending messages, or processing payments. Non-functional requirements define how well the system must perform, including scalability, availability, latency, reliability, security, and consistency. In system design, non-functional requirements are often the primary drivers of architectural decisions because they determine how the system behaves under real-world scale and failures.

## Question 4. What is scalability in system design?

# Direct answer

**Scalability** is a system's ability to handle increasing load (users, requests, data, or traffic) by adding resources, while maintaining acceptable performance and reliability.

In simple terms:

> A scalable system continues to work efficiently as demand grows.

---

# Why scalability matters

A system that works for:

- 100 users
- 1,000 requests/day

may completely fail at:

- 10 million users
- 100,000 requests/second

Without scalability, you may see:

- Slow response times
- Database bottlenecks
- Server crashes
- Increased downtime

Good system design anticipates growth and ensures the system can expand smoothly.

---

# Types of scalability

## 1. Vertical Scaling (Scale Up)

Increase the capacity of a single machine.

Example:

```text
4 CPU, 16 GB RAM
        ↓
32 CPU, 128 GB RAM
```

### Advantages

- Simple to implement
- No application changes required

### Disadvantages

- Hardware limits exist
- Expensive
- Single point of failure remains

Example:

- Upgrading a database server from 16 GB RAM to 128 GB RAM.

---

## 2. Horizontal Scaling (Scale Out)

Add more machines and distribute traffic among them.

Example:

```text
1 Server
   ↓
10 Servers
   ↓
100 Servers
```

Architecture:

```text
Users
   |
Load Balancer
   |
-------------------
|    |    |    |
S1   S2   S3   S4
```

### Advantages

- Nearly unlimited growth
- Better fault tolerance
- High availability

### Disadvantages

- More complex architecture
- Requires distributed system design

Most internet-scale systems use horizontal scaling.

---

# What can be scaled?

## Application Layer

Add more application servers.

```text
Load Balancer
      |
----------------
|      |       |
App1  App2   App3
```

This increases request handling capacity.

---

## Database Layer

As traffic grows, databases often become bottlenecks.

Techniques include:

### Read Replicas

```text
         Primary
         /     \
Replica1      Replica2
```

Reads are distributed across replicas.

---

### Sharding

Split data across multiple databases.

```text
Users A-M → Shard 1
Users N-Z → Shard 2
```

This distributes storage and traffic.

---

## Cache Layer

Instead of repeatedly hitting the database:

```text
User
  |
Cache
  |
Database
```

Frequently accessed data is served from cache.

This dramatically improves scalability.

---

# Scalability dimensions

A system may need to scale in multiple ways:

| Dimension        | Example                |
| ---------------- | ---------------------- |
| Users            | 1M → 100M users        |
| Requests         | 1K → 100K requests/sec |
| Storage          | GB → PB of data        |
| Bandwidth        | MB/s → GB/s            |
| Geographic Reach | One region → Global    |

---

# Example: URL Shortener

### Initial Design

```text
User
  |
Application
  |
Database
```

Works for thousands of requests.

---

### Scaled Design

```text
Users
   |
Load Balancer
   |
Application Servers
   |
Redis Cache
   |
Database Cluster
   |
Replicas/Shards
```

Now the system can support:

- Millions of users
- Billions of URLs
- High request rates

---

# Scalability vs Performance

These concepts are related but different.

| Performance                 | Scalability                       |
| --------------------------- | --------------------------------- |
| How fast the system is now  | How well it handles future growth |
| Current latency             | Growth capacity                   |
| Single-machine optimization | Distributed growth                |

Example:

- A server responding in 10 ms is high-performance.
- If it crashes when traffic doubles, it is not scalable.

---

# Scalability trade-offs

Scaling often introduces complexity.

| Benefit             | Cost                             |
| ------------------- | -------------------------------- |
| More throughput     | More infrastructure              |
| Better availability | More coordination                |
| Lower latency       | Additional caching               |
| More capacity       | Increased operational complexity |

Examples:

- Sharding improves scale but complicates queries.
- Replication improves availability but introduces consistency challenges.
- Caching improves performance but can create stale data issues.

---

# Common scalability techniques

- Load balancing
- Caching (Redis, Memcached)
- Database replication
- Database sharding
- CDN usage
- Asynchronous processing with queues
- Microservices
- Stateless application servers
- Distributed storage systems

---

# Interview-ready summary

> Scalability is the ability of a system to handle increasing load by adding resources without significantly degrading performance or reliability. It can be achieved through vertical scaling (adding more power to a single machine) or horizontal scaling (adding more machines). In large-scale systems, scalability is typically achieved using load balancing, caching, replication, sharding, and distributed architectures to support growing users, traffic, and data volumes.

## Question 5. What is the difference between vertical scaling and horizontal scaling?

# Direct answer

**Vertical scaling (Scale Up)** means increasing the resources of a **single machine** (more CPU, RAM, storage).

**Horizontal scaling (Scale Out)** means adding **more machines** and distributing load across them.

Example:

- Vertical Scaling: Upgrade a server from **8 CPUs → 32 CPUs**.
- Horizontal Scaling: Increase from **1 server → 10 servers** behind a load balancer.

---

# Comparison

| Aspect                        | Vertical Scaling (Scale Up)     | Horizontal Scaling (Scale Out) |
| ----------------------------- | ------------------------------- | ------------------------------ |
| Approach                      | Add resources to one machine    | Add more machines              |
| Example                       | 16 GB RAM → 128 GB RAM          | 1 server → 10 servers          |
| Complexity                    | Simpler                         | More complex                   |
| Scalability Limit             | Hardware limit exists           | Much higher limit              |
| Availability                  | Single point of failure remains | Better fault tolerance         |
| Cost                          | Expensive at high end           | Often more cost-effective      |
| Distributed System Challenges | Minimal                         | Significant                    |
| Typical Usage                 | Small/medium systems            | Large-scale internet systems   |

---

# Vertical Scaling

### Architecture

```text
Before

Application
    |
Database Server
(8 CPU, 32 GB RAM)
```

```text
After

Application
    |
Database Server
(64 CPU, 512 GB RAM)
```

We are still using **one machine**, just a larger one.

---

## Advantages

### Simplicity

No major architectural changes.

### No distributed-system complexity

Avoids:

- Sharding
- Replication coordination
- Distributed transactions

### Easier operations

One server is easier to manage than many servers.

---

## Disadvantages

### Hardware limits

A machine cannot grow forever.

```text
16 GB RAM
  ↓
64 GB RAM
  ↓
256 GB RAM
  ↓
Eventually hits a limit
```

### Single point of failure

If the machine crashes:

```text
Server Down
    ↓
Service Down
```

### Expensive

The cost of high-end hardware grows rapidly.

---

# Horizontal Scaling

### Architecture

```text
               Load Balancer
                     |
      --------------------------------
      |              |              |
   Server 1       Server 2       Server 3
```

Instead of making one machine bigger, we add more machines.

---

## Advantages

### Nearly unlimited growth

```text
1 Server
   ↓
10 Servers
   ↓
100 Servers
   ↓
1000 Servers
```

This is how large internet companies scale.

### Better availability

If one server fails:

```text
Server 1 ❌

Server 2 ✅
Server 3 ✅
```

Traffic continues to flow.

### Fault tolerance

Failures affect only part of the system.

---

## Disadvantages

### Increased complexity

Now we need:

- Load balancers
- Service discovery
- Distributed caches
- Replication
- Sharding

### Consistency challenges

Multiple nodes may have different views of data.

Example:

```text
User updates profile
        ↓
Replica synchronization pending
```

Some users may temporarily see stale data.

---

# Database Example

## Vertical Scaling Database

```text
Application
     |
 Large Database Server
```

Increase CPU, RAM, SSD capacity.

Works well initially.

---

## Horizontal Scaling Database

```text
           Router
             |
    -------------------
    |        |        |
Shard1   Shard2   Shard3
```

Data is partitioned across multiple databases.

This supports much larger datasets and traffic.

---

# Real-world examples

### Vertical Scaling

Typical for:

- Early-stage startups
- Internal tools
- Small business applications

Example:

- A single PostgreSQL server upgraded as traffic grows.

---

### Horizontal Scaling

Typical for:

- Social networks
- Search engines
- Streaming platforms
- Large e-commerce systems

Examples:

- Google
- Netflix
- Amazon

These companies run thousands of servers and rely heavily on horizontal scaling.

---

# When to choose which?

### Choose Vertical Scaling when:

- System is small.
- Traffic is moderate.
- Simplicity is important.
- Fast growth is not expected.

---

### Choose Horizontal Scaling when:

- Traffic is growing rapidly.
- High availability is required.
- Fault tolerance is important.
- Internet-scale growth is expected.

---

# Common interview answer

A strong interview response is:

> Vertical scaling increases the capacity of a single machine by adding more CPU, memory, or storage, while horizontal scaling increases capacity by adding more machines and distributing traffic among them. Vertical scaling is simpler but limited by hardware and retains a single point of failure. Horizontal scaling is more complex because it introduces distributed-system challenges, but it provides significantly better scalability, availability, and fault tolerance. Most large-scale systems prefer horizontal scaling.

---

# Interview-ready summary

> Vertical scaling (scale up) means upgrading a single server's resources, while horizontal scaling (scale out) means adding more servers and distributing load across them. Vertical scaling is simpler but has hardware limits and a single point of failure. Horizontal scaling is more complex but offers better scalability, availability, and fault tolerance, making it the preferred approach for large distributed systems.

## Question 6. What are the pros and cons of vertical scaling?

## Question 7. What are the pros and cons of horizontal scaling?

## Question 8. Explain latency vs throughput

## Question 9. What is availability? How is it measured?

## Question 10. What is consistency in distributed systems?

## Question 11. What is reliability in a system?

## Question 12. Explain the CAP theorem in simple terms

## Question 13. What is partition tolerance in the CAP theorem?

## Question 14. What is load balancing and why is it needed?

## Question 15. What are caching strategies in system design?

## Question 16. What are indexes in databases and why are they important?

## Question 17. What is sharding?

## Question 18. What is replication?

## Question 19. Difference between synchronous and asynchronous replication

## Question 20. What is eventual consistency?
