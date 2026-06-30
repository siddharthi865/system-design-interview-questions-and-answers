# Set 12

| S.No. | Question                                                                                                                     |
| ----- | ---------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you design a notification retry mechanism?](#question-1-how-do-you-design-a-notification-retry-mechanism)            |
| 2.    | [How do you design a spell checker system?](#question-2-how-do-you-design-a-spell-checker-system)                            |
| 3.    | [How do you design a plagiarism checker?](#question-3-how-do-you-design-a-plagiarism-checker)                                |
| 4.    | [How do you design a news feed ranking system?](#question-4-how-do-you-design-a-news-feed-ranking-system)                    |
| 5.    | [How do you design a comment threading system?](#question-5-how-do-you-design-a-comment-threading-system)                    |
| 6.    | [How do you design a shopping cart system?](#question-6-how-do-you-design-a-shopping-cart-system)                            |
| 7.    | [How do you design a dynamic pricing engine?](#question-7-how-do-you-design-a-dynamic-pricing-engine)                        |
| 8.    | [How do you design a taxi fare calculator system?](#question-8-how-do-you-design-a-taxi-fare-calculator-system)              |
| 9.    | [How do you design a leaderboard system for games?](#question-9-how-do-you-design-a-leaderboard-system-for-games)            |
| 10.   | [How do you design a coupon/voucher redemption system?](#question-10-how-do-you-design-a-couponvoucher-redemption-system)    |
| 11.   | [What is write amplification in storage systems?](#question-11-what-is-write-amplification-in-storage-systems)               |
| 12.   | [What are secondary indexes in NoSQL?](#question-12-what-are-secondary-indexes-in-nosql)                                     |
| 13.   | [What is a time-series database (TSDB)?](#question-13-what-is-a-time-series-database-tsdb)                                   |
| 14.   | [How do you design a database for IoT data?](#question-14-how-do-you-design-a-database-for-iot-data)                         |
| 15.   | [What is cold storage vs hot storage?](#question-15-what-is-cold-storage-vs-hot-storage)                                     |
| 16.   | [What is a columnar database? Give examples](#question-16-what-is-a-columnar-database-give-examples)                         |
| 17.   | [What is a key-value store?](#question-17-what-is-a-key-value-store)                                                         |
| 18.   | [How do you design a database for full-text search?](#question-18-how-do-you-design-a-database-for-full-text-search)         |
| 19.   | [What is an LSM tree (Log-Structured Merge Tree)?](#question-19-what-is-an-lsm-tree-log-structured-merge-tree)               |
| 20.   | [How do you prevent data skew in distributed databases?](#question-20-how-do-you-prevent-data-skew-in-distributed-databases) |

## Question 1. How do you design a notification retry mechanism?

# How do you design a notification retry mechanism?

## Direct answer

A **notification retry mechanism** ensures that notifications (email, SMS, push, webhook, etc.) are eventually delivered despite temporary failures. The design should:

- Retry only **retryable failures** (timeouts, 5xx, rate limits)
- Use **exponential backoff with jitter**
- Store retry state persistently
- Limit retries with a maximum retry count or time window
- Support **dead-letter queues (DLQ)** for permanently failed notifications
- Ensure **idempotency** to avoid duplicate deliveries
- Continuously monitor retry success and failure metrics

The goal is to maximize delivery while avoiding overwhelming downstream services.

---

# Requirements / Problem Framing

### Functional Requirements

- Send notifications asynchronously
- Retry failed deliveries
- Support multiple channels (Email, SMS, Push)
- Prevent duplicate notifications
- Handle temporary and permanent failures
- Allow manual replay of failed notifications

### Non-functional Requirements

- High reliability
- Scalable to millions of notifications
- Fault tolerant
- Observable
- Configurable retry policies

---

# High-Level Architecture

```text
              User Action
                    │
                    ▼
          Notification Service
                    │
                    ▼
             Message Queue
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
    Notification Worker   Retry Worker
          │                   │
          ▼                   │
   Email/SMS/Push Provider    │
          │                   │
      Success?                │
      │      │                │
     Yes     No───────────────┘
      │
      ▼
Mark Delivered

If retries exhausted
        │
        ▼
 Dead Letter Queue (DLQ)
```

---

# Retry Flow

### Step 1: Notification created

```text
Notification
Status = PENDING
Retry Count = 0
```

Worker picks it up.

---

### Step 2: Delivery succeeds

```text
Status = DELIVERED
```

Done.

---

### Step 3: Temporary failure

Examples:

- Network timeout
- HTTP 500
- SMTP unavailable
- Push provider unavailable

Worker schedules retry.

```text
Retry Count = Retry Count + 1

Next Retry Time =
Current Time + Backoff Delay
```

---

### Step 4: Permanent failure

Examples:

- Invalid email
- Invalid phone number
- User blocked notifications
- HTTP 400 validation error

No retry.

```text
Status = FAILED
```

or

```text
Move to DLQ
```

---

# Retry Policy

Typical retry schedule:

| Retry | Delay      |
| ----- | ---------- |
| 1     | 1 minute   |
| 2     | 5 minutes  |
| 3     | 15 minutes |
| 4     | 1 hour     |
| 5     | 6 hours    |
| 6     | 24 hours   |

After maximum retries:

```text
FAILED
```

or

```text
Dead Letter Queue
```

---

# Exponential Backoff

Instead of retrying immediately:

```text
1 sec
2 sec
4 sec
8 sec
16 sec
32 sec
```

Formula:

```text
delay = base × 2^retryCount
```

Example:

```
Base = 5 sec

Retry 1 → 5 sec
Retry 2 → 10 sec
Retry 3 → 20 sec
Retry 4 → 40 sec
```

Benefits:

- Reduces load
- Gives providers time to recover
- Prevents retry storms

---

# Why Add Jitter?

Without jitter:

```text
100,000 failed notifications

↓

All retry after exactly 60 seconds

↓

Huge traffic spike
```

With jitter:

```text
Retry between

55–65 sec

or

60–90 sec
```

Traffic becomes evenly distributed.

Example:

```text
delay = exponential_backoff + random(0–10 sec)
```

---

# Retryable vs Non-Retryable Errors

| Error                  | Retry?        |
| ---------------------- | ------------- |
| Network timeout        | ✅ Yes        |
| HTTP 500               | ✅ Yes        |
| Rate limit (429)       | ✅ Yes        |
| SMTP unavailable       | ✅ Yes        |
| Invalid email          | ❌ No         |
| Invalid phone          | ❌ No         |
| Authentication failure | ❌ Usually No |
| Bad request (400)      | ❌ No         |

A key design decision is classifying errors correctly so resources aren't wasted retrying requests that can never succeed.

---

# Data Model

```text
Notification

id
userId
channel
payload
status
retryCount
maxRetries
nextRetryAt
lastError
createdAt
updatedAt
```

Indexes:

```
(status, nextRetryAt)
```

Allows efficient lookup:

```sql
WHERE status = 'RETRY'
AND nextRetryAt <= NOW()
```

---

# Retry Queue

Instead of scanning the database constantly:

```text
Notification

↓

Retry Queue

↓

Retry Worker

↓

Provider
```

Possible implementations:

- Delayed queues
- Scheduled jobs
- Priority queues

This scales better than polling large tables.

---

# Dead Letter Queue (DLQ)

When retries are exhausted:

```text
Notification

↓

DLQ
```

Reasons:

- Permanent failure
- Too many retries
- Corrupted payload
- Unexpected exceptions

Operators can:

- Inspect failures
- Replay messages
- Fix bugs
- Alert support teams

---

# Idempotency

A worker may crash after the provider accepts the notification but before the success is recorded:

```text
Worker

↓

Provider accepted

↓

Worker crashes

↓

Retry happens

↓

Duplicate notification
```

Solutions:

- Generate an **idempotency key** per notification.
- Include it in requests to providers that support idempotent operations.
- Record completed notification IDs locally and ignore duplicate processing.

---

# Scaling Strategy

### Queue-based architecture

```text
Producer

↓

Kafka / RabbitMQ / SQS

↓

Multiple Notification Workers
```

Workers scale horizontally.

---

### Partitioning

Partition by:

- User ID
- Notification ID
- Channel

Benefits:

- Parallel processing
- Better throughput
- Reduced contention

---

### Separate workers by channel

```text
Email Worker

SMS Worker

Push Worker
```

Advantages:

- Independent scaling
- Different retry policies
- Isolation from channel-specific outages

---

# Observability

Track metrics such as:

- Notifications sent
- Retry count
- Retry success rate
- Average retries per notification
- Time to delivery
- DLQ size
- Provider error rate
- Queue length
- Queue processing latency

Set alerts when:

- Retry rate spikes
- DLQ grows rapidly
- Queue backlog increases
- Delivery latency exceeds thresholds

---

# Trade-offs

| Approach                     | Advantages                                       | Disadvantages                              |
| ---------------------------- | ------------------------------------------------ | ------------------------------------------ |
| Immediate retry              | Simple                                           | Can overwhelm providers during outages     |
| Fixed delay                  | Easy to implement                                | Less adaptive to varying failure durations |
| Exponential backoff          | Reduces load and improves recovery               | Higher latency for eventual delivery       |
| Exponential backoff + jitter | Best scalability and avoids synchronized retries | Slightly more complex                      |
| Database polling             | Simple                                           | Inefficient at scale                       |
| Delayed message queues       | Efficient and scalable                           | Requires messaging infrastructure          |

---

# Interview-ready summary

"I would implement notifications asynchronously using a message queue and worker processes. Workers classify failures into retryable and non-retryable categories. Retryable failures are rescheduled using exponential backoff with jitter, while permanent failures are marked as failed immediately. Retry state, counts, and the next retry time are stored persistently, and notifications exceeding retry limits are moved to a Dead Letter Queue for investigation or replay. The system uses idempotency keys to prevent duplicate deliveries, scales horizontally with multiple workers, and is monitored through metrics such as retry rate, delivery latency, queue depth, and DLQ size."

## Question 2. How do you design a spell checker system?

# How do you design a spell checker system?

## Direct answer

A spell checker system detects misspelled words and suggests the most likely corrections. At scale (e.g., in search engines, word processors, or messaging apps), the system combines:

- A **dictionary** (lexicon of valid words)
- **Efficient lookup structures** (Trie, Hash Set, BK-Tree)
- **Approximate string matching** algorithms (Edit Distance/Levenshtein Distance)
- **Ranking** using word frequency, context, and user behavior
- **Machine Learning/Language Models** for context-aware corrections

The primary challenge is balancing **accuracy**, **low latency**, and **memory efficiency**.

---

# Requirements / Problem Framing

### Functional Requirements

- Detect misspelled words
- Suggest one or more corrections
- Support multiple languages
- Learn new words (custom dictionary)
- Handle context-aware corrections (optional)

### Non-functional Requirements

- Response latency < 50 ms
- High accuracy
- High availability
- Scalable to millions of requests/day
- Efficient memory usage

---

# High-Level Architecture

```text
                 User Types Text
                        │
                        ▼
                Text Processing Service
                        │
          ┌─────────────┴─────────────┐
          ▼                           ▼
     Tokenization               Normalization
          │                           │
          └─────────────┬─────────────┘
                        ▼
                Dictionary Lookup
                        │
          ┌─────────────┴─────────────┐
          ▼                           ▼
      Word Exists?               Misspelled
          │                           │
         Yes                          ▼
          │                 Candidate Generator
          │                           │
          ▼                           ▼
      Return Word             Candidate Ranking
                                        │
                                        ▼
                              Best Suggestions
```

---

# Core Components

### 1. Dictionary

Stores valid words.

Example:

```text
apple
application
banana
computer
database
system
```

Implementation options:

| Structure    | Lookup   | Memory                   | Notes                 |
| ------------ | -------- | ------------------------ | --------------------- |
| Hash Set     | O(1)     | Higher                   | Fast exact lookup     |
| Trie         | O(L)     | Efficient prefix sharing | Supports autocomplete |
| Sorted Array | O(log N) | Low                      | Limited functionality |

For spell checking, a **Trie** or **Hash Set** is commonly used.

---

### 2. Tokenizer

Splits text into words.

Example:

```text
Input:

"I love applle pie."

↓

["I", "love", "applle", "pie"]
```

---

### 3. Normalization

Convert text into a standard form.

Examples:

```text
Apple
APPLE
apple

↓

apple
```

Also handles:

- Unicode normalization
- Removing punctuation
- Accent normalization (optional)

---

# Detecting Misspellings

Lookup:

```text
Dictionary.contains(word)
```

Example:

```text
apple

Exists?

Yes
```

Example:

```text
applle

Exists?

No
```

Generate suggestions.

---

# Candidate Generation

Several approaches are used.

## 1. Edit Distance (Levenshtein Distance)

Measures the minimum edits needed.

Allowed operations:

- Insert
- Delete
- Replace

Example:

```text
applle

↓

apple

Delete 'l'

Distance = 1
```

Another:

```text
boook

↓

book

Distance = 1
```

Time complexity:

```text
O(m × n)
```

where _m_ and _n_ are word lengths.

---

## 2. BK-Tree (Burkhard-Keller Tree)

Designed for approximate string matching.

Example:

```text
apple
apply
ape
apricot
banana
```

Instead of comparing against every word, the BK-Tree prunes large parts of the search space using edit distance, making candidate retrieval much faster for large dictionaries.

Advantages:

- Efficient fuzzy search
- Scales to large vocabularies

---

## 3. Trie-Based Search

Useful when the typo is near the beginning of the word.

Example:

```text
appl

↓

apple
application
apply
```

Excellent for prefix-based suggestions and autocomplete.

---

# Ranking Suggestions

Many candidates may have similar edit distances.

Example:

```text
appl

↓

apple
apply
ample
```

Ranking signals:

| Signal             | Example                             |
| ------------------ | ----------------------------------- |
| Edit distance      | Smaller is better                   |
| Word frequency     | "apple" is more common than "ample" |
| User history       | Frequently typed words rank higher  |
| Domain dictionary  | Medical/legal vocabulary            |
| Keyboard proximity | Nearby keys are common typo sources |

Example:

```text
Input:

applr

Suggestions:

apple
apply
applier
```

"apple" ranks highest because it is both close in edit distance and more frequently used.

---

# Context-Aware Spell Checking

Simple spell checkers process one word at a time.

Example:

```text
I ate bred.
```

Possible corrections:

```text
bred
bread
breed
```

Without context, either could be valid.

With a language model:

```text
I ate bread.
```

is much more probable than:

```text
I ate breed.
```

Modern systems use n-gram models or transformer-based language models to improve suggestion quality.

---

# Data Storage

```text
Dictionary

word
frequency
language
lastUpdated
```

Optional:

```text
CustomDictionary

userId
word
```

Allows user-specific additions.

---

# Scaling Strategy

### Sharding

Partition by:

- Language
- First character
- Hash of the word

Example:

```text
English Dictionary

Shard A-M

Shard N-Z
```

---

### Caching

Frequently checked words:

```text
the
computer
system
database
```

Store in an in-memory cache to avoid repeated lookups.

---

### Horizontal Scaling

```text
Load Balancer
        │
 ┌──────┴──────┐
 ▼             ▼
Spell Server  Spell Server
        │
        ▼
 Shared Dictionary Store
```

Since dictionary lookups are mostly read-only, servers scale easily by adding more instances.

---

# Trade-offs

| Approach                 | Advantages                     | Disadvantages                    |
| ------------------------ | ------------------------------ | -------------------------------- |
| Hash Set                 | Fast exact lookup              | No fuzzy search                  |
| Trie                     | Prefix search, autocomplete    | Higher implementation complexity |
| BK-Tree                  | Efficient approximate matching | More memory usage                |
| Levenshtein on all words | Accurate                       | Too slow for large dictionaries  |
| ML-based ranking         | Best accuracy                  | Higher latency and compute cost  |

---

# Security / Observability

Monitor:

- Request latency
- Suggestion accuracy
- Cache hit ratio
- Dictionary update failures
- Requests per second
- Top misspelled words
- Error rates

Protect against abuse with:

- Rate limiting
- Input length validation
- Logging and tracing for slow requests

---

# Interview-ready summary

"A scalable spell checker first normalizes and tokenizes input, then performs a fast dictionary lookup using a Trie or Hash Set. If a word is missing, it generates candidate corrections using approximate matching techniques like Levenshtein Distance or a BK-Tree. The candidates are ranked based on edit distance, word frequency, keyboard proximity, and optionally contextual language models. The service is largely read-heavy, making it easy to scale horizontally with caching, dictionary sharding, and replicated read-only dictionaries, while keeping response latency under a few tens of milliseconds."

## Question 3. How do you design a plagiarism checker?

# How do you design a plagiarism checker?

## Direct answer

A plagiarism checker compares a submitted document against a large corpus of documents to detect copied or highly similar content. Since comparing every document character-by-character is computationally expensive, modern systems use a multi-stage pipeline:

1. **Preprocess** the document (normalize and tokenize).
2. **Generate fingerprints** using techniques like **shingling** and **MinHash/SimHash**.
3. **Retrieve candidate documents** using an inverted index or Locality Sensitive Hashing (LSH).
4. **Perform detailed similarity comparison** only on the candidates.
5. **Generate a plagiarism report** with matching passages and similarity scores.

This approach scales to millions of documents while maintaining low latency.

---

# Requirements / Problem Framing

### Functional Requirements

- Upload documents (PDF, DOCX, TXT, etc.)
- Detect exact and near-duplicate plagiarism
- Highlight copied passages
- Compute an overall similarity score
- Compare against a large document repository
- Support adding new documents continuously

### Non-functional Requirements

- High accuracy
- Scalable to millions of documents
- Fast response (seconds rather than minutes)
- High availability
- Efficient storage and indexing

---

# High-Level Architecture

```text
                 User Uploads Document
                          │
                          ▼
                 Document Processing
                          │
          ┌───────────────┴───────────────┐
          ▼                               ▼
    Text Extraction                 Metadata Extraction
          │
          ▼
   Normalization & Tokenization
          │
          ▼
      Shingle Generation
          │
          ▼
 Fingerprint (MinHash/SimHash)
          │
          ▼
 Candidate Retrieval (LSH / Inverted Index)
          │
          ▼
 Detailed Similarity Engine
          │
          ▼
      Report Generator
          │
          ▼
      Similarity Report
```

---

# Step 1: Text Extraction

Extract plain text from different formats.

Supported formats:

- PDF
- DOCX
- HTML
- TXT
- Markdown

Example:

```text
Input PDF

↓

Extracted Text
```

---

# Step 2: Normalization

Normalize the text before comparison.

Example:

```text
"The Quick Brown Fox!"

↓

the quick brown fox
```

Typical normalization includes:

- Lowercasing
- Removing punctuation
- Removing extra whitespace
- Unicode normalization
- Optional stop-word removal
- Optional stemming/lemmatization

---

# Step 3: Shingling

Instead of comparing individual words, split the document into overlapping sequences of **k words** (k-shingles).

Example (k = 3):

```text
Document:

the quick brown fox jumps

Shingles:

the quick brown
quick brown fox
brown fox jumps
```

Benefits:

- Detects copied phrases
- Robust to small edits
- Preserves local context

---

# Step 4: Fingerprinting

Comparing all shingles from all documents is expensive.

Instead, generate compact fingerprints.

## MinHash

Produces a compact signature that approximates the similarity between sets of shingles.

Advantages:

- Excellent for estimating **Jaccard Similarity**
- Reduces storage requirements
- Enables fast comparisons

---

## SimHash

Represents a document as a binary fingerprint.

Advantages:

- Efficient near-duplicate detection
- Small memory footprint
- Hamming distance measures similarity

---

# Step 5: Candidate Retrieval

Comparing against every stored document is infeasible.

Instead:

```text
Document

↓

Fingerprint

↓

Locality Sensitive Hashing (LSH)

↓

Candidate Documents
```

LSH groups similar fingerprints into the same buckets, dramatically reducing the number of comparisons.

Alternative approach:

Use an **inverted index** mapping shingles to the documents containing them.

---

# Step 6: Detailed Similarity Comparison

Once candidate documents are identified, perform more accurate comparisons.

Common similarity metrics:

### Jaccard Similarity

```
Similarity =
Intersection(Set A, Set B)
---------------------------
Union(Set A, Set B)
```

Example:

```
Doc A shingles = 100

Doc B shingles = 90

Common = 80

Similarity = 80 / 110 = 72.7%
```

---

### Cosine Similarity

Convert documents into TF-IDF vectors and compute the cosine of the angle between them.

Useful for:

- Semantic similarity
- Documents with different lengths

---

### Edit Distance

Useful for detecting:

- Small modifications
- Typographical changes

Usually applied only to matching passages because it is computationally expensive.

---

# Highlight Matching Sections

After detecting similar documents:

```text
Student Document

↓

Sentence Alignment

↓

Matching Passages

↓

Highlighted Report
```

Example:

```text
Original:
Distributed systems improve scalability.

Copied:
Distributed systems improve scalability.
```

The report highlights the matching text and its source.

---

# Data Model

### Documents

| Field      | Description     |
| ---------- | --------------- |
| documentId | Unique ID       |
| title      | Document title  |
| text       | Normalized text |
| language   | Language        |
| uploadedAt | Timestamp       |

### Fingerprints

| Field      | Description     |
| ---------- | --------------- |
| documentId | Reference       |
| signature  | MinHash/SimHash |
| shingles   | Optional        |

### Inverted Index

```
shingle

↓

documentIds
```

Example:

```text
"distributed systems"

↓

Doc12
Doc45
Doc102
```

---

# Scaling Strategy

## Distributed Storage

```
Document Store

Shard 1
Shard 2
Shard 3
```

Shard by:

- Document ID
- Language
- Hash of document

---

## Distributed Index

Maintain distributed inverted indexes.

```
Shard A

distributed

↓

Doc1
Doc8

Shard B

database

↓

Doc9
Doc20
```

Each node stores only part of the index.

---

## Parallel Similarity Computation

```
Candidate Documents

↓

Worker Pool

↓

Similarity Scores
```

Each worker compares a subset of candidate documents.

---

## Incremental Indexing

When a new document arrives:

```
Upload

↓

Extract

↓

Fingerprint

↓

Update Index
```

Avoid rebuilding the entire index.

---

# Performance Optimizations

- Cache frequently queried documents
- Compress fingerprints
- Store only fingerprints in memory
- Batch index updates
- Parallelize similarity computation
- Precompute signatures offline

---

# Trade-offs

| Approach              | Advantages                 | Disadvantages                              |
| --------------------- | -------------------------- | ------------------------------------------ |
| Exact text comparison | Very accurate              | Doesn't scale                              |
| Shingling             | Detects copied phrases     | Larger index size                          |
| MinHash               | Fast similarity estimation | Approximate                                |
| SimHash               | Compact fingerprints       | Less precise for set similarity            |
| Inverted Index        | Fast candidate lookup      | Higher storage overhead                    |
| LSH                   | Excellent scalability      | May miss some edge cases (false negatives) |

---

# Security / Observability

Security:

- Virus scan uploaded files
- Validate supported formats
- Encrypt stored documents
- Access control for private repositories

Observability:

Monitor:

- Upload rate
- Processing latency
- Index update latency
- Candidate retrieval time
- Similarity computation latency
- False positive/negative rates
- Worker queue depth

---

# Interview-ready summary

> "I would design a plagiarism checker as a multi-stage pipeline. Documents are first normalized and converted into overlapping word shingles. Compact fingerprints such as MinHash or SimHash are generated, and candidate documents are retrieved efficiently using an inverted index or Locality Sensitive Hashing instead of scanning the entire corpus. Only those candidates undergo detailed similarity checks using metrics like Jaccard or cosine similarity. Matching passages are then highlighted in a report. The system scales by sharding document storage and indexes, processing comparisons in parallel, and incrementally updating fingerprints as new documents are added."

## Question 4. How do you design a news feed ranking system?

# How do you design a news feed ranking system?

## Direct answer

A news feed ranking system determines **which posts a user sees and in what order**. Since a user may be eligible to see thousands of posts, the system uses a **multi-stage ranking pipeline**:

1. **Candidate Generation** – Retrieve potentially relevant posts.
2. **Filtering** – Remove blocked, muted, duplicate, or policy-violating content.
3. **Ranking** – Score each candidate using ML models and ranking signals.
4. **Re-ranking** – Improve diversity, freshness, and business objectives.
5. **Feed Delivery** – Return the top N posts with low latency.

The challenge is balancing **relevance, freshness, engagement, fairness, diversity, and latency**.

---

# Requirements / Problem Framing

### Functional Requirements

- Generate a personalized feed for each user
- Rank posts by relevance
- Support photos, videos, and text
- Handle likes, comments, shares, and follows
- Continuously update the feed
- Support advertisements and sponsored posts

### Non-functional Requirements

- Feed latency < 200 ms
- High availability
- Scalable to billions of posts
- Personalized ranking
- Fault tolerant

---

# High-Level Architecture

```text
                  User Opens App
                         │
                         ▼
                 Feed Request API
                         │
                         ▼
              Candidate Generation
                         │
                         ▼
                 Filtering Pipeline
                         │
                         ▼
                 Ranking Service
                         │
                         ▼
                Re-ranking Service
                         │
                         ▼
                  Top N Feed Items
                         │
                         ▼
                     User Feed
```

---

# Step 1: Candidate Generation

The system first gathers a manageable set of posts.

Possible sources:

- Friends/following
- Groups
- Pages
- Trending posts
- Recommended creators
- Advertisements

Example:

```
Possible posts:

1. Friend A
2. Friend B
3. Group X
4. Sports Page
5. Trending News
6. Suggested Creator
```

Instead of searching billions of posts, retrieve only a few thousand relevant candidates.

Techniques:

- Fan-out on write
- Fan-out on read
- Graph-based retrieval
- Recommendation models

---

# Step 2: Filtering

Remove posts the user should not see.

Filters include:

- Blocked users
- Muted accounts
- Deleted posts
- Already viewed posts
- Spam
- Policy violations
- Age or region restrictions

Example:

```
1000 candidate posts

↓

850 valid posts
```

---

# Step 3: Ranking

Each post receives a relevance score.

Example:

```
Score(Post) =

0.30 × Engagement
+0.25 × Relationship
+0.20 × Freshness
+0.15 × Content Quality
+0.10 × Personal Interest
```

In production, these scores are typically generated by machine learning models rather than a simple formula.

---

# Ranking Signals

## User Signals

- Follow relationships
- Past likes
- Watch history
- Search history
- Session behavior
- Interests
- Location (when relevant)

---

## Content Signals

- Number of likes
- Comments
- Shares
- Watch time
- Click-through rate
- Topic
- Language

---

## Freshness

Recent posts usually rank higher.

Example:

```
10 minutes ago

>

2 days ago
```

Freshness often decays over time.

---

## Relationship Strength

Example:

```
User frequently interacts with Alice.

↓

Alice's posts receive a higher score.
```

Interaction signals:

- Messages
- Comments
- Likes
- Profile visits
- Tagged photos

---

## Quality Signals

Reduce low-quality content.

Signals:

- Spam probability
- Clickbait detection
- Misinformation score
- Toxicity score
- Duplicate content

---

# Machine Learning Ranking

Modern systems train models using historical interaction data.

Example features:

```
User Features
--------------
Age
Country
Interests
Watch Time

Post Features
--------------
Topic
Age
Popularity
Media Type

Interaction Features
--------------------
Friendship Strength
Previous Likes
Comment History
```

Possible models:

- Gradient Boosted Trees (e.g., XGBoost)
- Deep Neural Networks
- Learning-to-Rank models (RankNet, LambdaMART)
- Transformer-based recommendation models

The model outputs:

```
Probability(User interacts with Post)
```

This probability becomes the ranking score.

---

# Step 4: Re-ranking

The highest-scoring posts may still produce a poor experience.

Example:

```
Top 10

All sports

↓

Boring feed
```

Re-ranking introduces diversity.

Example:

```
Sports
Friend
Travel
Technology
Family
News
```

Goals:

- Topic diversity
- Creator diversity
- Fresh content
- Sponsored content placement
- Avoid duplicate stories

---

# Feed Caching

Popular feeds can be cached.

```
Feed Cache

↓

User Feed
```

Benefits:

- Lower latency
- Reduced database load

Cache invalidation occurs when:

- New posts arrive
- User follows someone
- User interacts with content

---

# Fan-out Strategies

## Fan-out on Write

```
User publishes

↓

Push post

↓

Followers' feed cache
```

Advantages:

- Fast feed reads
- Low read latency

Disadvantages:

- Expensive for users with millions of followers

---

## Fan-out on Read

```
User opens feed

↓

Fetch latest posts

↓

Rank

↓

Return feed
```

Advantages:

- Lower write cost
- Better for celebrities

Disadvantages:

- Higher read latency

---

## Hybrid Approach

Common in large social networks:

- Fan-out on write for regular users
- Fan-out on read for celebrities and high-follower accounts

This balances write amplification and read performance.

---

# Data Model

### Posts

| Field      | Description    |
| ---------- | -------------- |
| postId     | Unique ID      |
| authorId   | Creator        |
| content    | Text/media     |
| createdAt  | Timestamp      |
| visibility | Public/Friends |

### User Interactions

| Field     | Description             |
| --------- | ----------------------- |
| userId    | User                    |
| postId    | Post                    |
| action    | Like/Comment/Share/View |
| timestamp | Event time              |

These interaction logs also serve as training data for ranking models.

---

# Scaling Strategy

## Distributed Feed Service

```
Load Balancer
        │
 ┌──────┴──────┐
 ▼             ▼
Feed Server  Feed Server
        │
        ▼
Ranking Service
```

Scale horizontally by adding more feed servers.

---

## Partitioning

Shard data by:

- User ID
- Post ID
- Geographic region

---

## Event Streaming

```
Like

↓

Event Queue (e.g., Kafka)

↓

Feature Store

↓

ML Model Update
```

Streaming pipelines keep ranking features fresh.

---

## Feature Store

Maintain precomputed features such as:

- User interests
- Relationship strength
- Post popularity
- Historical engagement

This avoids expensive real-time computations.

---

# Trade-offs

| Approach           | Advantages             | Disadvantages               |
| ------------------ | ---------------------- | --------------------------- |
| Chronological feed | Simple and predictable | Less personalized           |
| Rule-based ranking | Easy to understand     | Lower relevance             |
| ML-based ranking   | High engagement        | Complex infrastructure      |
| Fan-out on write   | Fast reads             | Expensive writes            |
| Fan-out on read    | Cheap writes           | Higher read latency         |
| Hybrid fan-out     | Balanced scalability   | More operational complexity |

---

# Security / Observability

Security:

- Authentication and authorization
- Privacy controls (public, friends, private)
- Spam and abuse detection
- Content moderation
- Rate limiting

Observability:

Monitor:

- Feed generation latency
- Ranking latency
- Cache hit ratio
- Candidate retrieval time
- Click-through rate (CTR)
- Engagement (likes, comments, shares)
- Time spent on feed
- Error rates

A/B testing is essential to evaluate new ranking models before full rollout.

---

# Interview-ready summary

> "I would design a news feed ranking system using a multi-stage pipeline. Candidate generation retrieves a few thousand potentially relevant posts, filtering removes ineligible content, and a machine learning ranking model scores posts using signals such as user interests, relationship strength, engagement, freshness, and content quality. A re-ranking stage improves diversity and inserts sponsored content while respecting business rules. To scale, I'd use a hybrid fan-out strategy, distributed feed servers, caching, event streaming for real-time feature updates, and horizontally scalable ranking services. This architecture provides highly personalized feeds with low latency while supporting billions of users and posts."

## Question 5. How do you design a comment threading system?

# How do you design a comment threading system?

## Direct answer

A comment threading system organizes comments into a **hierarchical tree**, allowing users to reply to comments and create nested discussions. At scale (e.g., Reddit, YouTube, Facebook), the system should:

- Support top-level comments and nested replies
- Efficiently fetch an entire thread
- Handle millions of comments
- Support pagination for large discussions
- Rank comments (newest, oldest, most popular)
- Prevent excessively deep nesting for performance and usability

The key design challenge is storing and querying hierarchical data efficiently while keeping read latency low.

---

# Requirements / Problem Framing

### Functional Requirements

- Add top-level comments
- Reply to comments
- Fetch comment threads
- Edit/delete comments
- Like/upvote comments
- Sort comments (newest, oldest, top)
- Collapse/expand replies

### Non-functional Requirements

- Low read latency (<100 ms)
- High availability
- Scalable to millions of comments
- Consistent ordering
- Efficient pagination

---

# High-Level Architecture

```text
               User
                 │
                 ▼
          Comment API Service
                 │
     ┌───────────┴───────────┐
     ▼                       ▼
 Comment Database      Cache (Redis)
     │
     ▼
 Search / Analytics (Optional)
```

**Flow:**

1. User submits a comment or reply.
2. API validates and stores it.
3. Cache is updated or invalidated.
4. Clients fetch comments from cache/database.
5. Responses are assembled into a threaded tree.

---

# Data Model

A simple and scalable schema uses an adjacency list.

| Field           | Description                         |
| --------------- | ----------------------------------- |
| commentId       | Unique comment ID                   |
| postId          | Associated post/article             |
| parentCommentId | Parent comment (NULL for top-level) |
| authorId        | User ID                             |
| content         | Comment text                        |
| createdAt       | Timestamp                           |
| updatedAt       | Timestamp                           |
| score           | Likes/upvotes                       |
| replyCount      | Number of direct replies            |
| status          | Active/Deleted                      |

Example:

| commentId | parentCommentId |
| --------- | --------------- |
| C1        | NULL            |
| C2        | C1              |
| C3        | C1              |
| C4        | C2              |
| C5        | C4              |

Tree:

```text
C1
├── C2
│     └── C4
│            └── C5
└── C3
```

---

# Posting a Comment

### Top-level comment

```text
parentCommentId = NULL
```

### Reply

```text
parentCommentId = Parent ID
```

Store the reply and increment the parent's `replyCount`.

---

# Fetching Threads

A common approach:

1. Fetch top-level comments for a post.
2. Fetch replies separately.
3. Build the tree in memory.

```text
Post

↓

Top-Level Comments

↓

Replies

↓

Assemble Tree

↓

Return JSON
```

Example response:

```json
{
  "commentId": "C1",
  "content": "Great article!",
  "replies": [
    {
      "commentId": "C2",
      "content": "I agree!",
      "replies": []
    }
  ]
}
```

This minimizes recursive database queries.

---

# Sorting

Support multiple ordering strategies.

### Top-level comments

- Newest
- Oldest
- Most liked
- Most relevant

### Replies

Often sorted by:

- Oldest first (conversation flow)
- Newest first
- Most liked

Example:

```text
Top Comments

↓

Score DESC

↓

Newest Replies
```

---

# Pagination

Large threads can contain millions of comments.

Instead of loading everything:

```text
Top Comments

↓

First 20

↓

Load More
```

Replies can also be paginated:

```text
Comment

↓

View 15 More Replies
```

Cursor-based pagination is preferred over offset pagination for better scalability.

---

# Limiting Nesting Depth

Unlimited nesting creates:

- Slow rendering
- Complex queries
- Poor user experience

Example:

```text
Maximum Depth = 5

Depth 6

↓

Attach as child visually
or
Flatten into level 5
```

Many platforms cap nesting depth and continue replies under the deepest visible level.

---

# Caching

Popular discussions are read much more than written.

Cache:

- Top comments
- First page of replies
- Comment counts

```text
Client

↓

Redis

↓

Database
```

Invalidate or refresh cache when new comments are added.

---

# Scaling Strategy

## Database Sharding

Partition comments by:

- Post ID
- Hash(Post ID)

```text
Shard 1

Post 1-1000

Shard 2

Post 1001-2000
```

This keeps all comments for a post together, simplifying thread retrieval.

---

## Read Replicas

Heavy read traffic:

```text
Writes

↓

Primary DB

↓

Read Replicas

↓

Clients
```

Improves read scalability without affecting writes.

---

## Asynchronous Processing

Non-critical updates are processed asynchronously.

Examples:

- Notification delivery
- Search indexing
- Analytics
- Popularity score updates

```text
Comment Created

↓

Message Queue

↓

Notification Service
```

---

# Handling Deleted Comments

Avoid breaking thread structure.

Instead of removing a comment:

```text
[Deleted]
```

Children remain accessible.

Example:

```text
C1

↓

[Deleted]

↓

Replies remain visible
```

This preserves conversation context.

---

# Trade-offs

| Approach                           | Advantages                | Disadvantages                     |
| ---------------------------------- | ------------------------- | --------------------------------- |
| Adjacency List (`parentCommentId`) | Simple, easy writes       | Tree reconstruction required      |
| Nested Set Model                   | Fast subtree reads        | Expensive inserts/updates         |
| Materialized Path                  | Simple subtree queries    | Path updates on moves             |
| Closure Table                      | Fast hierarchical queries | More storage and write complexity |

**Interview tip:** For social media systems, the **Adjacency List** model is the most common choice because comments are inserted frequently and rarely moved.

---

# Security / Observability

### Security

- Authentication and authorization
- Spam detection
- Rate limiting
- Content moderation
- HTML/script sanitization to prevent XSS

### Observability

Monitor:

- Comment creation latency
- Thread retrieval latency
- Cache hit ratio
- Replies per comment
- Read/write QPS
- Database query latency
- Error rates

---

# Interview-ready summary

> "I would model comments using an adjacency list where each comment stores its parent comment ID. Top-level comments have a null parent, while replies reference their parent. To serve a thread efficiently, I'd fetch top-level comments and their replies, then assemble the hierarchy in memory. Popular threads would be cached, comments would be sharded by post ID, and read replicas would handle high read traffic. I'd use cursor-based pagination, cap nesting depth to keep queries efficient, preserve deleted comments as placeholders to maintain thread integrity, and process notifications and analytics asynchronously. This design provides scalable writes, efficient reads, and supports discussions with millions of comments."

## Question 6. How do you design a shopping cart system?

## Question 7. How do you design a dynamic pricing engine?

## Question 8. How do you design a taxi fare calculator system?

## Question 9. How do you design a leaderboard system for games?

## Question 10. How do you design a coupon/voucher redemption system?

## Question 11. What is write amplification in storage systems?

## Question 12. What are secondary indexes in NoSQL?

## Question 13. What is a time-series database (TSDB)?

## Question 14. How do you design a database for IoT data?

## Question 15. What is cold storage vs hot storage?

## Question 16. What is a columnar database? Give examples

## Question 17. What is a key-value store?

## Question 18. How do you design a database for full-text search?

## Question 19. What is an LSM tree (Log-Structured Merge Tree)?

## Question 20. How do you prevent data skew in distributed databases?
