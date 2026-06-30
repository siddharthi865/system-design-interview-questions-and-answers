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

# Architecture of a Distributed Build System (like Bazel)

## Direct answer

A distributed build system accelerates software builds by **analyzing the dependency graph (DAG), scheduling independent build actions across many remote workers, caching build outputs, and reusing previously built artifacts**. Systems like Bazel achieve scalability through **deterministic builds**, **content-addressable storage (CAS)**, **remote execution**, and **incremental builds**.

The key idea is:

- Model the build as a **Directed Acyclic Graph (DAG)**.
- Build only what has changed.
- Execute independent tasks in parallel.
- Cache outputs using content hashes.
- Reuse artifacts across developers and CI pipelines.

---

# 1. Requirements / Problem Framing

## Functional Requirements

- Build projects in multiple languages
- Resolve dependencies
- Incremental builds
- Parallel execution
- Remote execution
- Shared build cache
- Test execution
- Artifact generation
- Build reproducibility

## Non-Functional Requirements

- Low build latency
- Horizontal scalability
- High cache hit ratio
- Deterministic outputs
- Fault tolerance
- Efficient resource utilization
- Secure execution of untrusted build actions

---

# 2. High-Level Architecture

```text
                    Developer / CI
                           |
                    Build Client (CLI)
                           |
                  Build Coordinator
                           |
        +------------------+------------------+
        |                  |                  |
 Dependency Analyzer   Scheduler       Cache Service
        |                  |                  |
        |                  |         Content Addressable
        |                  |           Storage (CAS)
        |                  |
        +------------------+
                 |
          Remote Execution API
                 |
      +----------+-----------+-----------+
      |                      |           |
   Worker 1               Worker 2    Worker N
      |                      |           |
  Sandbox Build         Sandbox Build  Sandbox Build
      |
 Artifact Upload to CAS
```

---

# 3. Core Components

## Build Client

Responsible for:

- Parsing build commands
- Loading build configuration
- Sending build requests
- Downloading final artifacts

Examples:

```text
bazel build //app:server
```

---

## Dependency Analyzer

Reads build definitions and constructs a **Directed Acyclic Graph (DAG)**.

Example:

```text
Frontend
   |
 API
 /   \
Core Utils
```

The analyzer determines:

- What targets need rebuilding
- Build order
- Parallelizable tasks

---

## Scheduler

The scheduler:

- Traverses the DAG
- Identifies ready-to-run actions
- Assigns work to available workers
- Tracks completion
- Retries failed tasks when appropriate

Scheduling only begins once all dependencies of an action are complete.

---

## Remote Workers

Workers execute isolated build actions.

Responsibilities:

- Download inputs
- Compile/test/package
- Upload outputs
- Return execution status

Workers are stateless and can be scaled horizontally.

---

## Content Addressable Storage (CAS)

Instead of naming files by path:

```text
/bin/server
```

CAS stores objects by hash:

```text
SHA256(source + compiler + flags)
```

Benefits:

- Automatic deduplication
- Efficient sharing
- Immutable artifacts
- Fast cache lookup

---

# 4. Build Execution Flow

```text
Developer starts build
          |
Parse BUILD files
          |
Construct dependency DAG
          |
Check remote/local cache
          |
Cache hit?
     |
 +---+---+
 |       |
Yes      No
 |        |
Download  Schedule action
artifact      |
              |
      Remote worker executes
              |
      Upload artifact to CAS
              |
Return final executable
```

---

# 5. Incremental Builds

Only rebuild affected targets.

Example:

```text
A
|
B
|
C
|
D
```

If only **B** changes:

- Rebuild B
- Rebuild C
- Rebuild D

A is reused from cache.

This dramatically reduces build time for large codebases.

---

# 6. Remote Execution

Instead of compiling locally:

```text
Laptop

↓

Compile
```

Use:

```text
Laptop

↓

Coordinator

↓

Worker Cluster

↓

Artifacts
```

Advantages:

- Powerful build machines
- Better parallelism
- Consistent environments
- Faster CI builds

---

# 7. Caching Strategy

## Local Cache

Stored on the developer's machine.

Useful for:

- Repeated local builds
- Offline development

---

## Remote Cache

Shared across developers and CI.

If Developer A builds:

```text
foo.cpp
```

Developer B can reuse the compiled object without recompiling, provided the action hash matches exactly.

---

## Action Cache

Each build action is identified by a hash derived from:

- Source files
- Compiler version
- Compiler flags
- Environment variables (when declared)
- Dependency artifacts

If the hash matches a previous build:

```text
Return cached artifact
```

No compilation is needed.

---

# 8. Parallel Execution

Independent targets execute simultaneously.

Example:

```text
         App
       /     \
 Backend   Frontend
   /   \        |
Core Utils     UI
```

Possible execution:

```text
Core
Utils
UI
```

run in parallel.

Once complete:

```text
Backend
```

can execute.

Finally:

```text
App
```

This exploits all available CPUs or workers.

---

# 9. Fault Tolerance

Worker failure:

```text
Worker crashes

↓

Coordinator detects timeout

↓

Reschedule action
```

Since actions are deterministic and idempotent, they can safely be retried.

Worker state is not relied upon.

---

# 10. Scalability Strategy

## Stateless Coordinator

The coordinator maintains scheduling state, but front-end API instances can remain stateless and scale behind a load balancer.

---

## Horizontal Worker Scaling

Need more throughput?

```text
100 Workers

↓

500 Workers
```

No architectural changes required.

---

## Immutable Artifacts

Artifacts never change.

This enables:

- Safe replication
- Global caching
- Easy deduplication

---

## Distributed CAS

Large deployments shard CAS by hash.

Example:

```text
Hash Prefix

00–1F → Storage Node A

20–3F → Storage Node B

...
```

This supports petabytes of artifacts.

---

# 11. Security / Observability

## Security

- Sandbox or containerize each build action
- Authenticate clients and workers
- Encrypt communication (TLS)
- Verify artifact integrity with hashes
- Enforce access controls for repositories and caches
- Restrict network and filesystem access during builds for hermeticity

## Observability

Monitor:

- Build duration
- Queue wait time
- Worker utilization
- Cache hit ratio
- Cache download/upload latency
- Failed actions
- Worker health
- Scheduler throughput

Distributed tracing helps identify slow build stages across coordinator, cache, and workers.

---

# 12. Trade-offs

| Decision                    | Pros                                       | Cons                                      |
| --------------------------- | ------------------------------------------ | ----------------------------------------- |
| Remote execution            | Faster builds, better resource utilization | Network overhead                          |
| Shared remote cache         | Reuses work across developers and CI       | Requires deterministic builds             |
| Content-addressable storage | Deduplication, immutability                | Hash computation and storage management   |
| Incremental builds          | Very fast rebuilds                         | Accurate dependency tracking is essential |
| Distributed workers         | Massive parallelism                        | Scheduler and infrastructure complexity   |
| Hermetic builds             | Reproducible, cache-friendly               | Requires strict dependency declaration    |

---

# Interview-ready summary

> "I would design a distributed build system around a dependency DAG, where the client analyzes build targets and the scheduler dispatches independent actions to a pool of remote workers. Each action executes in a hermetic sandbox, with inputs and outputs stored in a content-addressable store. A shared action cache keyed by the hash of sources, tools, flags, and declared environment enables incremental and cross-user builds. Independent tasks execute in parallel, failed actions are retried on other workers, and stateless workers with distributed CAS allow the system to scale efficiently while producing deterministic, reproducible builds."

## Question 3. How do you optimize for tail latency in large-scale distributed systems?

# Optimizing Tail Latency in Large-Scale Distributed Systems

## Direct answer

**Tail latency** refers to the slowest requests in a system, typically measured at the **P95, P99, or P99.9** percentiles. In large-scale distributed systems, users often experience these slowest requests rather than the average latency.

Optimizing tail latency involves:

- Eliminating bottlenecks
- Reducing queueing delays
- Replicating or hedging slow requests
- Using timeouts and failover
- Prioritizing critical traffic
- Minimizing fan-out
- Continuously monitoring high-percentile latencies

Large systems like Google, Amazon, and Microsoft focus on **P99 latency** because a single slow component can significantly impact end-to-end response time.

---

# Why tail latency matters

Average latency can be misleading.

Example:

| Percentile | Latency |
| ---------- | ------- |
| P50        | 20 ms   |
| P90        | 40 ms   |
| P95        | 70 ms   |
| P99        | 600 ms  |
| P99.9      | 2 s     |

Although the average might be around **30 ms**, **1% of users experience 600 ms**, which noticeably degrades user experience.

---

# The fan-out problem

Distributed systems frequently call multiple services in parallel.

```text
               User Request
                     |
         -------------------------
         |      |      |      |
      ServiceA ServiceB ServiceC ServiceD
```

The overall response time is approximately determined by the **slowest** dependency.

Example:

| Service | Latency |
| ------- | ------- |
| A       | 25 ms   |
| B       | 30 ms   |
| C       | 28 ms   |
| D       | 250 ms  |

Overall latency ≈ **250 ms**, even though most services responded quickly.

As the number of downstream calls increases, the probability that one experiences a long-tail delay also increases.

---

# Common causes of tail latency

- CPU contention
- Queue buildup
- Garbage collection pauses
- Disk I/O delays
- Network congestion
- Cache misses
- Lock contention
- Noisy neighbors in shared infrastructure
- Slow replicas
- Cross-region communication
- Large payloads

---

# Techniques to optimize tail latency

## 1. Reduce queueing delays

Queueing often contributes more latency than processing.

Instead of:

```text
Request
   ↓
Long queue
   ↓
Worker
```

Use:

- More worker threads/processes (up to practical limits)
- Autoscaling
- Backpressure
- Admission control

Keeping utilization below saturation reduces queueing dramatically.

---

## 2. Replicate or hedge requests

If one replica is slow:

```text
Read Request
     |
Replica A
Replica B
Replica C
```

Send a duplicate request after a small delay (for example, after the expected P95 latency).

Use the first successful response and cancel the others.

Benefits:

- Masks temporary slowdowns
- Reduces P99 latency

Trade-off:

- Higher resource consumption
- Best suited for idempotent operations

---

## 3. Use timeouts

Never wait indefinitely.

Example:

```text
Service timeout = 80 ms

If exceeded

↓

Fallback
```

Timeouts prevent slow dependencies from consuming resources unnecessarily.

---

## 4. Retry intelligently

Retries should use:

- Exponential backoff
- Jitter
- Retry budgets
- Only for transient failures

Avoid immediate retries, which can amplify overload.

---

## 5. Reduce fan-out

Instead of:

```text
API

↓

50 downstream calls
```

Use:

- Aggregated APIs
- Precomputed data
- Batch requests
- Denormalized read models

Fewer dependencies reduce the chance of encountering a slow component.

---

## 6. Cache aggressively

Avoid unnecessary remote calls.

Examples:

- CDN
- Redis
- Local in-memory cache

A cache hit:

```text
Memory

↓

1 ms
```

instead of:

```text
Remote DB

↓

30 ms
```

reduces both average and tail latency.

---

## 7. Load balancing

Distribute requests evenly.

Avoid repeatedly sending traffic to overloaded instances.

Useful strategies:

- Least-connections
- Least-request
- Power of Two Choices
- Latency-aware load balancing

Latency-aware balancing can steer traffic away from consistently slow replicas.

---

## 8. Isolate noisy neighbors

Prevent one workload from degrading another.

Techniques:

- Resource quotas
- CPU pinning
- Dedicated worker pools
- Separate queues
- Container or VM isolation

---

## 9. Prioritize requests

Maintain separate queues for different priorities.

Example:

```text
High Priority Queue

Normal Queue

Background Queue
```

Critical user-facing requests avoid being blocked by batch jobs.

---

## 10. Keep data local

Cross-region communication increases latency.

Instead of:

```text
India

↓

US Database
```

Use:

```text
India

↓

India Replica
```

This reduces both average and tail latency.

---

## 11. Optimize garbage collection

Long GC pauses can create latency spikes.

Approaches include:

- Smaller heaps
- Concurrent or low-pause collectors
- Reducing object allocations
- Object pooling where appropriate

---

## 12. Use asynchronous processing

Move non-critical work off the request path.

Instead of:

```text
Request

↓

Send email

↓

Write analytics

↓

Return
```

Use:

```text
Request

↓

Queue events

↓

Return

↓

Background workers
```

This shortens the critical path.

---

## 13. Adaptive concurrency control

When a service is overloaded:

- Reduce concurrent requests
- Shed excess load
- Reject early rather than timing out

This protects latency for accepted requests.

---

# Observability

Average latency is insufficient.

Track:

- P50
- P90
- P95
- P99
- P99.9
- Queue wait time
- Service time
- Cache hit ratio
- Retry rate
- Timeout rate
- Error rate

Distributed tracing helps identify which downstream service contributes most to end-to-end tail latency.

---

# Trade-offs

| Technique        | Benefit                     | Trade-off                                     |
| ---------------- | --------------------------- | --------------------------------------------- |
| Hedged requests  | Lower P99 latency           | Extra CPU/network usage                       |
| Caching          | Faster responses            | Invalidation complexity                       |
| Timeouts         | Prevent resource exhaustion | May fail slow but valid requests              |
| Retries          | Recover transient failures  | Retry storms if uncontrolled                  |
| Local replicas   | Lower latency               | Replication lag                               |
| Request batching | Fewer network calls         | Slight increase in individual request latency |
| Async processing | Shorter critical path       | Eventual consistency                          |

---

# Interview-ready summary

> "Tail latency is driven by the slowest requests, so optimizing only average latency is insufficient. In distributed systems, I would focus on reducing queueing delays, minimizing fan-out, using caching, latency-aware load balancing, and keeping data geographically close to users. For resilience against transient slowness, I'd use timeouts, retry budgets with exponential backoff and jitter, and hedged requests for idempotent operations. Finally, I'd monitor P95/P99 latencies, queue lengths, and distributed traces, since these metrics reveal bottlenecks that average latency hides."

## Question 4. How would you design a decentralized social network (like Mastodon)?

# Design a Decentralized Social Network (like Mastodon)

## Direct answer

A decentralized social network consists of **independent servers (instances)** that each manage their own users, storage, moderation policies, and timelines while communicating through a **federation protocol** (such as ActivityPub). Users can follow and interact with accounts on any instance, creating a global social graph without relying on a single central authority.

The key architectural principles are:

- **Federation instead of centralization**
- **Local autonomy with global interoperability**
- **Asynchronous communication between servers**
- **Eventual consistency across instances**
- **Independent moderation and administration**

---

# 1. Requirements / Problem Framing

## Functional Requirements

### User Features

- Register on an instance
- Create posts
- Follow users on other instances
- Like, boost (repost), reply
- View home and public timelines
- Search users and posts
- Upload media

### Federation

- Discover remote users
- Deliver posts across instances
- Synchronize profile updates
- Propagate likes, replies, boosts
- Handle follows/unfollows

### Moderation

- Block users
- Block instances
- Report abuse
- Content filtering
- Instance-specific moderation policies

---

## Non-Functional Requirements

- High availability
- Horizontal scalability
- Eventual consistency
- Fault tolerance
- Low latency for local operations
- Secure inter-instance communication

---

# 2. High-Level Architecture

```text
                  Instance A
             +------------------+
             | API Gateway      |
             | User Service     |
             | Timeline Service |
             | Media Service    |
             | Federation Svc   |
             +--------+---------+
                      |
          ActivityPub Messages
                      |
---------------------------------------------------
                      |
             +--------+---------+
             | Federation Svc   |
             | User Service     |
             | Timeline Service |
             | Media Service    |
             | Database         |
             +------------------+
                  Instance B

Users on different instances can:
Follow
Reply
Boost
Like
Message
```

Each instance operates independently while participating in the federation.

---

# 3. Core Components

## API Gateway

Handles:

- Authentication
- Rate limiting
- Request routing

---

## User Service

Stores:

- Profiles
- Followers
- Following
- Preferences
- Block lists

Local users are authoritative on their home instance.

Remote users are represented by cached profile records.

---

## Post Service

Responsible for:

- Creating posts
- Editing (if supported)
- Deleting
- Visibility rules
- Media attachments

Posts receive globally unique identifiers (typically URLs).

---

## Timeline Service

Maintains:

- Home timeline
- Local timeline
- Federated timeline

The home timeline is built from followed accounts, both local and remote.

---

## Federation Service

Responsible for:

- Sending activities
- Receiving remote activities
- Signature verification
- Retry logic
- Deduplication
- Queue management

This is the heart of a decentralized platform.

---

# 4. Federation Model

Suppose:

```text
Alice@instanceA.com

follows

Bob@instanceB.com
```

Flow:

```text
Alice follows Bob

↓

Instance A sends Follow activity

↓

Instance B verifies request

↓

Bob accepts

↓

Future posts are delivered to Instance A

↓

Alice sees Bob's posts
```

Instead of clients polling every remote instance, servers exchange activities asynchronously.

---

# 5. Activity Flow

### Posting

```text
Alice creates post

↓

Store locally

↓

Generate Create activity

↓

Identify remote followers

↓

Send activities

↓

Remote instances validate

↓

Store locally

↓

Update followers' timelines
```

This push-based model reduces cross-instance read traffic.

---

# 6. Timeline Generation

## Fan-out on Write

When Alice posts:

```text
Followers

↓

Timeline updates immediately
```

Advantages:

- Fast timeline reads
- Good for typical social graphs

Disadvantages:

- Expensive for celebrity accounts

---

## Fan-out on Read

Store posts once.

Generate timelines dynamically.

Advantages:

- Cheap writes
- Less duplication

Disadvantages:

- Slower reads

---

### Practical Hybrid

Use:

- Fan-out on write for normal users
- Fan-out on read for high-follower ("celebrity") accounts

This balances write amplification with read performance.

---

# 7. Federation Queue

Federation should be asynchronous.

```text
Create Post

↓

Message Queue

↓

Federation Workers

↓

Remote Instances
```

Benefits:

- Decouples posting from delivery
- Handles retries
- Absorbs traffic spikes
- Improves reliability

---

# 8. Data Storage

| Data             | Storage                  |
| ---------------- | ------------------------ |
| Users            | SQL                      |
| Posts            | SQL / Distributed DB     |
| Followers        | Graph-friendly tables    |
| Timeline cache   | Redis                    |
| Media            | Object Storage           |
| Search index     | Elasticsearch/OpenSearch |
| Federation queue | Kafka/RabbitMQ           |

---

# 9. Media Handling

Media files are stored separately.

```text
Upload

↓

Object Storage

↓

CDN

↓

Referenced by posts
```

Remote instances may:

- Cache media
- Proxy media through their own servers
- Fetch on demand

This reduces repeated downloads and improves privacy.

---

# 10. Consistency Model

Strong consistency across all instances is impractical.

Instead:

- Local writes are immediately consistent.
- Remote updates propagate asynchronously.
- Temporary differences between instances are expected.

Examples:

- Likes may appear seconds later.
- Profile changes propagate gradually.
- Deleted posts may persist briefly on remote caches.

Eventual consistency provides better availability and scalability.

---

# 11. Scaling Strategy

## Horizontal Scaling

Each instance scales independently.

```text
API Servers

↓

Load Balancer

↓

Database

↓

Workers
```

---

## Cache

Redis caches:

- Timelines
- Profiles
- Trending hashtags

---

## Queue Workers

Dedicated workers handle:

- Federation delivery
- Media processing
- Notifications
- Search indexing

---

## CDN

Static assets:

- Images
- Videos
- Avatars

served through CDNs.

---

# 12. Security / Observability

## Security

- OAuth/OpenID Connect for clients
- HTTPS/TLS for all communication
- HTTP Signatures or signed requests for federation
- Input validation and sanitization
- Rate limiting
- Spam detection
- Per-instance block and allow lists
- Secure media handling and virus scanning

## Observability

Monitor:

- Federation queue length
- Delivery success/failure rates
- Remote instance response times
- Timeline generation latency
- API P95/P99 latency
- Database load
- Cache hit ratio
- Worker utilization

Distributed tracing helps diagnose delays in federation and timeline updates.

---

# 13. Trade-offs

| Decision             | Pros                                   | Cons                                     |
| -------------------- | -------------------------------------- | ---------------------------------------- |
| Federation           | No central point of failure or control | Operational complexity                   |
| Eventual consistency | High availability and resilience       | Temporary inconsistencies                |
| Fan-out on write     | Fast timeline reads                    | Expensive for users with many followers  |
| Fan-out on read      | Efficient writes                       | Higher read latency                      |
| Local moderation     | Community autonomy                     | Inconsistent moderation across instances |
| Media caching        | Lower latency and bandwidth            | Additional storage requirements          |

---

# Interview-ready summary

> "I would design a Mastodon-like platform as a federation of independent instances. Each instance owns its users, posts, moderation policies, and databases while communicating with other instances through a federation protocol like ActivityPub. Local writes are processed immediately, and activities such as follows, posts, likes, and boosts are delivered asynchronously through queues, providing eventual consistency. Timelines use a hybrid fan-out strategy, media is stored in object storage and served via CDNs, and each instance scales independently with stateless services, caches, and background workers. This architecture provides decentralization, fault isolation, and interoperability while allowing each community to govern itself."

## Question 5. What are hybrid consistency models in databases?

# Hybrid Consistency Models in Databases

## Direct answer

A **hybrid consistency model** combines multiple consistency guarantees within the same database or system, allowing different operations or data to use different consistency levels based on their requirements.

Instead of enforcing **strong consistency everywhere** or **eventual consistency everywhere**, hybrid models let you make trade-offs between **latency, availability, and correctness** on a per-operation, per-table, or per-transaction basis.

This is the approach used by many modern distributed databases because different workloads have different consistency needs.

---

# Why hybrid consistency?

No single consistency model is optimal for every use case.

For example, in an e-commerce application:

- **Payment processing** requires **strong consistency**.
- **Product recommendations** can tolerate **eventual consistency**.
- **View counts** may use **causal** or **session consistency**.

Using strong consistency for all operations would unnecessarily increase latency and reduce availability.

---

# Common consistency models

| Model                | Guarantee                               | Typical Use Case           |
| -------------------- | --------------------------------------- | -------------------------- |
| Strong Consistency   | Always reads the latest committed value | Banking, inventory         |
| Eventual Consistency | Replicas converge over time             | Social media feeds, caches |
| Read-Your-Writes     | A user always sees their own writes     | User profiles              |
| Monotonic Reads      | Reads never go backward in time         | Dashboards                 |
| Causal Consistency   | Preserves cause-and-effect ordering     | Messaging, comments        |
| Session Consistency  | Guarantees consistency within a session | Web applications           |

A hybrid system may support several of these simultaneously.

---

# Example architecture

```text
               Client
                  |
           Database Router
                  |
      +-----------+-----------+
      |                       |
 Strong Consistency      Eventual Consistency
     Partition              Partition
      |                       |
  Raft/Paxos             Async Replication
```

Different requests are routed based on their consistency requirements.

---

# Approaches to hybrid consistency

## 1. Per-operation consistency

The client specifies the required consistency level for each request.

Example:

```text
Read account balance
→ Strong consistency

Read product recommendations
→ Eventual consistency
```

This gives applications fine-grained control.

---

## 2. Per-table or per-collection consistency

Different datasets use different replication strategies.

Example:

| Table           | Consistency         |
| --------------- | ------------------- |
| Accounts        | Strong              |
| Orders          | Strong              |
| User Sessions   | Session consistency |
| Analytics       | Eventual            |
| Recommendations | Eventual            |

This is common in large microservice architectures.

---

## 3. Tunable consistency

Some databases let clients choose how many replicas participate in reads and writes.

Example:

- Write to **1 replica** → low latency
- Write to **majority** → stronger consistency
- Read from **leader** → latest data
- Read from **nearest replica** → lower latency

This allows balancing latency and consistency dynamically.

---

## 4. Transaction-level consistency

Some systems allow:

```text
Transaction A
Strong consistency

Transaction B
Eventual consistency
```

Critical transactions use distributed consensus, while non-critical operations use asynchronous replication.

---

# Real-world examples

| System               | Hybrid Consistency Approach                                    |
| -------------------- | -------------------------------------------------------------- |
| Google Cloud Spanner | Strong transactions with read-only stale reads                 |
| Amazon DynamoDB      | Per-request strongly consistent or eventually consistent reads |
| Apache Cassandra     | Tunable consistency (ONE, QUORUM, ALL, etc.)                   |
| Azure Cosmos DB      | Multiple selectable consistency levels                         |
| CockroachDB          | Strong transactions with follower/stale reads                  |

---

# Design considerations

## Use strong consistency for

- Financial transactions
- Inventory management
- Authentication
- Distributed locks
- Configuration data

These operations require correctness over latency.

---

## Use eventual consistency for

- News feeds
- Likes
- Analytics
- Metrics
- Search indexes
- Recommendations

These can tolerate temporary stale data.

---

## Combine them

Example:

```text
User places an order
        |
        +--> Payment → Strong
        |
        +--> Inventory → Strong
        |
        +--> Recommendation update → Eventual
        |
        +--> Analytics event → Eventual
        |
        +--> Notification → Eventual
```

This reduces latency while preserving correctness where it matters.

---

# Trade-offs

| Approach                        | Pros                                 | Cons                                                 |
| ------------------------------- | ------------------------------------ | ---------------------------------------------------- |
| Strong consistency everywhere   | Simple correctness model             | Higher latency, lower availability during partitions |
| Eventual consistency everywhere | High availability and low latency    | Stale reads, conflict resolution required            |
| Hybrid consistency              | Balances correctness and performance | More complex application design and reasoning        |

---

# Interview-ready summary

> "Hybrid consistency models allow a distributed database to provide different consistency guarantees for different operations or datasets instead of enforcing a single global model. Critical operations like payments or inventory updates typically use strong consistency, while non-critical workloads such as analytics, feeds, and recommendations use eventual consistency. Many modern databases also support per-request or tunable consistency levels, enabling applications to balance latency, availability, and correctness based on business requirements. This flexibility is why hybrid consistency has become the preferred approach in large-scale distributed systems."

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
