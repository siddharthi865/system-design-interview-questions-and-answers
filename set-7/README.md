# Set 7

| S.No. | Question                                                                                                                                       |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you design an online car rental system (like Zipcar/Turo)?](#question-1-how-do-you-design-an-online-car-rental-system-like-zipcarturo) |
| 2.    | [How do you design a cricket/football score tracking app?](#question-2-how-do-you-design-a-cricketfootball-score-tracking-app)                 |
| 3.    | [How do you design a calendar booking system?](#question-3-how-do-you-design-a-calendar-booking-system)                                        |
| 4.    | [How do you design a document versioning system?](#question-4-how-do-you-design-a-document-versioning-system)                                  |
| 5.    | [How do you design a banking transaction system?](#question-5-how-do-you-design-a-banking-transaction-system)                                  |
| 6.    | [How do you design a hospital management system?](#question-6-how-do-you-design-a-hospital-management-system)                                  |
| 7.    | [How do you design a warehouse inventory management system?](#question-7-how-do-you-design-a-warehouse-inventory-management-system)            |
| 8.    | [How do you design a railway reservation system?](#question-8-how-do-you-design-a-railway-reservation-system)                                  |
| 9.    | [How do you design an airline ticketing system?](#question-9-how-do-you-design-an-airline-ticketing-system)                                    |
| 10.   | [How do you design an online quiz platform with leaderboards?](#question-10-how-do-you-design-an-online-quiz-platform-with-leaderboards)       |
| 11.   | [What is write-ahead logging (WAL) in databases?](#question-11-what-is-write-ahead-logging-wal-in-databases)                                   |
| 12.   | [What is a B-Tree vs B+ Tree in indexing?](#question-12-what-is-a-b-tree-vs-b-tree-in-indexing)                                                |
| 13.   | [What are Bloom filters and how are they used in databases?](#question-13-what-are-bloom-filters-and-how-are-they-used-in-databases)           |
| 14.   | [What is a materialized view?](#question-14-what-is-a-materialized-view)                                                                       |
| 15.   | [What is query optimization in databases?](#question-15-what-is-query-optimization-in-databases)                                               |
| 16.   | [What are database deadlocks and how to prevent them?](#question-16-what-are-database-deadlocks-and-how-to-prevent-them)                       |
| 17.   | [How do you implement pagination efficiently at scale?](#question-17-how-do-you-implement-pagination-efficiently-at-scale)                     |
| 18.   | [What is hot partitioning in databases?](#question-18-what-is-hot-partitioning-in-databases)                                                   |
| 19.   | [What is gossip protocol in distributed databases?](#question-19-what-is-gossip-protocol-in-distributed-databases)                             |
| 20.   | [How do you implement multi-master replication?](#question-20-how-do-you-implement-multi-master-replication)                                   |

## Question 1. How do you design an online car rental system (like Zipcar/Turo)?

# Design an Online Car Rental System (like Zipcar/Turo)

## Direct answer

An online car rental system allows users to search for available cars, book them for a specific time period, make payments, and pick up/return the vehicle. At scale, the system must handle real-time inventory availability, prevent double bookings, process payments reliably, manage pricing, and support millions of users.

The key challenge is maintaining **consistent vehicle availability** while providing a responsive search experience.

---

# 1. Requirements / Problem Framing

## Functional Requirements

### Customer Features

- User registration and login
- Search cars by location, date, price, type
- View vehicle details
- Check availability
- Book a vehicle
- Online payment
- Cancel booking
- View booking history
- Rate vehicle/host

### Owner Features (Turo-like)

- Register vehicle
- Upload photos
- Set pricing
- Set availability calendar
- Accept/reject bookings
- Track earnings

### Admin Features

- Manage users
- Manage vehicles
- Fraud detection
- Customer support
- Analytics

---

## Non-functional Requirements

- High availability
- Low search latency (<200 ms)
- No double booking
- Strong consistency for reservations
- Scalable to millions of users
- Fault tolerant
- Secure payment processing

---

# 2. High-Level Architecture

```
                 Users
                    |
              Load Balancer
                    |
             API Gateway
                    |
     -------------------------------------
     |       |         |        |         |
 User   Search   Booking   Payment   Vehicle
Service Service  Service   Service   Service
     |       |         |        |         |
     --------------------------------------
                    |
             Event Bus (Kafka)
                    |
      -------------------------------
      |             |               |
 Notification   Analytics      Recommendation
      |
 Email/SMS/Push

Databases
-----------
User DB
Vehicle DB
Booking DB
Payment DB
Search Index
Redis Cache
```

---

# 3. Core Components

## User Service

Responsible for

- Authentication
- User profiles
- Driver verification
- Booking history

Database

```
Users
------
user_id
name
license_number
rating
verification_status
```

---

## Vehicle Service

Stores

- Vehicle information
- Images
- Features
- Owner information
- Current status

```
Vehicle
---------
vehicle_id
owner_id
model
brand
year
type
location
price_per_day
status
```

---

## Search Service

Search filters

- Location
- Pickup time
- Drop time
- Car type
- Price
- Ratings

Uses

- Elasticsearch/OpenSearch
- Redis

Search should not hit the primary database every time.

---

## Booking Service

This is the heart of the system.

Responsibilities

- Create booking
- Prevent double booking
- Maintain reservation state

Booking states

```
Requested

↓

Reserved

↓

Paid

↓

Active Rental

↓

Completed
```

or

```
Cancelled
Expired
```

---

## Payment Service

Responsibilities

- Payment authorization
- Capture payment
- Refund
- Invoice

Uses external payment gateways.

Payment status

```
Pending

Authorized

Captured

Refunded

Failed
```

---

## Notification Service

Sends

- Booking confirmation
- Pickup reminder
- Return reminder
- Cancellation alerts

Implemented asynchronously.

---

# 4. Database Design

## Vehicle Table

```
vehicle_id
owner_id
location_id
model
brand
year
status
price
```

---

## Booking Table

```
booking_id
user_id
vehicle_id
start_time
end_time
status
payment_status
created_at
```

Indexes

```
vehicle_id

start_time

end_time

status
```

---

## Availability Table

```
vehicle_id
date
available
```

or maintain reservation intervals.

---

# 5. Booking Flow

```
User searches

↓

Search Service

↓

Available cars

↓

User selects car

↓

Booking Service

↓

Lock vehicle

↓

Payment

↓

Booking confirmed

↓

Notification sent
```

---

# 6. Preventing Double Booking

This is one of the most important interview topics.

Suppose

```
Car A

Available

10:00–12:00
```

Two users simultaneously book it.

Without protection

```
User A
creates booking

User B
creates booking

Result:
Double booking
```

---

## Solution 1: Database Transaction + Row Lock

```
BEGIN

SELECT ...

FOR UPDATE

Check availability

Insert booking

COMMIT
```

Pros

- Simple
- Strong consistency

Cons

- Doesn't scale well under heavy contention.

---

## Solution 2: Distributed Lock

Using

- Redis
- ZooKeeper

```
Acquire Lock(vehicle_id)

↓

Check availability

↓

Create booking

↓

Release lock
```

---

## Solution 3: Optimistic Locking

Vehicle record

```
version = 5
```

Update

```
UPDATE

WHERE version=5
```

If another request updates first

```
Rows affected = 0

Retry
```

---

## Solution 4 (Preferred)

Maintain booking intervals and use a transaction that checks for overlapping reservations before inserting the new booking. Combine this with database constraints or locking where supported.

---

# 7. Search Optimization

Searching every booking table is expensive.

Instead

Maintain

```
Search Index

↓

Available Vehicles

↓

Location

↓

Price

↓

Ratings
```

Updated asynchronously using events.

Redis caches

```
Popular cities

Popular searches

Vehicle details
```

---

# 8. Scaling Strategy

## Stateless Services

All application servers remain stateless.

```
LB

↓

API Server 1

API Server 2

API Server 3
```

Easy horizontal scaling.

---

## Database Scaling

### Read Replicas

```
Primary

↓

Replica 1

Replica 2
```

Searches go to replicas.

Bookings always go to primary.

---

## Sharding

Partition vehicles by

- Region
- City
- Country

Example

```
US-East

US-West

Europe

Asia
```

Reduces database size per shard.

---

## Caching

Redis stores

- Vehicle details
- User sessions
- Frequently searched locations
- Popular vehicle lists

---

# 9. Event-Driven Architecture

Many operations don't need to block the booking flow.

```
Booking Created

↓

Kafka

↓

Notification

↓

Analytics

↓

Recommendation

↓

Email

↓

Fraud Detection
```

Benefits

- Loose coupling
- Better scalability
- Faster user response

---

# 10. Reliability

- Retry transient failures
- Circuit breakers around payment gateways
- Idempotent booking/payment APIs
- Dead-letter queues for failed events
- Multi-AZ database deployment
- Regular backups and disaster recovery

---

# 11. Security / Observability

### Security

- OAuth/JWT authentication
- Encrypt PII and payment data
- TLS for all APIs
- Role-based authorization
- Rate limiting
- Audit logs
- PCI DSS compliance for payment handling

### Observability

Monitor

- Search latency
- Booking success rate
- Payment failures
- Double-booking attempts
- API latency
- Database slow queries
- Cache hit ratio
- Queue lag

Use centralized logging, metrics, distributed tracing, and alerts.

---

# 12. Trade-offs

| Design Choice              | Pros                                                  | Cons                                     |
| -------------------------- | ----------------------------------------------------- | ---------------------------------------- |
| SQL database for bookings  | ACID transactions, prevents inconsistent reservations | Harder to scale writes                   |
| NoSQL for vehicle catalog  | Flexible schema, easy horizontal scaling              | Weaker transactional guarantees          |
| Cache search results       | Very fast reads                                       | Potentially stale data                   |
| Event-driven notifications | Decoupled and scalable                                | Eventual consistency                     |
| Distributed locking        | Prevents conflicting bookings across instances        | Additional infrastructure and complexity |

---

# Interview-Ready Summary

> "I would design the system as a set of stateless microservices: User, Vehicle, Search, Booking, Payment, and Notification. Vehicle metadata would be indexed in a search engine for fast discovery, while bookings would be stored in a relational database to leverage ACID transactions. The critical challenge is preventing double bookings, which I'd address using transactional overlap checks with locking or optimistic concurrency. Redis would cache popular search data, Kafka would handle asynchronous workflows like notifications and analytics, and the system would scale horizontally with load balancers, read replicas, sharding by region, and comprehensive monitoring."

## Question 2. How do you design a cricket/football score tracking app?

## Question 3. How do you design a calendar booking system?

## Question 4. How do you design a document versioning system?

## Question 5. How do you design a banking transaction system?

## Question 6. How do you design a hospital management system?

## Question 7. How do you design a warehouse inventory management system?

## Question 8. How do you design a railway reservation system?

## Question 9. How do you design an airline ticketing system?

## Question 10. How do you design an online quiz platform with leaderboards?

## Question 11. What is write-ahead logging (WAL) in databases?

## Question 12. What is a B-Tree vs B+ Tree in indexing?

## Question 13. What are Bloom filters and how are they used in databases?

## Question 14. What is a materialized view?

## Question 15. What is query optimization in databases?

## Question 16. What are database deadlocks and how to prevent them?

## Question 17. How do you implement pagination efficiently at scale?

## Question 18. What is hot partitioning in databases?

## Question 19. What is gossip protocol in distributed databases?

## Question 20. How do you implement multi-master replication?
