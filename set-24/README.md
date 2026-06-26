# Set 24

| S.No. | Question                                                                                                                                                                |
| ----- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How would you design a global-scale video streaming platform (like Netflix)?](#question-1-how-would-you-design-a-global-scale-video-streaming-platform-like-netflix)   |
| 2.    | [What's the architecture of a distributed build system (like Bazel)?](#question-2-whats-the-architecture-of-a-distributed-build-system-like-bazel)                      |
| 3.    | [How do you optimize for tail latency in large-scale distributed systems?](#question-3-how-do-you-optimize-for-tail-latency-in-large-scale-distributed-systems)         |
| 4.    | [How would you design a decentralized social network (like Mastodon)?](#question-4-how-would-you-design-a-decentralized-social-network-like-mastodon)                   |
| 5.    | [What are hybrid consistency models in databases?](#question-5-what-are-hybrid-consistency-models-in-databases)                                                         |
| 6.    | [How would you design a multi-tenant SaaS application with strong isolation?](#question-6-how-would-you-design-a-multi-tenant-saas-application-with-strong-isolation)   |
| 7.    | [What are shadow writes, and how are they used in migration strategies?](#question-7-what-are-shadow-writes-and-how-are-they-used-in-migration-strategies)              |
| 8.    | [How would you design an API Gateway from scratch?](#question-8-how-would-you-design-an-api-gateway-from-scratch)                                                       |
| 9.    | [How do you scale metadata storage in a distributed filesystem?](#question-9-how-do-you-scale-metadata-storage-in-a-distributed-filesystem)                             |
| 10.   | [What's the role of a transaction coordinator in distributed systems?](#question-10-whats-the-role-of-a-transaction-coordinator-in-distributed-systems)                 |
| 11.   | [How would you design a high-frequency trading (HFT) system?](#question-11-how-would-you-design-a-high-frequency-trading-hft-system)                                    |
| 12.   | [What's the difference between leader election and consensus algorithms?](#question-12-whats-the-difference-between-leader-election-and-consensus-algorithms)           |
| 13.   | [How do you optimize for "geo-distributed writes" in a database?](#question-13-how-do-you-optimize-for-geo-distributed-writes-in-a-database)                            |
| 14.   | [How would you design a system like Dropbox for real-time file sync?](#question-14-how-would-you-design-a-system-like-dropbox-for-real-time-file-sync)                  |
| 15.   | [What is write amplification in SSD-based systems?](#question-15-what-is-write-amplification-in-ssd-based-systems)                                                      |
| 16.   | [How would you design a system for distributed ML model training?](#question-16-how-would-you-design-a-system-for-distributed-ml-model-training)                        |
| 17.   | [What's the difference between OLTP and OLAP at the architecture level?](#question-17-whats-the-difference-between-oltp-and-olap-at-the-architecture-level)             |
| 18.   | [How do you design a hybrid transactional/analytical processing (HTAP) system?](#question-18-how-do-you-design-a-hybrid-transactionalanalytical-processing-htap-system) |
| 19.   | [How would you handle replay attacks in system design?](#question-19-how-would-you-handle-replay-attacks-in-system-design)                                              |
| 20.   | [How would you design a distributed system with Byzantine fault tolerance?](#question-20-how-would-you-design-a-distributed-system-with-byzantine-fault-tolerance)      |

## Question 1. How would you design a global-scale video streaming platform (like Netflix)?

# Design a Global-Scale Video Streaming Platform (like Netflix)

## Direct answer

A global-scale video streaming platform is designed using a **microservices architecture**, **CDNs for content delivery**, **distributed object storage**, **adaptive bitrate streaming**, **recommendation services**, and **globally replicated metadata services**.

The core idea is:

- Store video masters centrally.
- Transcode into multiple formats/resolutions.
- Distribute videos to CDNs worldwide.
- Stream from the nearest CDN using adaptive bitrate (HLS/DASH).
- Separate user metadata, playback state, and recommendation systems.
- Optimize for **low latency, high availability, and massive scalability**.

---

# 1. Requirements / Problem Framing

## Functional Requirements

### Content Management

- Upload movies/shows
- Store metadata
- Manage subtitles
- Support multiple audio languages

### Video Playback

- Search content
- Browse catalog
- Stream videos
- Resume playback
- Adaptive quality switching
- Continue watching

### User Features

- Authentication
- Profiles
- Recommendations
- Watch history
- Favorites
- Ratings

### Admin

- Upload content
- Schedule releases
- Regional restrictions
- Analytics

---

## Non-Functional Requirements

- Millions of concurrent viewers
- Global availability
- Very low buffering
- High throughput
- Fault tolerant
- Elastic scaling
- Low startup latency
- DRM support

---

# 2. High-Level Architecture

```text
                 Mobile / TV / Web Apps
                          |
                    Global DNS
                          |
                 API Gateway / LB
                          |
         ------------------------------------
         |          |         |             |
   User Service Search  Catalog     Recommendation
         |          |         |             |
         ------------------------------------
                      |
               Metadata Database
                      |
                Playback Service
                      |
          Returns Manifest (HLS/DASH)
                      |
                 Nearest CDN Edge
                      |
               Video Segments (.ts/.m4s)
                      |
                  User Device

-----------------------------------------------------

 Content Upload

Uploader
    |
Upload Service
    |
Object Storage
    |
Transcoding Pipeline
    |
Multiple Resolutions
    |
CDN Distribution
```

---

# 3. Major Components

## API Gateway

Handles

- Authentication
- Rate limiting
- Routing
- Request aggregation

---

## User Service

Stores

- Accounts
- Profiles
- Subscription plans
- Devices

Database

- SQL

---

## Catalog Service

Stores

- Movies
- Genres
- Cast
- Languages
- Artwork
- Availability

Read-heavy

Uses:

- Cache
- Search index

---

## Search Service

Uses

- Inverted index
- Full-text search

Supports

- Autocomplete
- Fuzzy matching
- Filters

Typically backed by search engines like Elasticsearch/OpenSearch.

---

## Recommendation Service

Consumes

- Watch history
- Ratings
- Similar users
- Trending
- ML models

Returns personalized homepage rows.

---

## Playback Service

Responsible for

- Authorization
- DRM license validation
- Generate playback token
- Return streaming manifest

Returns

```
movie.m3u8
```

or

```
movie.mpd
```

---

# 4. Video Storage Pipeline

## Step 1

Upload master video

Example

```
4K ProRes
100 GB
```

Stored in object storage.

---

## Step 2

Transcoding

Generate

```
240p
360p
480p
720p
1080p
4K
```

Multiple codecs

```
H.264
H.265
AV1
VP9
```

---

## Step 3

Chunking

Break video into segments

Example

```
6-second chunks

segment1.ts
segment2.ts
segment3.ts
```

---

## Step 4

Generate Manifest

Example

```
master.m3u8

720p.m3u8

1080p.m3u8

4K.m3u8
```

---

## Step 5

Push to CDN

CDN caches

- Segments
- Images
- Posters
- Subtitles

---

# 5. Adaptive Bitrate Streaming (ABR)

Instead of downloading one huge video,

Client downloads

```
Manifest
```

Then requests chunks.

If bandwidth drops

```
1080p

↓

720p

↓

480p
```

without restarting playback.

Benefits

- Less buffering
- Better user experience

Protocols

- HLS
- MPEG-DASH

---

# 6. CDN Strategy

This is the most important scaling component.

Instead of

```
User -> Origin
```

Use

```
User

↓

Nearest CDN

↓

Origin
```

Benefits

- Lower latency
- Reduced origin traffic
- Better throughput
- Regional scaling

Example

```
India → Mumbai CDN

USA → Virginia CDN

Europe → Frankfurt CDN
```

Only cache misses reach the origin.

---

# 7. Data Storage

| Data              | Storage                  |
| ----------------- | ------------------------ |
| Users             | SQL                      |
| Catalog           | SQL                      |
| Watch history     | NoSQL                    |
| Playback progress | NoSQL                    |
| Recommendations   | Key-value store          |
| Search            | Elasticsearch/OpenSearch |
| Videos            | Object Storage           |
| Images            | CDN/Object Storage       |
| Logs              | Data Lake                |

---

# 8. Caching Strategy

Several cache layers.

## Metadata Cache

```
Movie details
Genres
Cast
```

Redis

---

## Homepage Cache

Popular rows

```
Trending

Top Picks

Continue Watching
```

---

## CDN Cache

Caches

- Videos
- Images
- Thumbnails
- Subtitles

---

## Client Cache

Stores

- Recently watched segments
- Images
- Metadata

---

# 9. Playback Flow

```text
User presses Play

↓

API Gateway

↓

Playback Service

↓

Verify Subscription

↓

Generate Playback Token

↓

Return Manifest URL

↓

Client requests CDN

↓

CDN returns chunks

↓

Player buffers

↓

Playback starts

↓

Client periodically updates progress
```

---

# 10. Recommendation Pipeline

Offline

```
Watch logs

↓

Spark/Flink

↓

Feature Engineering

↓

ML Models

↓

Recommendation Store
```

Online

```
User opens app

↓

Recommendation Service

↓

Fetch Top N

↓

Return homepage
```

---

# 11. Scaling Strategy

## Stateless Services

All APIs remain stateless.

Easy horizontal scaling.

---

## Database Sharding

Users

```
Shard by UserID
```

Watch history

```
Shard by UserID
```

Playback logs

```
Time-based partitions
```

---

## Read Replicas

Catalog

```
1 Primary

10 Replicas
```

Thousands of reads/sec.

---

## CDN Scaling

Almost all traffic goes to CDN.

Origin traffic remains low.

---

# 12. Reliability Strategy

Multiple origin regions.

```
US

Europe

Asia
```

Cross-region replication.

If one region fails,

DNS redirects users.

---

## Retry Logic

Retry

- Metadata fetch
- Playback token
- Recommendation

Do **not** retry video segments excessively to avoid worsening congestion; rely on player retry policies with exponential backoff.

---

## Graceful Degradation

If recommendations fail

Return

```
Trending
```

If subtitles fail

Continue playback.

If personalization fails

Serve cached homepage.

---

# 13. Capacity / Sizing (Example)

### Assumptions

- 100 million daily active users
- 20 million concurrent streams at peak
- Average stream bitrate: **5 Mbps** (mixed resolutions)

### Bandwidth

```
20M × 5 Mbps
≈ 100 Tbps
```

This traffic is primarily served by geographically distributed CDNs rather than origin servers.

### Storage

If:

- 200,000 titles
- Average encoded size across all renditions: 25 GB/title

```
≈ 5 PB
```

With replication and multiple codecs, practical storage requirements can grow to tens of petabytes.

---

# 14. Security / Observability

## Security

- OAuth/JWT authentication
- Short-lived signed playback URLs
- DRM (e.g., Widevine, PlayReady, FairPlay)
- TLS everywhere
- Geo-restriction enforcement
- Secure license servers
- Watermarking for premium content

## Observability

Track:

- Startup time
- Buffering ratio
- Rebuffer events
- CDN cache hit ratio
- Bitrate switches
- Playback failures
- API latency (P50/P95/P99)
- Transcoding failures
- Origin-to-CDN replication lag

Distributed tracing helps follow requests across gateway, playback, recommendation, and metadata services.

---

# 15. Trade-offs

| Decision                | Pros                              | Cons                                      |
| ----------------------- | --------------------------------- | ----------------------------------------- |
| CDN vs Origin streaming | Low latency, huge scalability     | CDN cost                                  |
| Adaptive bitrate        | Smooth playback                   | More storage and encoding cost            |
| Object storage          | Durable, scalable                 | Higher access latency without CDN         |
| Pre-transcoding         | Fast startup                      | Increased storage usage                   |
| Microservices           | Independent scaling               | Operational complexity                    |
| Event-driven processing | Decoupled ingestion and analytics | Event ordering and operational complexity |

---

# Interview-ready summary

> "I would design a Netflix-like platform using microservices backed by distributed object storage for video assets, an asynchronous transcoding pipeline to generate multiple codecs and resolutions, adaptive bitrate streaming over HLS or DASH, and globally distributed CDNs to serve content from edge locations. Metadata, search, user profiles, and recommendations would be separate services with appropriate databases and caches. The system would scale horizontally, use replication and multi-region failover for availability, and employ DRM, signed playback URLs, and comprehensive monitoring to ensure secure, reliable, low-latency streaming for millions of concurrent users worldwide."

## Question 2. What's the architecture of a distributed build system (like Bazel)?

## Question 3. How do you optimize for tail latency in large-scale distributed systems?

## Question 4. How would you design a decentralized social network (like Mastodon)?

## Question 5. What are hybrid consistency models in databases?

## Question 6. How would you design a multi-tenant SaaS application with strong isolation?

## Question 7. What are shadow writes, and how are they used in migration strategies?

## Question 8. How would you design an API Gateway from scratch?

## Question 9. How do you scale metadata storage in a distributed filesystem?

## Question 10. What's the role of a transaction coordinator in distributed systems?

## Question 11. How would you design a high-frequency trading (HFT) system?

## Question 12. What's the difference between leader election and consensus algorithms?

## Question 13. How do you optimize for "geo-distributed writes" in a database?

## Question 14. How would you design a system like Dropbox for real-time file sync?

## Question 15. What is write amplification in SSD-based systems?

## Question 16. How would you design a system for distributed ML model training?

## Question 17. What's the difference between OLTP and OLAP at the architecture level?

## Question 18. How do you design a hybrid transactional/analytical processing (HTAP) system?

## Question 19. How would you handle replay attacks in system design?

## Question 20. How would you design a distributed system with Byzantine fault tolerance?
