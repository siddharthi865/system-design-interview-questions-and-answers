# Set 17

| S.No. | Question                                                                                                                                                                  |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you design a music playlist management system?](#question-1-how-do-you-design-a-music-playlist-management-system)                                                 |
| 2.    | [How do you design a product recommendation engine at the class diagram level?](#question-2-how-do-you-design-a-product-recommendation-engine-at-the-class-diagram-level) |
| 3.    | [How do you design a loyalty points system?](#question-3-how-do-you-design-a-loyalty-points-system)                                                                       |
| 4.    | [How do you design a micro-billing system for pay-per-use services?](#question-4-how-do-you-design-a-micro-billing-system-for-pay-per-use-services)                       |
| 5.    | [How do you design a personal budgeting system?](#question-5-how-do-you-design-a-personal-budgeting-system)                                                               |
| 6.    | [How do you design a fitness class reservation system?](#question-6-how-do-you-design-a-fitness-class-reservation-system)                                                 |
| 7.    | [How do you design a drone delivery scheduling system?](#question-7-how-do-you-design-a-drone-delivery-scheduling-system)                                                 |
| 8.    | [How do you design a movie recommendation engine (LLD)?](#question-8-how-do-you-design-a-movie-recommendation-engine-lld)                                                 |
| 9.    | [How do you design a payment settlement service internally?](#question-9-how-do-you-design-a-payment-settlement-service-internally)                                       |
| 10.   | [How do you design an online auction bidding bot?](#question-10-how-do-you-design-an-online-auction-bidding-bot)                                                          |
| 11.   | [What is database shadowing?](#question-11-what-is-database-shadowing)                                                                                                    |
| 12.   | [What are write-heavy database optimization techniques?](#question-12-what-are-write-heavy-database-optimization-techniques)                                              |
| 13.   | [What are hot keys in distributed caches?](#question-13-what-are-hot-keys-in-distributed-caches)                                                                          |
| 14.   | [How do you design a time-to-live (TTL) database?](#question-14-how-do-you-design-a-time-to-live-ttl-database)                                                            |
| 15.   | [What are vector databases?](#question-15-what-are-vector-databases)                                                                                                      |
| 16.   | [How do you handle schema evolution in NoSQL?](#question-16-how-do-you-handle-schema-evolution-in-nosql)                                                                  |
| 17.   | [What is ZooKeeper and how is it used in distributed databases?](#question-17-what-is-zookeeper-and-how-is-it-used-in-distributed-databases)                              |
| 18.   | [What is a two-phase commit protocol?](#question-18-what-is-a-two-phase-commit-protocol)                                                                                  |
| 19.   | [What is a three-phase commit protocol?](#question-19-what-is-a-three-phase-commit-protocol)                                                                              |
| 20.   | [How do you implement idempotent writes in databases?](#question-20-how-do-you-implement-idempotent-writes-in-databases)                                                  |

## Question 1. How do you design a music playlist management system?

# Design a Music Playlist Management System

## Direct answer

A music playlist management system allows users to create, modify, organize, and share playlists while supporting millions of users and songs. The design should optimize for:

- Fast playlist reads
- Efficient insertions, deletions, and reordering
- High availability
- Support for collaborative editing (optional)
- Scalable metadata storage

The main challenge is efficiently maintaining ordered collections that can grow large while handling frequent updates.

---

# 1. Requirements / Problem Framing

## Functional Requirements

- Create playlist
- Rename/delete playlist
- Add songs
- Remove songs
- Reorder songs
- Retrieve playlists
- Search playlists
- Share playlists
- Follow public playlists
- (Optional) Collaborative editing

---

## Non-functional Requirements

- Low latency (<100 ms reads)
- Highly available
- Durable storage
- Support millions of playlists
- Eventually consistent collaboration is acceptable
- Strong consistency for playlist ownership and permissions

---

# 2. High-Level Architecture

```text
                Clients
                   |
             API Gateway
                   |
        -----------------------
        |         |           |
 Playlist   User Service   Search
 Service
        |
 -------------------------------
 |             |               |
Metadata DB   Cache      Event Queue
 |                              |
 |                              |
Object Store               Search Index
```

---

## Components

### Playlist Service

Responsible for:

- Create playlists
- Add/remove songs
- Reordering
- Sharing

---

### User Service

Stores

- ownership
- permissions
- followers

---

### Metadata Database

Stores

- playlists
- song references
- ordering

---

### Cache

Frequently accessed playlists.

Example:

```
Playlist 123

[
 Song A
 Song B
 Song C
]
```

Cached in memory.

---

### Search Service

Indexes

- playlist name
- creator
- tags
- genres

---

# 3. Data Model

## Playlist

```text
Playlist

playlistId
ownerId
name
description
visibility
createdAt
updatedAt
```

---

## PlaylistSong

```text
playlistId
songId
position
addedBy
addedAt
```

Composite key:

```
playlistId + position
```

---

## Song

Usually owned by a separate music catalog service.

```text
songId
artist
album
duration
metadata
```

---

# 4. APIs

Create playlist

```http
POST /playlists
```

---

Get playlist

```http
GET /playlists/{id}
```

---

Add song

```http
POST /playlists/{id}/songs
```

Body

```json
{
  "songId": 456
}
```

---

Delete song

```http
DELETE /playlists/{id}/songs/{songId}
```

---

Move song

```http
PATCH /playlists/{id}/songs/reorder
```

Example

```json
{
  "songId": 456,
  "newPosition": 12
}
```

---

Share playlist

```http
POST /playlists/{id}/share
```

---

# 5. High-Level Flow

### Create Playlist

```text
Client

↓

Playlist Service

↓

Metadata DB

↓

Return Playlist ID
```

---

### Add Song

```text
Client

↓

Playlist Service

↓

Validate Song

↓

Update DB

↓

Invalidate Cache

↓

Publish Event
```

---

### Read Playlist

```text
Client

↓

Cache

↓

Miss

↓

Database

↓

Cache Result

↓

Return Playlist
```

---

# 6. Efficient Ordering Strategy

Naively storing positions as

```
1
2
3
4
5
```

causes problems.

Moving song

```
5

to

2
```

requires updating every row.

Expensive.

---

## Better Approach

Use sparse ordering.

Example

```
100

200

300

400
```

Insert between

```
200

and

300
```

becomes

```
250
```

No massive updates.

---

Eventually rebalance positions when gaps become small.

---

Alternative:

Fractional indexing

```
1.0

2.0

2.5

2.75

3.0
```

Common in collaborative editors.

---

# 7. Scaling Strategy

Suppose

```
100 M users

1 B playlists

Average 60 songs
```

Rows

```
60 Billion PlaylistSong rows
```

Need partitioning.

---

## Shard by Playlist ID

```text
Playlist 1

↓

Shard A

Playlist 2

↓

Shard B
```

Advantages

- Playlist reads hit one shard
- Writes localized
- Easy scaling

---

# 8. Caching

Most playlists are read repeatedly.

Cache

```
Playlist metadata

Song order

Top playlists
```

Redis example

```
playlist:123
```

TTL

```
30 minutes
```

Updates invalidate cache.

---

# 9. Collaboration

If multiple users edit simultaneously.

Possible issues

```
User A removes Song X

User B moves Song X
```

Conflict.

---

Solutions

### Option 1

Last Write Wins

Simple

Good enough for many systems.

---

### Option 2

Operational Transform (OT)

Google Docs approach.

Complex.

---

### Option 3

Conflict-free Replicated Data Types (CRDTs)

Excellent for collaborative editing.

Spotify-like collaborative playlists often use approaches similar to CRDTs.

---

# 10. Reliability

Use event-driven architecture.

Example

```
Song Added

↓

Publish Event

↓

Recommendation Service

↓

Activity Feed

↓

Analytics
```

Playlist service remains independent.

---

# 11. Security / Observability

### Security

- Authentication (JWT/OAuth)
- Authorization (owner/editor/viewer)
- Private/public playlists
- Rate limiting
- Input validation

---

### Observability

Monitor

- Playlist creation rate
- Add/remove latency
- Cache hit ratio
- Failed writes
- DB latency
- Reorder operations
- API error rate

Use distributed tracing to debug slow requests across services.

---

# 12. Trade-offs

| Design Choice             | Advantages              | Disadvantages                  |
| ------------------------- | ----------------------- | ------------------------------ |
| Store integer positions   | Simple                  | Expensive reordering           |
| Sparse positions          | Efficient inserts       | Periodic rebalancing           |
| Cache playlists           | Fast reads              | Cache invalidation complexity  |
| SQL                       | ACID, transactions      | Harder horizontal scaling      |
| NoSQL                     | Massive scalability     | Limited joins and transactions |
| Last Write Wins           | Simple                  | Lost concurrent updates        |
| CRDT                      | Excellent collaboration | Complex implementation         |
| Event-driven architecture | Decoupled services      | Eventual consistency           |

---

# Interview-ready Summary

> "I'd separate playlist management into a dedicated service backed by a metadata database storing playlists and ordered song references. Playlists would be sharded by `playlistId` for scalability, cached in Redis for low-latency reads, and updated using sparse or fractional indexing to avoid rewriting large portions of the playlist during reordering. Changes would invalidate the cache and emit events for downstream systems like recommendations and analytics. For collaborative playlists, I'd start with last-write-wins for simplicity and evolve to CRDTs or Operational Transform if real-time concurrent editing becomes a key requirement. This design provides fast reads, efficient updates, high availability, and scales to billions of playlists."

## Question 2. How do you design a product recommendation engine at the class diagram level?

# Design a Product Recommendation Engine (LLD / Class Diagram)

## Direct answer

At the class diagram level, a product recommendation engine should separate responsibilities into:

- **Recommendation strategies** (Collaborative Filtering, Content-Based, Trending, etc.)
- **Recommendation engine** (orchestrates recommendation generation)
- **Feature providers** (user behavior, product metadata)
- **Ranking service** (scores and sorts candidates)
- **Repositories** (fetch data)
- **Recommendation models** (User, Product, Recommendation)

A good LLD uses the **Strategy Pattern**, making it easy to plug in new recommendation algorithms without modifying the engine.

---

# Core Classes

```text
                +----------------------+
                | RecommendationEngine |
                +----------------------+
                | strategy             |
                | rankingService       |
                | repository           |
                +----------------------+
                | recommend(userId)    |
                +----------+-----------+
                           |
                           |
              uses RecommendationStrategy
                           |
          ------------------------------------
          |                 |                |
          |                 |                |
+----------------+  +----------------+ +----------------+
| Collaborative  |  | ContentBased   | | Trending       |
| Strategy       |  | Strategy       | | Strategy       |
+----------------+  +----------------+ +----------------+
| recommend()    |  | recommend()    | | recommend()    |
+----------------+  +----------------+ +----------------+

                |
                |
        ProductRepository
                |
      -------------------
      |                 |
    Product          UserHistory

                |
        RankingService
                |
         Recommendation
```

---

# Main Classes

## User

```javascript
class User {
  constructor(id, preferences = []) {
    this.id = id;
    this.preferences = preferences;
  }
}
```

---

## Product

```javascript
class Product {
  constructor(id, category, tags, rating) {
    this.id = id;
    this.category = category;
    this.tags = tags;
    this.rating = rating;
  }
}
```

---

## Recommendation

```javascript
class Recommendation {
  constructor(product, score) {
    this.product = product;
    this.score = score;
  }
}
```

---

# Strategy Interface

```javascript
class RecommendationStrategy {
  recommend(user, repository) {
    throw new Error("Implement recommend()");
  }
}
```

This is the abstraction that allows different recommendation algorithms.

---

# Concrete Strategies

## Collaborative Filtering

```javascript
class CollaborativeFilteringStrategy extends RecommendationStrategy {
  recommend(user, repository) {
    // Find similar users
    // Recommend products they liked
  }
}
```

---

## Content-Based Filtering

```javascript
class ContentBasedStrategy extends RecommendationStrategy {
  recommend(user, repository) {
    // Match products with user interests
  }
}
```

---

## Trending Products

```javascript
class TrendingStrategy extends RecommendationStrategy {
  recommend(user, repository) {
    // Return globally popular products
  }
}
```

---

## Hybrid Strategy

Combines multiple algorithms.

```javascript
class HybridStrategy extends RecommendationStrategy {
  constructor(strategies) {
    super();
    this.strategies = strategies;
  }

  recommend(user, repository) {
    let results = [];

    for (const strategy of this.strategies) {
      results.push(...strategy.recommend(user, repository));
    }

    return results;
  }
}
```

---

# Product Repository

Responsible for fetching data.

```javascript
class ProductRepository {
  getProducts() {}

  getProduct(id) {}

  getUserHistory(userId) {}

  getPopularProducts() {}
}
```

The recommendation engine doesn't know where data comes from (SQL, NoSQL, cache, ML feature store, etc.).

---

# Ranking Service

Multiple strategies may return hundreds of candidates.

Ranking decides the final order.

```javascript
class RankingService {
  rank(products) {
    return products.sort((a, b) => b.score - a.score);
  }
}
```

In production, ranking often incorporates signals such as:

- User affinity
- Click-through rate (CTR)
- Purchase probability
- Product popularity
- Freshness
- Diversity
- Business rules (e.g., sponsored items)

---

# Recommendation Engine

The orchestrator.

```javascript
class RecommendationEngine {
  constructor(strategy, repository, rankingService) {
    this.strategy = strategy;
    this.repository = repository;
    this.rankingService = rankingService;
  }

  recommend(user) {
    const candidates = this.strategy.recommend(user, this.repository);

    return this.rankingService.rank(candidates);
  }
}
```

The engine doesn't implement recommendation logic itself—it delegates to the selected strategy and then ranks the results.

---

# Supporting Classes

## UserHistory

```javascript
class UserHistory {
  constructor(userId) {
    this.userId = userId;
    this.viewed = [];
    this.clicked = [];
    this.purchased = [];
  }
}
```

---

## Feature Store

```javascript
class FeatureStore {
  getUserFeatures(userId) {}

  getProductFeatures(productId) {}
}
```

This abstracts access to precomputed ML features used during recommendation.

---

## Recommendation Cache

```javascript
class RecommendationCache {
  get(userId) {}

  put(userId, recommendations) {}
}
```

Popular recommendation lists can be cached to reduce latency.

---

# Design Patterns Used

| Pattern                         | Usage                                                                              |
| ------------------------------- | ---------------------------------------------------------------------------------- |
| **Strategy**                    | Switch between recommendation algorithms without changing the engine.              |
| **Repository**                  | Abstract data access from business logic.                                          |
| **Dependency Injection**        | Inject strategies, repositories, and ranking services for flexibility and testing. |
| **Composite (Hybrid Strategy)** | Combine outputs from multiple recommendation strategies.                           |

---

# Extensibility

Adding a new recommendation algorithm requires no changes to the engine:

```javascript
class RecentlyViewedStrategy extends RecommendationStrategy {
  recommend(user, repository) {
    // Recommend products similar to recently viewed items
  }
}
```

Then simply inject it:

```javascript
const strategy = new HybridStrategy([
  new CollaborativeFilteringStrategy(),
  new ContentBasedStrategy(),
  new RecentlyViewedStrategy(),
]);
```

This adheres to the **Open/Closed Principle**—the engine is open for extension but closed for modification.

---

# Interview-ready Summary

> "For the LLD, I'd model the recommendation engine around the Strategy pattern. The `RecommendationEngine` orchestrates the flow, delegating candidate generation to interchangeable `RecommendationStrategy` implementations like Collaborative Filtering, Content-Based, Trending, or Hybrid. A `ProductRepository` abstracts data access, a `FeatureStore` provides user and product features, and a `RankingService` scores and orders candidates. This design is modular, testable, and extensible, allowing new recommendation algorithms to be added without changing the engine itself."

## Question 3. How do you design a loyalty points system?

## Question 4. How do you design a micro-billing system for pay-per-use services?

## Question 5. How do you design a personal budgeting system?

## Question 6. How do you design a fitness class reservation system?

## Question 7. How do you design a drone delivery scheduling system?

## Question 8. How do you design a movie recommendation engine (LLD)?

## Question 9. How do you design a payment settlement service internally?

## Question 10. How do you design an online auction bidding bot?

## Question 11. What is database shadowing?

## Question 12. What are write-heavy database optimization techniques?

## Question 13. What are hot keys in distributed caches?

## Question 14. How do you design a time-to-live (TTL) database?

## Question 15. What are vector databases?

## Question 16. How do you handle schema evolution in NoSQL?

## Question 17. What is ZooKeeper and how is it used in distributed databases?

## Question 18. What is a two-phase commit protocol?

## Question 19. What is a three-phase commit protocol?

## Question 20. How do you implement idempotent writes in databases?
