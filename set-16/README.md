# Set 16

| S.No. | Question                                                                                                                                                   |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What are trade-offs between consistency and availability?](#question-1-what-are-trade-offs-between-consistency-and-availability)                          |
| 2.    | [What is sticky caching?](#question-2-what-is-sticky-caching)                                                                                              |
| 3.    | [What is a cold cache vs a warm cache?](#question-3-what-is-a-cold-cache-vs-a-warm-cache)                                                                  |
| 4.    | [What is a dead letter queue (DLQ)?](#question-4-what-is-a-dead-letter-queue-dlq)                                                                          |
| 5.    | [What is an append-only log in distributed systems?](#question-5-what-is-an-append-only-log-in-distributed-systems)                                        |
| 6.    | [Explain optimistic vs pessimistic locking](#question-6-explain-optimistic-vs-pessimistic-locking)                                                         |
| 7.    | [What is eventual leader election?](#question-7-what-is-eventual-leader-election)                                                                          |
| 8.    | [What is quorum-based replication?](#question-8-what-is-quorum-based-replication)                                                                          |
| 9.    | [What is hard state vs soft state in distributed systems?](#question-9-what-is-hard-state-vs-soft-state-in-distributed-systems)                            |
| 10.   | [What are compensating transactions?](#question-10-what-are-compensating-transactions)                                                                     |
| 11.   | [How do you design a global flight tracking system like FlightRadar24?](#question-11-how-do-you-design-a-global-flight-tracking-system-like-flightradar24) |
| 12.   | [How do you design a fitness tracking app like Strava?](#question-12-how-do-you-design-a-fitness-tracking-app-like-strava)                                 |
| 13.   | [How do you design an online payment splitting app like Splitwise?](#question-13-how-do-you-design-an-online-payment-splitting-app-like-splitwise)         |
| 14.   | [How do you design an online photo gallery like Flickr?](#question-14-how-do-you-design-an-online-photo-gallery-like-flickr)                               |
| 15.   | [How do you design a ticketing system for large-scale events?](#question-15-how-do-you-design-a-ticketing-system-for-large-scale-events)                   |
| 16.   | [How do you design a podcast transcription system?](#question-16-how-do-you-design-a-podcast-transcription-system)                                         |
| 17.   | [How do you design a file diffing system like Git?](#question-17-how-do-you-design-a-file-diffing-system-like-git)                                         |
| 18.   | [How do you design a live sports commentary platform?](#question-18-how-do-you-design-a-live-sports-commentary-platform)                                   |
| 19.   | [How do you design a collaborative code editor like VSCode Live Share?](#question-19-how-do-you-design-a-collaborative-code-editor-like-vscode-live-share) |
| 20.   | [How do you design an app marketplace like Google Play Store?](#question-20-how-do-you-design-an-app-marketplace-like-google-play-store)                   |

## Question 1. What are trade-offs between consistency and availability?

# Trade-offs Between Consistency and Availability

## Direct answer

Consistency and availability are two fundamental goals in distributed systems, but during a network partition you generally cannot guarantee both simultaneously. This is the core idea behind the CAP theorem.

- **Consistency (C):** Every client sees the latest committed data, regardless of which server they access.
- **Availability (A):** Every request receives a response, even if some servers are down or disconnected.

When a **network partition (P)** occurs, a distributed system must choose between:

- **Consistency:** Reject or delay some requests until replicas synchronize.
- **Availability:** Continue serving requests, even if different replicas temporarily return different values.

---

## Intuition

Imagine a banking system replicated across two data centers.

```
           Network Partition

     DC-A  XXXXXXXXXXXXX  DC-B
```

A user deposits ₹100 into DC-A.

Another user immediately checks the balance from DC-B.

The system has two choices:

### Option 1: Prioritize Consistency

DC-B refuses or delays the read because it cannot verify the latest balance.

```
Write -> Success

Read -> Wait / Error
```

**Result**

- Everyone sees the same data.
- Some requests fail or time out.

---

### Option 2: Prioritize Availability

DC-B immediately returns the locally stored balance.

```
Write -> Success

Read -> Old balance
```

**Result**

- System always responds.
- Users may temporarily see stale data.

---

## Comparison

| Aspect                    | Consistency     | Availability         |
| ------------------------- | --------------- | -------------------- |
| User always gets response | ❌ Not always   | ✅ Yes               |
| Latest data guaranteed    | ✅ Yes          | ❌ Not always        |
| Stale reads possible      | ❌ No           | ✅ Yes               |
| Writes during partition   | May be rejected | Usually accepted     |
| Latency                   | Higher          | Lower                |
| Data correctness          | Highest         | Eventually converges |

---

## Real-world examples

### Consistency-first systems

Examples:

- Distributed SQL databases
- Banking systems
- Payment processing
- Inventory management

Why?

Incorrect data is more harmful than temporary downtime.

Example:

```
Account Balance

Actual: ₹15,000

Every user sees:
₹15,000
```

Even if some requests are temporarily unavailable.

---

### Availability-first systems

Examples:

- Social media feeds
- News feeds
- Product catalogs
- DNS

Why?

Users prefer getting slightly stale data over receiving an error.

Example:

```
Like Count

Replica A:
100

Replica B:
98
```

Both responses are acceptable because the replicas will eventually synchronize.

---

## Eventual consistency

Many modern distributed systems choose **availability** and rely on **eventual consistency**.

```
Time 0

Replica A = 100
Replica B = 95

↓

Replication

↓

Replica B = 100
```

Eventually, all replicas converge to the same state.

---

## When to prioritize each

### Prioritize consistency when

- Banking
- Payments
- Financial ledgers
- Seat reservations
- Stock trading
- Inventory with limited quantities

Incorrect data can cause financial loss or double-booking.

---

### Prioritize availability when

- Social media
- Chat presence indicators
- Analytics dashboards
- News feeds
- Product recommendations
- Video streaming metadata

Temporary inconsistencies are acceptable if the service remains responsive.

---

## Trade-offs

| Choose Consistency                 | Choose Availability                         |
| ---------------------------------- | ------------------------------------------- |
| Correct data                       | Faster responses                            |
| May reject requests                | Always responds                             |
| Higher latency                     | Lower latency                               |
| Better for financial systems       | Better for user-facing web apps             |
| Reduced user-visible inconsistency | Temporary stale reads or conflicting writes |

---

## Interview-ready summary

> Consistency ensures every client sees the latest data, while availability ensures every request receives a response. According to the CAP theorem, when a network partition occurs, a distributed system must choose between these two properties. Consistency-first systems may reject or delay requests to preserve correctness, making them suitable for domains like banking and payments. Availability-first systems continue serving requests despite temporary inconsistencies, making them ideal for applications such as social media and content delivery. The right choice depends on whether correctness or uninterrupted service is more critical for the application's requirements.

## Question 2. What is sticky caching?

## Question 3. What is a cold cache vs a warm cache?

## Question 4. What is a dead letter queue (DLQ)?

## Question 5. What is an append-only log in distributed systems?

## Question 6. Explain optimistic vs pessimistic locking

## Question 7. What is eventual leader election?

## Question 8. What is quorum-based replication?

## Question 9. What is hard state vs soft state in distributed systems?

## Question 10. What are compensating transactions?

## Question 11. How do you design a global flight tracking system like FlightRadar24?

## Question 12. How do you design a fitness tracking app like Strava?

## Question 13. How do you design an online payment splitting app like Splitwise?

## Question 14. How do you design an online photo gallery like Flickr?

## Question 15. How do you design a ticketing system for large-scale events?

## Question 16. How do you design a podcast transcription system?

## Question 17. How do you design a file diffing system like Git?

## Question 18. How do you design a live sports commentary platform?

## Question 19. How do you design a collaborative code editor like VSCode Live Share?

## Question 20. How do you design an app marketplace like Google Play Store?
