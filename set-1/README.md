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

## Question 3. What are functional and non-functional requirements in system design?

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
