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

## Question 5. What is the difference between vertical scaling and horizontal scaling?

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
