# Set 9

| S.No. | Question                                                                                                                                          |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What is Byzantine fault tolerance?](#question-1-what-is-byzantine-fault-tolerance)                                                               |
| 2.    | [What is eventual vs strong consistency tradeoff?](#question-2-what-is-eventual-vs-strong-consistency-tradeoff)                                   |
| 3.    | [What is active-active vs active-passive setup?](#question-3-what-is-active-active-vs-active-passive-setup)                                       |
| 4.    | [How do you design a failover mechanism?](#question-4-how-do-you-design-a-failover-mechanism)                                                     |
| 5.    | [What is split-brain problem in distributed systems?](#question-5-what-is-split-brain-problem-in-distributed-systems)                             |
| 6.    | [How do you design a consensus algorithm?](#question-6-how-do-you-design-a-consensus-algorithm)                                                   |
| 7.    | [How do you recover from data corruption in distributed databases?](#question-7-how-do-you-recover-from-data-corruption-in-distributed-databases) |
| 8.    | [What is tracing in microservices?](#question-8-what-is-tracing-in-microservices)                                                                 |
| 9.    | [What is the difference between metrics, logging, and tracing?](#question-9-what-is-the-difference-between-metrics-logging-and-tracing)           |
| 10.   | [What is Prometheus and how is it used in monitoring?](#question-10-what-is-prometheus-and-how-is-it-used-in-monitoring)                          |
| 11.   | [How do you design an alerting system for anomalies?](#question-11-how-do-you-design-an-alerting-system-for-anomalies)                            |
| 12.   | [How do you handle log aggregation at scale?](#question-12-how-do-you-handle-log-aggregation-at-scale)                                            |
| 13.   | [How do you design a dashboard for system health?](#question-13-how-do-you-design-a-dashboard-for-system-health)                                  |
| 14.   | [How do you track SLAs, SLOs, and SLIs in system design?](#question-14-how-do-you-track-slas-slos-and-slis-in-system-design)                      |
| 15.   | [What is distributed tracing and why is it important?](#question-15-what-is-distributed-tracing-and-why-is-it-important)                          |
| 16.   | [How do you implement request correlation IDs?](#question-16-how-do-you-implement-request-correlation-ids)                                        |
| 17.   | [How do you design a metrics pipeline?](#question-17-how-do-you-design-a-metrics-pipeline)                                                        |
| 18.   | [What is zero-trust architecture?](#question-18-what-is-zero-trust-architecture)                                                                  |
| 19.   | [What is rate limiting vs throttling?](#question-19-what-is-rate-limiting-vs-throttling)                                                          |
| 20.   | [How do you prevent replay attacks in distributed systems?](#question-20-how-do-you-prevent-replay-attacks-in-distributed-systems)                |

## Question 1. What is Byzantine fault tolerance?

# Byzantine Fault Tolerance (BFT)

## Direct answer

**Byzantine Fault Tolerance (BFT)** is the ability of a distributed system to continue operating correctly even when some nodes behave **arbitrarily or maliciously**—for example, sending conflicting information, lying, or acting unpredictably.

Unlike ordinary fault tolerance, which assumes nodes simply crash or stop responding, BFT assumes nodes can behave in **any possible incorrect way**.

---

## Intuition

Imagine four generals surrounding a city who must agree whether to attack.

One of the generals is a traitor.

The traitor sends:

- "Attack" to one general.
- "Retreat" to another.
- Different messages to everyone.

The loyal generals now receive conflicting information.

The challenge is:

> **How can honest participants still reach the same decision despite malicious participants?**

This is known as the **Byzantine Generals Problem**.

---

## Crash Fault vs Byzantine Fault

| Crash Fault           | Byzantine Fault                        |
| --------------------- | -------------------------------------- |
| Node stops working    | Node behaves arbitrarily               |
| Easy to detect        | Difficult to detect                    |
| No incorrect messages | May send false or conflicting messages |
| Simpler consensus     | Requires stronger consensus algorithms |

Example:

Crash fault:

```
Server A ✓
Server B ✓
Server C ✗ (offline)
```

Byzantine fault:

```
Server A: "Value = 10"
Server B: "Value = 10"
Server C: tells A "20", tells B "30"
```

---

## Why is BFT needed?

In distributed systems:

- Nodes may be compromised.
- Hardware may malfunction unpredictably.
- Software bugs may produce incorrect outputs.
- Attackers may control some servers.

The system should still:

- Reach the correct consensus.
- Prevent incorrect data from being accepted.
- Continue serving requests.

---

## Byzantine Generals Problem

Suppose there are 4 generals.

```
      G1
     /  \
   G2 -- G3
     \
      G4
```

One general is malicious.

If G2 says:

```
To G1 -> Attack
To G3 -> Retreat
To G4 -> Attack
```

How do the honest generals know the correct decision?

A consensus algorithm is needed.

---

## The 3f + 1 Rule

Most practical BFT algorithms require:

> **To tolerate `f` Byzantine failures, you need at least `3f + 1` replicas.**

Examples:

| Faulty Nodes | Total Nodes Needed |
| ------------ | ------------------ |
| 1            | 4                  |
| 2            | 7                  |
| 3            | 10                 |
| 4            | 13                 |

### Why?

If:

```
Total = 4

1 malicious
3 honest
```

The honest majority can outvote the malicious node.

---

## Consensus Process

A simplified flow:

```
Client
   |
Request
   |
Leader
   |
Broadcast proposal
   |
Replica A
Replica B
Replica C
Replica D
   |
Exchange votes
   |
Majority agreement
   |
Commit
```

Even if one replica lies:

```
A -> Yes
B -> Yes
C -> Yes
D -> Fake
```

The honest replicas still reach agreement.

---

## Popular BFT Algorithms

### 1. Practical Byzantine Fault Tolerance (PBFT)

Most well-known BFT algorithm.

Phases:

```
Pre-Prepare
      ↓
Prepare
      ↓
Commit
```

Advantages:

- Fast
- Deterministic
- Widely studied

Disadvantages:

- Communication complexity is **O(n²)**.
- Doesn't scale well to hundreds or thousands of nodes.

---

### 2. Tendermint Consensus

Used in many blockchain systems.

Features:

- Voting rounds
- Validator set
- Immediate finality
- BFT consensus

---

### 3. HotStuff

Modern BFT protocol.

Advantages:

- Simpler than PBFT
- Better scalability
- Used by several modern blockchain platforms

---

## Message Complexity

### Crash Fault Consensus

Usually:

```
O(n)
```

### PBFT

Every node communicates with every other node.

```
O(n²)
```

Example:

10 nodes:

```
10 × 10 = 100 messages
```

100 nodes:

```
100 × 100 = 10,000 messages
```

This is why classical BFT protocols don't scale well.

---

## Where is BFT used?

### Blockchain

Examples include:

- Cosmos
- Hyperledger Fabric (in some deployments)
- Permissioned blockchain systems

Nodes may be owned by different organizations that do not fully trust one another.

---

### Distributed Databases

Some highly secure replicated databases use BFT to protect against malicious replicas.

---

### Financial Systems

Banks and payment networks may use BFT-based replication when strong integrity guarantees are required.

---

### Military Systems

Critical command-and-control systems require resilience against compromised nodes.

---

## Trade-offs

| Advantages                 | Disadvantages                                    |
| -------------------------- | ------------------------------------------------ |
| Handles malicious failures | High communication overhead                      |
| Strong consistency         | More complex algorithms                          |
| Prevents incorrect commits | Requires at least `3f + 1` replicas              |
| High security              | Doesn't scale as easily as crash-fault consensus |

---

## Crash Fault Tolerance vs Byzantine Fault Tolerance

| Feature                       | Crash Fault Tolerance | Byzantine Fault Tolerance  |
| ----------------------------- | --------------------- | -------------------------- |
| Handles crashes               | ✅                    | ✅                         |
| Handles malicious nodes       | ❌                    | ✅                         |
| Handles inconsistent messages | ❌                    | ✅                         |
| Typical algorithms            | Raft, Paxos           | PBFT, Tendermint, HotStuff |
| Required replicas             | `2f + 1`              | `3f + 1`                   |
| Complexity                    | Lower                 | Higher                     |

---

## When should you use BFT?

Use BFT when:

- Participants do **not fully trust** one another.
- Some nodes could be **compromised or malicious**.
- Strong correctness is more important than maximizing throughput.
- You're building blockchains, cross-organization systems, or highly secure distributed services.

For systems within a single trusted organization (such as most microservices at a company), **crash-fault tolerance** using algorithms like Raft is usually sufficient and far more efficient.

---

## Interview-ready summary

> "Byzantine Fault Tolerance is the ability of a distributed system to reach correct consensus even when some nodes behave arbitrarily or maliciously. Unlike crash fault tolerance, BFT assumes nodes may lie, send conflicting messages, or act unpredictably. Most BFT protocols require **3f + 1 replicas** to tolerate **f** Byzantine failures. Algorithms like PBFT, Tendermint, and HotStuff are common in blockchains and other trustless or high-security environments, while systems in trusted environments typically prefer Raft or Paxos because they are simpler and more efficient."

## Question 2. What is eventual vs strong consistency tradeoff?

# Eventual Consistency vs Strong Consistency Trade-off

## Direct answer

The trade-off between **strong consistency** and **eventual consistency** is a balance between **data correctness (consistency)** and **availability, latency, and scalability**.

- **Strong consistency** guarantees that every read returns the **latest committed write**, but may increase latency or reduce availability during failures.
- **Eventual consistency** allows temporary inconsistencies, but provides lower latency, higher availability, and better scalability. Given no new updates, all replicas eventually converge to the same value.

This trade-off is closely related to the **CAP theorem**, where network partitions often force a system to prioritize either consistency or availability.

---

## Strong Consistency

With strong consistency, once a write is acknowledged, **all subsequent reads** return the latest value.

### Example

```
Time T1:
User A updates profile name:
"John" → "Johnny"

Time T2:
Any user reading immediately sees:

Johnny ✅
```

There is no possibility of reading stale data.

### How it's achieved

- Synchronous replication
- Consensus algorithms (e.g., Raft or Paxos)
- Quorum-based writes with appropriate read/write quorum sizes

### Advantages

- No stale reads
- Easier application logic
- Strong correctness guarantees

### Disadvantages

- Higher write latency
- Lower availability during network partitions
- Harder to scale globally

---

## Eventual Consistency

With eventual consistency, a write is accepted immediately by one replica and propagated asynchronously to others.

For a short period, different replicas may return different values.

### Example

```
Replica A:
Johnny

Replica B:
John

Replica C:
John
```

After replication completes:

```
Replica A:
Johnny

Replica B:
Johnny

Replica C:
Johnny
```

All replicas converge eventually.

### Advantages

- Very low latency
- High availability
- Excellent horizontal scalability
- Better performance across multiple geographic regions

### Disadvantages

- Temporary stale reads
- Applications may need conflict resolution
- More complex client logic in some cases

---

## Comparison

| Feature                        | Strong Consistency       | Eventual Consistency |
| ------------------------------ | ------------------------ | -------------------- |
| Latest data guaranteed         | ✅                       | ❌                   |
| Stale reads possible           | ❌                       | ✅                   |
| Write latency                  | Higher                   | Lower                |
| Read latency                   | Higher (in some designs) | Lower                |
| Availability during partitions | Lower                    | Higher               |
| Scalability                    | Moderate                 | Excellent            |
| Typical replication            | Synchronous              | Asynchronous         |

---

## Real-world examples

### Strong consistency

Suitable when correctness is critical:

- Banking transactions
- Payment processing
- Inventory management
- Order processing
- Distributed locks

Example:

```
Balance = ₹500

Withdraw ₹500

Next read must return:

₹0
```

Returning ₹500 after the withdrawal could lead to incorrect business decisions.

---

### Eventual consistency

Suitable when brief delays are acceptable:

- Social media feeds
- Likes and reactions
- Product reviews
- DNS propagation
- Analytics dashboards

Example:

```
You like a post.

Your friend sees:

0 likes

Two seconds later:

1 like
```

This temporary inconsistency is generally acceptable.

---

## Timeline comparison

### Strong consistency

```
Write
  |
Commit everywhere
  |
Read

Always latest value
```

### Eventual consistency

```
Write
  |
Replica A updated
  |
Read from Replica B
Old value
  |
Replication completes
  |
Future reads
Latest value
```

---

## Impact on system design

### Strong consistency

```
Client
   |
Leader
   |
Sync Replication
   |
Followers
   |
ACK
```

The client waits until replication satisfies the consistency requirement before receiving success.

### Eventual consistency

```
Client
   |
Primary Replica
   |
ACK immediately
   |
Async replication
   |
Other replicas updated later
```

The client receives a faster response, but some replicas may temporarily lag behind.

---

## Choosing between them

| Use Strong Consistency When       | Use Eventual Consistency When         |
| --------------------------------- | ------------------------------------- |
| Money is involved                 | High read throughput is needed        |
| Data correctness is essential     | Minor delays are acceptable           |
| Double spending must be prevented | Global low-latency access is required |
| Inventory must be exact           | Social interactions and feeds         |

---

## Common techniques

### Strong consistency

- Leader-based replication
- Consensus protocols
- Synchronous replication
- Read/write quorums (e.g., ensuring **R + W > N**)

### Eventual consistency

- Asynchronous replication
- Background synchronization
- Version vectors
- Conflict resolution (last-write-wins, CRDTs, or application-specific merge logic)

---

## Trade-offs

| Strong Consistency           | Eventual Consistency           |
| ---------------------------- | ------------------------------ |
| Highest correctness          | Highest availability           |
| Predictable reads            | Faster responses               |
| Simpler application logic    | More complex conflict handling |
| Higher latency               | Lower latency                  |
| Less resilient to partitions | More resilient to partitions   |

---

## Interview-ready summary

> "Strong consistency guarantees that every read sees the most recent successful write, making it ideal for systems like banking or inventory management where correctness is critical. Eventual consistency allows replicas to be temporarily out of sync but converge over time, enabling lower latency, higher availability, and better scalability. In distributed systems, the choice depends on business requirements: prioritize strong consistency when incorrect data is unacceptable, and eventual consistency when performance and availability matter more than immediate freshness."

## Question 3. What is active-active vs active-passive setup?

## Question 4. How do you design a failover mechanism?

## Question 5. What is split-brain problem in distributed systems?

## Question 6. How do you design a consensus algorithm?

## Question 7. How do you recover from data corruption in distributed databases?

## Question 8. What is tracing in microservices?

## Question 9. What is the difference between metrics, logging, and tracing?

## Question 10. What is Prometheus and how is it used in monitoring?

## Question 11. How do you design an alerting system for anomalies?

## Question 12. How do you handle log aggregation at scale?

## Question 13. How do you design a dashboard for system health?

## Question 14. How do you track SLAs, SLOs, and SLIs in system design?

## Question 15. What is distributed tracing and why is it important?

## Question 16. How do you implement request correlation IDs?

## Question 17. How do you design a metrics pipeline?

## Question 18. What is zero-trust architecture?

## Question 19. What is rate limiting vs throttling?

## Question 20. How do you prevent replay attacks in distributed systems?
