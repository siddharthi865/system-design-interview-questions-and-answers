# Set 2

| S.No. | Question                                                                                                                                             |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you approach designing a large-scale system from scratch?](#question-1-how-do-you-approach-designing-a-large-scale-system-from-scratch)      |
| 2.    | [What are the major components of a distributed system?](#question-2-what-are-the-major-components-of-a-distributed-system)                          |
| 3.    | [Explain client-server architecture](#question-3-explain-client-server-architecture)                                                                 |
| 4.    | [Explain microservices architecture vs monolithic architecture](#question-4-explain-microservices-architecture-vs-monolithic-architecture)           |
| 5.    | [What is service discovery and how does it work?](#question-5-what-is-service-discovery-and-how-does-it-work)                                        |
| 6.    | [How do you design an API rate limiter?](#question-6-how-do-you-design-an-api-rate-limiter)                                                          |
| 7.    | [What is CDN (Content Delivery Network) and how does it work?](#question-7-what-is-cdn-content-delivery-network-and-how-does-it-work)                |
| 8.    | [What is a message queue? Give examples](#question-8-what-is-a-message-queue-give-examples)                                                          |
| 9.    | [What is a publish-subscribe model?](#question-9-what-is-a-publish-subscribe-model)                                                                  |
| 10.   | [How would you design a notification system?](#question-10-how-would-you-design-a-notification-system)                                               |
| 11.   | [How do you design a URL shortener like bit.ly?](#question-11-how-do-you-design-a-url-shortener-like-bitly)                                          |
| 12.   | [How do you design a file storage system like Google Drive?](#question-12-how-do-you-design-a-file-storage-system-like-google-drive)                 |
| 13.   | [How do you design a video streaming service like YouTube/Netflix?](#question-13-how-do-you-design-a-video-streaming-service-like-youtubenetflix)    |
| 14.   | [How do you design a ride-sharing system like Uber?](#question-14-how-do-you-design-a-ride-sharing-system-like-uber)                                 |
| 15.   | [How do you design a food delivery system like Zomato/Swiggy?](#question-15-how-do-you-design-a-food-delivery-system-like-zomatoswiggy)              |
| 16.   | [How do you design an e-commerce website like Amazon?](#question-16-how-do-you-design-an-e-commerce-website-like-amazon)                             |
| 17.   | [How do you design a chat application like WhatsApp?](#question-17-how-do-you-design-a-chat-application-like-whatsapp)                               |
| 18.   | [How do you design a payment gateway like PayPal?](#question-18-how-do-you-design-a-payment-gateway-like-paypal)                                     |
| 19.   | [How do you design an email service like Gmail?](#question-19-how-do-you-design-an-email-service-like-gmail)                                         |
| 20.   | [How do you design an online collaborative editor like Google Docs?](#question-20-how-do-you-design-an-online-collaborative-editor-like-google-docs) |

## Question 1. How do you approach designing a large-scale system from scratch?

## Direct answer

A structured way to approach designing a large-scale system is:

1. **Clarify requirements**
2. **Estimate scale**
3. **Design a high-level architecture**
4. **Define APIs and data model**
5. **Identify bottlenecks and scale each component**
6. **Address reliability, availability, and consistency**
7. **Add security and observability**
8. **Discuss trade-offs and future improvements**

In a system design interview, the process is often more important than the final architecture.

---

## Requirements / Problem Framing

Before drawing any architecture, understand what you're building.

### Functional requirements

What should the system do?

Examples:

- Users can upload photos
- Users can send messages
- Users can search content
- Users can place orders

### Non-functional requirements

How should the system behave?

Examples:

- 99.99% availability
- <200ms response latency
- Support 100M users
- Strong consistency for payments
- High durability for stored data

### Clarify assumptions

Ask questions like:

- Expected traffic?
- Read-heavy or write-heavy?
- Global or regional users?
- Real-time requirements?
- Data retention period?

---

## Capacity / Sizing

Before choosing technologies, estimate scale.

Example:

Assume:

- 10 million DAU
- 100 requests/user/day

```
Total requests/day = 1B
Average QPS ≈ 11,500
Peak QPS ≈ 5–10x average
Peak ≈ 100,000 QPS
```

Estimate:

- Storage growth
- Network bandwidth
- Cache size
- Database throughput

These estimates drive architectural decisions.

---

## High-Level Architecture

Start simple.

```text
Clients
   |
Load Balancer
   |
Application Servers
   |
+---------+---------+
|                   |
Cache           Database
|
CDN (for static content)
```

### Core components

**Load Balancer**

- Distributes traffic
- Removes unhealthy servers

**Application Servers**

- Stateless whenever possible
- Easy horizontal scaling

**Cache**

- Reduce database load
- Improve latency

**Database**

- Persistent storage
- Source of truth

**CDN**

- Serve static assets closer to users

---

## Deep Design Considerations

### 1. Scalability

When traffic grows:

#### Scale application layer

```text
LB
 |
+----+----+----+
App1 App2 App3
```

Add more servers horizontally.

#### Scale database

Techniques:

- Read replicas
- Sharding
- Partitioning

```text
Users A-M -> Shard 1
Users N-Z -> Shard 2
```

---

### 2. Caching Strategy

Frequently accessed data should not hit the database every time.

```text
Request
   |
 Cache
   |
Database
```

Common pattern:

```text
Cache Aside
```

1. Check cache
2. Cache miss → DB
3. Store result in cache

Benefits:

- Lower latency
- Lower DB load

---

### 3. Availability

Avoid single points of failure.

```text
        LB
      /    \
   App1   App2
      \    /
     Database Cluster
```

Use:

- Multiple application instances
- Database replication
- Multi-AZ deployment

---

### 4. Reliability

Ensure data is not lost.

Methods:

- Replication
- Backups
- Durable storage
- Disaster recovery plans

Example:

```text
Primary DB
    |
Replicas
```

---

### 5. Consistency

Choose based on business needs.

| System      | Consistency Requirement |
| ----------- | ----------------------- |
| Banking     | Strong consistency      |
| Inventory   | Strong or near-strong   |
| Social feed | Eventual consistency    |
| Analytics   | Eventual consistency    |

Not every component needs strong consistency.

---

### 6. Asynchronous Processing

For expensive operations:

```text
User Request
     |
 Application
     |
 Message Queue
     |
 Background Workers
```

Examples:

- Email sending
- Notifications
- Video processing
- Analytics

Benefits:

- Faster user response
- Better fault tolerance

---

## Security / Observability

### Security

- Authentication
- Authorization
- TLS encryption
- Rate limiting
- Input validation
- Secrets management

### Observability

Monitor:

**Metrics**

- QPS
- Latency
- Error rate
- CPU/Memory

**Logs**

- Request logs
- Error logs

**Tracing**

- End-to-end request tracking

Tools commonly used:

- Prometheus
- Grafana
- ELK
- OpenTelemetry

---

## Trade-offs

Every design choice has trade-offs.

| Choice             | Benefit           | Cost                   |
| ------------------ | ----------------- | ---------------------- |
| Cache              | Low latency       | Stale data             |
| Replication        | High availability | Consistency complexity |
| Sharding           | Massive scale     | Operational complexity |
| Async processing   | Higher throughput | Eventual consistency   |
| Strong consistency | Correctness       | Higher latency         |

A strong system design discussion is often about explaining these trade-offs.

---

## Interview-Ready Summary

When designing a large-scale system from scratch, I first clarify functional and non-functional requirements, then estimate scale to understand traffic and storage needs. Next, I design a simple high-level architecture with load balancers, stateless services, caches, databases, and CDNs. After that, I focus on scaling, reliability, availability, consistency, caching, replication, and asynchronous processing. Finally, I discuss security, observability, and the trade-offs behind key architectural decisions. This structured approach ensures the design can evolve from a simple system to one that supports millions of users.

## Question 2. What are the major components of a distributed system?

## Question 3. Explain client-server architecture

## Question 4. Explain microservices architecture vs monolithic architecture

## Question 5. What is service discovery and how does it work?

## Question 6. How do you design an API rate limiter?

## Question 7. What is CDN (Content Delivery Network) and how does it work?

## Question 8. What is a message queue? Give examples

## Question 9. What is a publish-subscribe model?

## Question 10. How would you design a notification system?

## Question 11. How do you design a URL shortener like bit.ly?

## Question 12. How do you design a file storage system like Google Drive?

## Question 13. How do you design a video streaming service like YouTube/Netflix?

## Question 14. How do you design a ride-sharing system like Uber?

## Question 15. How do you design a food delivery system like Zomato/Swiggy?

## Question 16. How do you design an e-commerce website like Amazon?

## Question 17. How do you design a chat application like WhatsApp?

## Question 18. How do you design a payment gateway like PayPal?

## Question 19. How do you design an email service like Gmail?

## Question 20. How do you design an online collaborative editor like Google Docs?
