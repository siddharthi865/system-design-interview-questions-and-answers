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

## Direct answer

A distributed system is typically composed of multiple independent machines that work together to appear as a single system to users. The major components are:

1. **Clients**
2. **Load Balancers**
3. **Application Services**
4. **Databases**
5. **Caches**
6. **Message Queues / Event Streams**
7. **Storage Systems**
8. **Service Discovery & Coordination**
9. **Monitoring & Logging Systems**
10. **Security Components**
11. **Network Infrastructure**

Not every distributed system needs all of them, but these are the building blocks used in most large-scale architectures.

---

## High-Level View

```text
                Clients
                   |
            Load Balancer
                   |
        +----------+----------+
        |                     |
   Service A            Service B
        |                     |
   +----+----+           +----+----+
   |         |           |         |
 Cache     Database    Queue     Storage
              |
          Replicas

        Monitoring / Logging
               Security
```

---

## 1. Clients

Clients are the entry point into the system.

Examples:

- Web browsers
- Mobile apps
- IoT devices
- Other services

Responsibilities:

- Send requests
- Display results
- Handle user interactions

Example:
A mobile app sends a request to fetch a user's timeline.

---

## 2. Load Balancer

Distributes incoming traffic across multiple servers.

Responsibilities:

- Traffic distribution
- Health checking
- Failover
- SSL termination

Example:

```text
          LB
        /  |  \
     App1 App2 App3
```

Benefits:

- Improves availability
- Prevents server overload
- Enables horizontal scaling

---

## 3. Application Services

Contain the business logic.

Examples:

- User Service
- Payment Service
- Order Service
- Notification Service

Responsibilities:

- Process requests
- Validate data
- Communicate with databases and other services

Typically deployed as:

```text
Multiple Stateless Instances
```

so they can scale horizontally.

---

## 4. Databases

Store persistent data.

Examples:

- User profiles
- Orders
- Transactions

Common types:

| Type            | Example Use Case     |
| --------------- | -------------------- |
| Relational DB   | Payments, Orders     |
| Key-Value Store | Sessions             |
| Document DB     | User content         |
| Graph DB        | Social relationships |

Responsibilities:

- Durability
- Query processing
- Data consistency

---

## 5. Cache

Stores frequently accessed data in memory.

Examples:

- User profiles
- Product information
- Session data

```text
Request
   |
 Cache
   |
Database
```

Benefits:

- Lower latency
- Reduced database load

Popular choices:

- Redis
- Memcached

---

## 6. Message Queue / Event Streaming System

Enables asynchronous communication.

Examples:

- Kafka
- RabbitMQ
- Amazon SQS

Architecture:

```text
Producer
   |
 Queue
   |
Consumer
```

Use cases:

- Email delivery
- Notifications
- Order processing
- Analytics pipelines

Benefits:

- Decoupling
- Better scalability
- Improved fault tolerance

---

## 7. Distributed Storage

Stores large files and objects.

Examples:

- Images
- Videos
- Backups
- Logs

Popular systems:

- HDFS
- Object storage services
- Distributed file systems

Responsibilities:

- Durability
- Replication
- Large-scale storage

---

## 8. Service Discovery & Coordination

Helps services find each other dynamically.

Example:

```text
Service A
    |
Service Registry
    |
Service B
```

Responsibilities:

- Service registration
- Health tracking
- Leader election
- Distributed coordination

Examples:

- ZooKeeper
- etcd
- Consul

Especially important in microservice architectures.

---

## 9. Monitoring, Logging, and Tracing

Provides visibility into system health.

### Monitoring

Tracks:

- CPU
- Memory
- QPS
- Latency
- Error rates

### Logging

Stores:

- Request logs
- Error logs
- Audit logs

### Distributed Tracing

Tracks requests across services.

```text
User Request
   |
 Service A
   |
 Service B
   |
 Database
```

Popular tools:

- Prometheus
- Grafana
- ELK Stack
- OpenTelemetry

---

## 10. Security Components

Protect the system.

Examples:

- Authentication services
- Authorization systems
- API gateways
- Secret management

Responsibilities:

- Identity verification
- Access control
- Encryption
- Rate limiting

---

## 11. Network Infrastructure

The communication layer connecting all nodes.

Responsibilities:

- Routing
- DNS
- Secure communication
- Cross-region connectivity

Components:

- DNS
- Reverse proxies
- Firewalls
- Virtual networks

Without reliable networking, a distributed system cannot function.

---

## How They Work Together

Consider an e-commerce request:

```text
User
  |
Load Balancer
  |
Order Service
  |
Cache
  |
Database
  |
Kafka
  |
Notification Service
```

Flow:

1. User places an order.
2. Load balancer routes request.
3. Order service validates request.
4. Cache is checked.
5. Database stores order.
6. Event is published to Kafka.
7. Notification service sends confirmation email.

Each component has a specialized responsibility, making the overall system scalable and resilient.

---

## Interview-Ready Summary

The core components of a distributed system are clients, load balancers, application services, databases, caches, message queues, distributed storage, service discovery systems, monitoring infrastructure, security services, and networking components. Together they enable scalability, availability, fault tolerance, and maintainability by distributing work across multiple machines while presenting a unified system to users.

## Question 3. Explain client-server architecture

## Direct answer

**Client-server architecture** is a distributed computing model where **clients** send requests for services or data, and **servers** process those requests and return responses.

The client is responsible for the user interface and user interactions, while the server is responsible for business logic, data processing, and storage.

---

## Basic Concept

```text
+---------+      Request      +---------+
| Client  | ----------------> | Server  |
|         | <---------------- |         |
+---------+      Response     +---------+
```

Examples:

- A web browser requesting a webpage.
- A mobile app fetching user profiles.
- An ATM querying a banking server.

---

## Components

### 1. Client

The client initiates communication.

Examples:

- Web browser
- Mobile app
- Desktop application

Responsibilities:

- User interface
- Input validation (basic)
- Sending requests
- Displaying responses

Example:

```text
Browser → GET /profile/123
```

---

### 2. Server

The server provides services and resources.

Responsibilities:

- Execute business logic
- Authenticate users
- Access databases
- Generate responses

Example:

```text
GET /profile/123
      |
      v
 Fetch user data
      |
      v
 Return JSON response
```

---

## Request-Response Flow

Consider a user opening Instagram:

```text
User
  |
Mobile App (Client)
  |
HTTPS Request
  |
Instagram Server
  |
Database
  |
Response
  |
Display Feed
```

Step-by-step:

1. User opens app.
2. Client sends request.
3. Server validates request.
4. Server queries database.
5. Server generates response.
6. Client displays data.

---

## Types of Client-Server Architectures

### 1. Two-Tier Architecture

Direct communication between client and server.

```text
Client
   |
Server + Database
```

Example:

- Small desktop applications
- Internal business tools

Pros:

- Simple
- Easy to build

Cons:

- Poor scalability
- Tight coupling

---

### 2. Three-Tier Architecture

Most common web architecture.

```text
Client
   |
Application Server
   |
Database
```

Example:

- E-commerce websites
- Social media platforms

Benefits:

- Better scalability
- Easier maintenance
- Improved security

---

### 3. N-Tier Architecture

Large-scale modern systems.

```text
Client
   |
Load Balancer
   |
API Layer
   |
Microservices
   |
Databases / Caches / Queues
```

Used by:

- Netflix
- Amazon
- Uber
- Facebook

---

## Characteristics

### Centralized Management

Servers control:

- Data
- Authentication
- Business logic

Benefits:

- Easier updates
- Better security

---

### Resource Sharing

Many clients can use the same server.

```text
Client A
Client B
Client C
    |
  Server
```

Example:
Thousands of users accessing the same website.

---

### Request-Response Communication

Most systems follow:

```text
Request → Processing → Response
```

Protocols:

- HTTP/HTTPS
- gRPC
- WebSocket
- TCP

---

## Scalability Challenges

A single server eventually becomes a bottleneck.

### Initial Design

```text
Clients
   |
 Server
```

Problems:

- CPU exhaustion
- Memory exhaustion
- Single point of failure

---

### Scaled Design

```text
Clients
   |
Load Balancer
   |
+----+----+----+
|    |    |    |
S1   S2   S3  S4
```

Benefits:

- Higher throughput
- Better availability
- Fault tolerance

This evolution is common in system design interviews.

---

## Advantages

| Advantage           | Explanation                       |
| ------------------- | --------------------------------- |
| Centralized control | Easier management                 |
| Security            | Data stays on server              |
| Resource sharing    | Many clients use same service     |
| Easier maintenance  | Server updates affect all clients |
| Scalability         | Servers can be replicated         |

---

## Disadvantages

| Disadvantage               | Explanation                             |
| -------------------------- | --------------------------------------- |
| Server bottleneck          | Heavy traffic can overload server       |
| Single point of failure    | One server failure can impact all users |
| Network dependency         | Requires connectivity                   |
| Higher infrastructure cost | Multiple servers needed at scale        |

---

## Real-World Examples

| System   | Client         | Server              |
| -------- | -------------- | ------------------- |
| Gmail    | Browser/App    | Google Mail Servers |
| Netflix  | Mobile/Web App | Netflix Backend     |
| Amazon   | Browser/App    | Amazon Services     |
| WhatsApp | Mobile App     | WhatsApp Servers    |

---

## Client-Server vs Peer-to-Peer (P2P)

| Feature      | Client-Server         | Peer-to-Peer          |
| ------------ | --------------------- | --------------------- |
| Control      | Centralized           | Decentralized         |
| Data Storage | Server                | Multiple peers        |
| Management   | Easier                | More complex          |
| Scalability  | Good with replication | Naturally distributed |
| Example      | Web applications      | BitTorrent            |

### Client-Server

```text
Clients
   |
 Server
```

### P2P

```text
Node A <--> Node B
   ^           |
   |           v
Node D <--> Node C
```

---

## Interview-Ready Summary

Client-server architecture is a model where clients request services and servers process those requests and return responses. Clients handle presentation and user interaction, while servers manage business logic and data storage. Modern large-scale systems extend this model with load balancers, multiple application servers, caches, databases, and queues to achieve scalability, availability, and fault tolerance.

## Question 4. Explain microservices architecture vs monolithic architecture

## Direct answer

The key difference is:

- **Monolithic Architecture**: The entire application is built and deployed as a single unit.
- **Microservices Architecture**: The application is split into multiple independent services, each responsible for a specific business capability and deployable independently.

A monolith is usually simpler to build and operate initially, while microservices provide better scalability, fault isolation, and team autonomy for large, complex systems.

---

## Monolithic Architecture

In a monolithic application, all components run within the same codebase and deployment unit.

### Architecture

```text
                Monolithic Application
+--------------------------------------------------+
| User Module | Order Module | Payment Module      |
| Inventory   | Notification | Authentication      |
+--------------------------------------------------+
                      |
                  Database
```

### Characteristics

- Single codebase
- Single deployment artifact
- Usually one database
- All modules share the same runtime

### Example Request Flow

```text
Client
   |
Monolith
   |
Database
```

A request for placing an order goes through the same application that handles users, payments, notifications, and inventory.

---

## Advantages of Monolithic Architecture

### Simpler Development

- Easy to understand initially
- Fewer moving parts
- Easier local development

### Simpler Deployment

```text
Build → Deploy One Application
```

### Easier Debugging

- Single process
- No network calls between modules

### Better Performance

Internal function calls are faster than network communication.

---

## Challenges of Monolithic Architecture

### Scaling Limitations

If only the payment module is overloaded:

```text
Need to scale entire application
```

even though other modules may be underutilized.

### Large Codebase

Over time:

```text
100k LOC → 1M LOC → 10M LOC
```

Development becomes slower and riskier.

### Slower Deployments

A small change requires redeploying the entire application.

### Reduced Fault Isolation

A memory leak in one module can impact the whole system.

---

## Microservices Architecture

Microservices split the application into independently deployable services.

### Architecture

```text
                Clients
                    |
              API Gateway
                    |
      +------+------+------+------+
      |             |             |
 User Service   Order Service   Payment Service
      |             |             |
      +------+------+------+------+
                    |
              Event Queue
                    |
         Notification Service

Each service owns its database.
```

---

## Characteristics

- Small, focused services
- Independent deployment
- Independent scaling
- Often own their databases
- Communicate via APIs or events

Examples:

- User Service
- Order Service
- Payment Service
- Inventory Service
- Notification Service

---

## Advantages of Microservices

### Independent Scaling

If Payment Service receives heavy traffic:

```text
Payment Service: 20 instances
User Service: 5 instances
```

Scale only what needs scaling.

---

### Independent Deployment

```text
Deploy Payment Service
```

without redeploying the entire system.

---

### Better Fault Isolation

If Notification Service fails:

```text
Orders still work
Payments still work
```

assuming proper isolation.

---

### Team Independence

Different teams can own different services.

Example:

| Team   | Service         |
| ------ | --------------- |
| Team A | User Service    |
| Team B | Payment Service |
| Team C | Order Service   |

This is one reason companies like Amazon and Netflix adopted microservices.

---

## Challenges of Microservices

### Distributed System Complexity

Now every service communicates over the network.

```text
Service A → Service B → Service C
```

New challenges:

- Timeouts
- Retries
- Network failures
- Service discovery

---

### Data Consistency

In a monolith:

```text
Single ACID transaction
```

In microservices:

```text
Order Service DB
Payment Service DB
Inventory Service DB
```

Distributed transactions become difficult.

Patterns such as:

- Saga
- Event-driven architecture
- Outbox pattern

are commonly used.

---

### Observability Complexity

Tracking a request becomes harder:

```text
Client
  |
Gateway
  |
Order Service
  |
Payment Service
  |
Inventory Service
```

Requires:

- Distributed tracing
- Centralized logging
- Correlation IDs

---

### Operational Overhead

Need to manage:

- Service discovery
- API gateways
- CI/CD pipelines
- Monitoring
- Container orchestration

Often with tools like:

- Docker
- Kubernetes
- Service Mesh

---

## Comparison

| Aspect               | Monolith          | Microservices             |
| -------------------- | ----------------- | ------------------------- |
| Codebase             | Single            | Multiple                  |
| Deployment           | Single deployment | Independent deployments   |
| Scaling              | Scale entire app  | Scale individual services |
| Complexity           | Low initially     | High                      |
| Development Speed    | Faster initially  | Better for large teams    |
| Fault Isolation      | Weak              | Strong                    |
| Data Management      | Usually one DB    | Multiple DBs              |
| Operational Overhead | Low               | High                      |
| Team Autonomy        | Limited           | High                      |

---

## When to Choose Monolith

Use a monolith when:

- Startup or early-stage product
- Small engineering team
- Requirements are evolving rapidly
- Low to moderate traffic
- Fast development is more important than scalability

Example:

- MVP
- Internal tools
- Small SaaS application

---

## When to Choose Microservices

Use microservices when:

- Large engineering organization
- Multiple teams working independently
- Different scaling requirements
- Very high traffic
- Need independent deployments

Example:

- E-commerce platforms
- Ride-sharing systems
- Streaming platforms
- Large fintech systems

---

## Trade-off

A common mistake is starting with microservices too early.

Many successful companies began with a monolith and moved to microservices only after:

- Traffic increased
- Team size grew
- Deployment bottlenecks appeared

A well-structured monolith is often the right starting point, while microservices become valuable as organizational and scaling complexity grows.

---

## Interview-Ready Summary

A monolithic architecture packages all business functionality into a single deployable application, making development and operations simpler initially. Microservices architecture decomposes the system into independently deployable services, enabling better scalability, fault isolation, and team autonomy. The trade-off is that microservices introduce distributed-system challenges such as service communication, data consistency, observability, and operational complexity. For most products, starting with a monolith and evolving toward microservices when scale and organizational needs justify it is a practical approach.

## Question 5. What is service discovery and how does it work?

## Direct answer

**Service discovery** is a mechanism that allows services in a distributed system to **find and communicate with each other dynamically**, without hardcoding IP addresses or hostnames.

It solves the problem that in modern cloud environments, service instances are constantly being created, destroyed, scaled, or moved, causing their IP addresses to change frequently.

---

## Why Service Discovery is Needed

Imagine a microservices system:

```text
User Service
Order Service
Payment Service
Notification Service
```

Suppose the Order Service needs to call the Payment Service.

### Without Service Discovery

```text
Order Service
     |
     |--> 10.0.1.5:8080
```

Problems:

- Payment Service may restart.
- Kubernetes may create a new instance.
- Auto-scaling may add more instances.
- IP addresses change frequently.

Hardcoding addresses becomes impossible at scale.

---

## Basic Idea

Instead of knowing actual IPs, services use a **logical service name**.

```text
Order Service
      |
      |--> Payment Service
```

A service discovery system translates:

```text
Payment Service
      ↓
10.0.1.5
10.0.1.8
10.0.1.12
```

and returns available instances.

---

## How It Works

### Step 1: Service Registration

When a service starts:

```text
Payment Service Instance
        |
        v
Service Registry
```

It registers:

- Service name
- IP address
- Port
- Metadata

Example:

```text
Service: Payment-Service
IP: 10.0.1.5
Port: 8080
```

---

### Step 2: Health Checks

The registry continuously verifies:

```text
Is Payment-Service healthy?
```

If an instance becomes unhealthy:

```text
Payment-Service @ 10.0.1.5
      |
      X Removed
```

it is removed from the registry.

---

### Step 3: Service Lookup

When Order Service needs Payment Service:

```text
Order Service
      |
      v
Service Registry
      |
      v
[10.0.1.8, 10.0.1.12]
```

The registry returns available instances.

---

### Step 4: Request Routing

```text
Order Service
      |
      +--> Payment Instance 1
      |
      +--> Payment Instance 2
```

The caller selects an instance and sends the request.

---

## Architecture

```text
                  +------------------+
                  | Service Registry |
                  +------------------+
                     ^            ^
                     |            |
         Register    |            | Lookup
                     |            |
+------------+       |      +------------+
| Payment S1 |-------+      | Order Svc  |
+------------+              +------------+

+------------+
| Payment S2 |
+------------+
      |
      +---- Register
```

---

## Types of Service Discovery

### 1. Client-Side Discovery

The client queries the registry directly.

```text
Order Service
      |
      v
Service Registry
      |
      v
Payment Instances
      |
      v
Payment Service
```

Examples:

- Netflix Eureka
- Consul

#### Advantages

- Less infrastructure
- Better control over load balancing

#### Disadvantages

- Discovery logic must exist in every client

---

### 2. Server-Side Discovery

The client calls a load balancer or proxy.

```text
Order Service
      |
      v
Load Balancer
      |
      v
Payment Instances
```

The load balancer handles service discovery.

Examples:

- Kubernetes Services
- AWS Elastic Load Balancer

#### Advantages

- Simpler clients
- Centralized routing

#### Disadvantages

- Additional infrastructure component

---

## Service Discovery in Kubernetes

This is one of the most common interview examples.

Suppose there are three Payment Service pods:

```text
Payment Pod 1
Payment Pod 2
Payment Pod 3
```

Kubernetes creates a Service:

```text
payment-service
```

Order Service simply calls:

```text
http://payment-service
```

Kubernetes DNS resolves:

```text
payment-service
      ↓
Available Pods
```

and load-balances requests automatically.

---

## Popular Service Discovery Systems

| Tool             | Usage                               |
| ---------------- | ----------------------------------- |
| Consul           | Service discovery and health checks |
| etcd             | Configuration and discovery         |
| Apache ZooKeeper | Coordination and leader election    |
| Netflix Eureka   | Service registration and discovery  |
| Kubernetes       | Built-in DNS-based discovery        |

---

## Related Concepts

### Service Registry

Stores service information.

Example:

```text
Payment Service -> 10.0.1.5
Order Service   -> 10.0.2.3
User Service    -> 10.0.3.7
```

---

### Health Checks

Ensures only healthy instances receive traffic.

```text
Healthy   -> Available
Unhealthy -> Removed
```

---

### Load Balancing

Often works together with service discovery.

```text
Order Service
      |
      v
Payment Service
      |
 +----+----+
 |         |
P1        P2
```

Requests are distributed across instances.

---

## Interview Perspective

When discussing microservices, mention:

> "Because service instances are dynamic and their IP addresses change frequently, we use service discovery. Services register themselves with a registry, periodically send health information, and other services discover healthy instances through the registry. Service discovery can be client-side (Eureka, Consul) or server-side (Kubernetes Services, load balancers)."

This demonstrates understanding of a key distributed systems challenge.

---

## Interview-Ready Summary

Service discovery is a mechanism that enables services to dynamically locate and communicate with each other in a distributed system. Services register their network locations with a service registry, health checks ensure only healthy instances are available, and clients or load balancers query the registry to find service instances. It eliminates hardcoded addresses and is essential for scalable microservices environments where instances are frequently created, destroyed, or moved.

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
