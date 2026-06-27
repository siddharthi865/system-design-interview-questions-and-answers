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

# Design a Cricket/Football Score Tracking App

## Direct Answer

A score tracking app (like Cricbuzz, ESPN, SofaScore, or Flashscore) provides **live scores, commentary, match statistics, scorecards, notifications, and historical data** to millions of users with very low latency. The primary challenge is handling **high fan-out (one match → millions of viewers)** while ensuring live updates are delivered in near real time.

The architecture is primarily **event-driven**, with **WebSockets**, **publish-subscribe messaging**, **caching**, and **CDNs** to efficiently distribute live match updates.

---

# 1. Requirements / Problem Framing

## Functional Requirements

### User Features

- View live scores
- Ball-by-ball (cricket) or minute-by-minute (football) updates
- Match commentary
- Live scorecard
- Team/player statistics
- Fixtures and schedules
- Points table/league standings
- Push notifications
- Search teams, players, tournaments

### Admin/Data Operator Features

- Feed live match events
- Correct scoring mistakes
- Update player statistics
- Manage tournaments

---

## Non-functional Requirements

- Millions of concurrent users
- Score updates within 1–2 seconds
- High availability (99.99%)
- Low latency (<100 ms for score updates)
- Scalable during major tournaments (e.g., IPL, FIFA World Cup)
- Fault tolerant
- Eventually consistent for non-critical statistics

---

# 2. High-Level Architecture

```text
                Users (Mobile/Web)
                       |
                 CDN + Load Balancer
                       |
                  API Gateway
                       |
 ----------------------------------------------------
 |            |            |           |             |
 Match      Score      Notification   Stats      User
 Service    Service       Service     Service    Service
      \         |            |           /
       \        |            |          /
        ---------- Event Bus (Kafka) ----------
                     |
             Live Data Processor
                     |
          Score Feed / Data Providers
                     |
               Operator Dashboard

Storage
--------
Redis
Match DB
Stats DB
Time-Series DB
Search Index
```

---

# 3. Core Components

## Match Service

Stores

- Match metadata
- Teams
- Venue
- Toss
- Schedule
- Match status

Example

```text
Match
--------
match_id
team1
team2
venue
start_time
status
```

---

## Live Score Service

Responsible for

- Current score
- Overs/minutes
- Wickets/goals
- Match state

Example

```text
MatchState

Runs: 182/4

Overs: 18.3

Target: 201
```

For football

```text
Score

Liverpool 2

Arsenal 1

Minute 84
```

---

## Commentary Service

Stores

- Ball-by-ball commentary
- Match events

Example

```text
18.4

Four!

Excellent cover drive.
```

or

```text
82'

Goal!

Messi scores.
```

---

## Statistics Service

Maintains

- Batting statistics
- Bowling statistics
- Player ratings
- Possession
- Passing accuracy
- Heatmaps

Can be computed asynchronously.

---

## Notification Service

Examples

- Match started
- Wicket
- Goal
- Half time
- Full time
- Milestones

Notifications are generated from match events.

---

# 4. Data Flow

```text
Official Feed

↓

Data Ingestion

↓

Kafka

↓

Score Processor

↓

Redis

↓

WebSocket Gateway

↓

Millions of Users
```

Every event is published once and consumed by multiple services.

---

# 5. Database Design

## Match Table

```text
match_id
team1
team2
venue
status
start_time
```

---

## Event Table

```text
event_id
match_id
timestamp
event_type
payload
```

Example payload

```json
{
  "over": 18.3,
  "runs": 6,
  "batsman": "Virat"
}
```

or

```json
{
  "minute": 84,
  "goal_by": "Messi"
}
```

---

## Score Snapshot

```text
match_id

current_score

overs

wickets

last_updated
```

Redis stores this for fast access.

---

# 6. Real-Time Update Strategy

Polling every second does not scale well.

### Better Solution: WebSockets

```text
Client

↓

WebSocket

↓

Live Update Server

↓

Score Updates
```

When a wicket falls

```text
Score Updated

↓

Publish Event

↓

All Connected Clients Receive Update
```

Advantages

- Low latency
- Reduced network overhead
- Efficient for millions of users

---

# 7. Event-Driven Architecture

Each match event is published once.

Example

```text
Goal Scored

↓

Kafka Topic

↓

Score Service

↓

Commentary Service

↓

Statistics Service

↓

Notification Service

↓

Analytics
```

Each service processes the event independently.

---

# 8. Scaling Strategy

## Stateless API Servers

```text
Load Balancer

↓

API Server 1

API Server 2

API Server 3
```

Horizontal scaling is straightforward.

---

## WebSocket Servers

```text
Users

↓

Load Balancer

↓

WebSocket Cluster
```

Clients remain connected throughout the match.

---

## Redis Cache

Store

- Current score
- Current over/minute
- Live scoreboard
- Match metadata

This avoids frequent database reads.

---

## CDN

Static assets

- Images
- Team logos
- Player photos
- JavaScript
- CSS

Served from a CDN.

---

## Database Scaling

### Read Replicas

Historical data

```text
Primary

↓

Read Replica 1

↓

Read Replica 2
```

Live scores come from Redis rather than the database.

---

# 9. Reliability

- Kafka replication for durable event streams
- Multiple WebSocket servers behind a load balancer
- Redis replication and failover
- Idempotent event processing to handle duplicate events
- Retry and dead-letter queues for failed consumers
- Health checks and auto-scaling during major sporting events

---

# 10. Capacity / Sizing (Example)

Assumptions:

- 10 million concurrent users
- 100 live matches
- 5 score events/second per match (cricket)
- Each event ≈ 500 bytes

### Event Rate

```
100 × 5 = 500 events/sec
```

### Fan-out

```
500 events/sec

↓

10 million users
```

The backend processes only 500 incoming events per second, but must distribute them to millions of WebSocket connections. This fan-out is the primary scalability challenge.

---

# 11. Security / Observability

### Security

- OAuth/JWT authentication for logged-in users
- Rate limiting on APIs
- TLS encryption
- Secure admin dashboard with role-based access
- Audit logging for score corrections

### Observability

Track

- Event processing latency
- WebSocket connection count
- Message delivery latency
- Cache hit ratio
- Kafka consumer lag
- API latency
- Error rates
- Notification delivery success

Use centralized logging, metrics, distributed tracing, and alerts.

---

# 12. Trade-offs

| Design Choice                    | Pros                            | Cons                                                                           |
| -------------------------------- | ------------------------------- | ------------------------------------------------------------------------------ |
| WebSockets                       | Real-time, low latency          | Persistent connections require more infrastructure                             |
| Polling                          | Simple to implement             | High bandwidth and server load                                                 |
| Redis for live scores            | Extremely fast reads            | Data must be kept in sync with the source of truth                             |
| Event-driven architecture        | Decoupled, scalable, extensible | More operational complexity                                                    |
| Relational DB for match metadata | Strong consistency              | Less suitable for high-frequency event ingestion than specialized event stores |

---

# Interview-Ready Summary

> "I would build the score tracking app using an event-driven architecture. Live score events from official data providers are ingested into Kafka, processed by the Score Service, cached in Redis, and pushed to clients through WebSocket servers for near real-time updates. Match metadata and historical data are stored in databases, while Redis serves current match state to minimize latency. Independent services consume the same event stream for commentary, notifications, statistics, and analytics, allowing the system to scale horizontally and handle millions of concurrent users during major sporting events."

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
