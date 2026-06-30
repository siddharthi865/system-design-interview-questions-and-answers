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

# Active-Active vs Active-Passive Setup

## Direct answer

**Active-Active** and **Active-Passive** are two high-availability deployment strategies.

- **Active-Active:** Multiple servers or instances actively serve traffic at the same time. If one fails, the remaining instances continue handling requests.
- **Active-Passive:** One server actively handles all traffic, while another remains on standby. If the active server fails, the passive server takes over.

The choice is a trade-off between **availability, resource utilization, complexity, and cost**.

---

## Active-Active Setup

In an active-active architecture, **all instances are live and serving requests simultaneously**.

### Architecture

```text
               Load Balancer
              /      |      \
             /       |       \
      Server A   Server B   Server C
       Active     Active     Active
```

Traffic is distributed across all healthy instances.

### Failure scenario

```text
Before failure:

LB → A, B, C

After Server B fails:

LB → A, C
```

Users continue to be served with little or no interruption.

### Advantages

- High availability
- Better resource utilization
- Higher throughput
- Easy horizontal scaling
- No idle infrastructure

### Disadvantages

- More complex synchronization
- Data consistency challenges
- Load balancing required
- Conflict resolution may be needed for distributed writes

---

## Active-Passive Setup

In an active-passive architecture, only one instance serves requests.

The passive instance waits for failover.

### Architecture

```text
             Load Balancer
                   |
             Active Server
                   |
            State Replication
                   |
            Passive Server
```

The passive server continuously receives state updates but does not process client traffic.

### Failure scenario

```text
Before:

Active → Serving traffic
Passive → Standby

After failure:

Passive → Promoted to Active
```

Traffic resumes once failover is complete.

### Advantages

- Simpler architecture
- Easier data consistency
- Easier operational management
- Predictable behavior

### Disadvantages

- Standby resources are idle
- Lower overall throughput
- Brief downtime during failover
- Less efficient infrastructure utilization

---

## Comparison

| Feature              | Active-Active                        | Active-Passive                      |
| -------------------- | ------------------------------------ | ----------------------------------- |
| Traffic handling     | All nodes serve traffic              | One node serves traffic             |
| Failover             | Immediate (remaining nodes continue) | Passive node promoted after failure |
| Resource utilization | High                                 | Lower (standby is mostly idle)      |
| Scalability          | Excellent                            | Limited                             |
| Complexity           | Higher                               | Lower                               |
| Data synchronization | More challenging                     | Simpler                             |
| Cost efficiency      | Better                               | Less efficient                      |

---

## Where each is commonly used

### Active-Active

Suitable for:

- Web servers
- API gateways
- Content delivery systems
- Global SaaS applications
- Microservices

Example:

```text
Users
   |
Load Balancer
   |
------------------------
|         |            |
App1     App2        App3
```

Adding another instance immediately increases capacity.

---

### Active-Passive

Suitable for:

- Primary/standby databases
- Disaster recovery systems
- Legacy enterprise applications
- Stateful applications where only one node should process requests

Example:

```text
Primary Database
        |
Synchronous Replication
        |
Standby Database
```

If the primary fails, the standby is promoted.

---

## Database example

### Active-Passive

```text
Primary
   |
Write
   |
Standby
```

- Writes go only to the primary.
- Reads may optionally go to replicas, depending on the design.

---

### Active-Active

```text
DB A <------> DB B
   ^            ^
   |            |
Clients      Clients
```

Both databases accept writes.

Challenges include:

- Write conflicts
- Replication lag
- Conflict resolution
- Split-brain prevention

---

## Multi-region deployment

### Active-Active

```text
          Users
             |
    -----------------
    |               |
Region A        Region B
 Active          Active
```

Users are routed to the nearest healthy region, improving latency and resilience.

### Active-Passive

```text
Primary Region
      |
Replication
      |
Backup Region
```

The backup region is activated only if the primary region becomes unavailable.

---

## Trade-offs

| Active-Active                            | Active-Passive                     |
| ---------------------------------------- | ---------------------------------- |
| Maximizes availability and throughput    | Simpler operations and consistency |
| Efficient hardware utilization           | Idle standby resources             |
| Handles failures with minimal disruption | Short failover delay               |
| Requires sophisticated synchronization   | Easier to implement and maintain   |

---

## When to choose which?

Choose **Active-Active** when:

- High traffic must be handled continuously.
- Near-zero downtime is required.
- You need horizontal scalability.
- The application is stateless or can safely manage distributed state.

Choose **Active-Passive** when:

- Simplicity and operational stability are priorities.
- The application is stateful.
- Only one node should process writes at a time.
- Brief failover delays are acceptable.

---

## Interview-ready summary

> "Active-Active means multiple instances serve traffic simultaneously, providing high availability, better resource utilization, and easy horizontal scaling. It is ideal for stateless services and global applications but requires careful handling of synchronization and consistency. Active-Passive uses one active instance with a standby replica that takes over during failures. It is simpler to operate and well-suited for stateful systems like primary-standby databases, though it leaves standby resources mostly idle and introduces a small failover delay."

## Question 4. How do you design a failover mechanism?

# How do you design a failover mechanism?

## Direct answer

A **failover mechanism** ensures that when a server, service, or entire region fails, traffic is automatically redirected to a healthy instance with minimal downtime and data loss.

A robust failover design typically includes:

- **Health checks** to detect failures
- **Redundant instances** (Active-Active or Active-Passive)
- **Automatic failover logic**
- **State replication**
- **Traffic rerouting** via a load balancer or DNS
- **Recovery and failback** procedures

The exact implementation depends on your availability and consistency requirements (e.g., RTO/RPO targets).

---

# Requirements / Problem Framing

### Functional Requirements

- Detect unhealthy instances
- Redirect traffic automatically
- Recover failed instances safely
- Avoid routing requests to unhealthy nodes

### Non-Functional Requirements

- High availability (e.g., 99.99%+)
- Low failover time
- No single point of failure
- Minimal data loss
- Prevent split-brain scenarios

---

# High-Level Architecture

```text
                Clients
                    |
            Global DNS / GSLB
                    |
          -----------------------
          |                     |
     Region A              Region B
      Active                 Standby
          |
     Load Balancer
      /    |     \
 App1    App2   App3
          |
   Health Monitoring
          |
 Leader Election / Failover
          |
 Replicated Database
```

---

# Components

### 1. Health Checks

Continuously monitor service health.

Checks may include:

- HTTP `/health` endpoint
- TCP connectivity
- Database connectivity
- Dependency availability
- CPU/Memory thresholds

Example:

```text
Every 5 seconds

LB → Server A → 200 OK ✅
LB → Server B → Timeout ❌
LB → Server C → 200 OK ✅
```

After a configurable number of consecutive failures, the instance is marked unhealthy.

---

### 2. Load Balancer

The load balancer sends traffic only to healthy instances.

```text
Healthy:
A
B
C

↓

Requests distributed to all

If B fails:

↓

Requests → A and C only
```

This provides near-instant failover for stateless services.

---

### 3. State Replication

For stateful services, standby instances must receive replicated state.

Example:

```text
Primary DB
     |
Synchronous / Asynchronous
Replication
     |
Standby DB
```

Trade-offs:

- **Synchronous replication:** Strong consistency, higher latency, minimal data loss.
- **Asynchronous replication:** Lower latency, possible data loss during failover.

---

### 4. Automatic Failover

A coordinator or consensus system decides when to promote a standby.

```text
Primary fails

↓

Failure detected

↓

Elect new leader

↓

Update routing

↓

Resume traffic
```

Distributed systems commonly use consensus protocols such as Raft to safely elect a new leader and avoid multiple primaries.

---

### 5. Traffic Redirection

Traffic can be redirected at multiple layers:

| Layer             | Typical Use                 |
| ----------------- | --------------------------- |
| Load Balancer     | Server or instance failures |
| Service Discovery | Microservice failures       |
| DNS               | Regional failover           |
| Anycast           | Network-level failover      |

---

# Active-Active vs Active-Passive

## Active-Active

```text
        LB
      /    \
     A      B
   Active Active
```

If A fails:

```text
LB → B
```

Advantages:

- Immediate failover
- Higher throughput
- Better resource utilization

---

## Active-Passive

```text
Primary

↓

Replication

↓

Standby
```

If Primary fails:

```text
Standby promoted

↓

Traffic redirected
```

Advantages:

- Simpler
- Easier consistency management

Disadvantages:

- Standby remains mostly idle

---

# Failure Detection

Avoid reacting to a single missed heartbeat.

Example policy:

```text
Heartbeat every 2 seconds

Miss 1 → Ignore

Miss 2 → Warning

Miss 3 → Mark unhealthy
```

This reduces false positives caused by transient network issues.

---

# Split-Brain Prevention

A dangerous scenario occurs when two nodes both believe they are the primary.

```text
Network Partition

Primary A

Primary B

Both accept writes
```

Result:

- Data divergence
- Corruption
- Conflicting updates

Solutions:

- Leader election
- Quorum voting
- Fencing tokens
- Distributed consensus (e.g., Raft)

---

# Recovery and Failback

Once the failed node comes back:

```text
Recover

↓

Synchronize missing data

↓

Health check passes

↓

Rejoin cluster
```

Avoid immediately making it primary again unless explicitly desired, as repeated promotions can cause instability ("flapping").

---

# Deep Design Considerations

### Scalability

- Multiple load balancers
- Stateless application servers
- Distributed health monitoring
- Horizontal scaling

### Availability

- Multiple Availability Zones
- Multi-region deployments
- Database replicas
- Redundant networking

### Reliability

- Automatic retries with exponential backoff
- Circuit breakers
- Graceful degradation
- Idempotent request handling

### Consistency

Choose replication based on business requirements:

- Strong consistency → synchronous replication
- Eventual consistency → asynchronous replication

---

# Security / Observability

Monitor:

- Health check failures
- Failover events
- Replication lag
- Heartbeat latency
- Error rates
- Request latency
- Leader election frequency

Log every failover decision and expose metrics and alerts so operators can quickly identify recurring failures or excessive failovers.

---

# Trade-offs

| Decision                   | Benefits                    | Drawbacks                    |
| -------------------------- | --------------------------- | ---------------------------- |
| Active-Active              | Fast failover, scalable     | More complex synchronization |
| Active-Passive             | Simpler, easier consistency | Idle standby resources       |
| Synchronous replication    | Minimal data loss           | Higher latency               |
| Asynchronous replication   | Better performance          | Possible data loss           |
| Aggressive health checks   | Faster detection            | More false positives         |
| Conservative health checks | Fewer false positives       | Slower failover              |

---

# Interview-ready summary

> "A failover mechanism combines health checks, redundancy, state replication, and automated traffic redirection to maintain service availability during failures. Stateless services typically use load balancers to remove unhealthy instances, while stateful services require replicated state and leader election before promoting a standby. Key design concerns include accurate failure detection, preventing split-brain with consensus protocols, choosing between synchronous and asynchronous replication based on RPO/RTO goals, and ensuring observability to validate failover behavior in production."

## Question 5. What is split-brain problem in distributed systems?

# What is the split-brain problem in distributed systems?

## Direct answer

A **split-brain problem** occurs when a distributed system **incorrectly allows two or more nodes to believe they are the leader (or primary) at the same time**.

As a result, multiple nodes accept writes independently, leading to **conflicting updates, data inconsistency, and potential corruption**.

It most commonly happens due to **network partitions** or incorrect failover mechanisms.

---

# Intuition

Imagine a database cluster with:

```text
Primary
   |
Secondary
```

Normally:

- Primary accepts writes.
- Secondary replicates data.

Now suppose the network connection between them fails.

```text
Primary   X   Secondary
```

The secondary can no longer determine whether the primary has crashed or is merely unreachable.

If it promotes itself to primary while the original primary is still running:

```text
Primary A   (accepting writes)

Primary B   (also accepting writes)
```

Now there are **two primaries**.

This is the split-brain problem.

---

# Example

Suppose a banking application has two database servers.

Initially:

```text
        Clients
           |
      Primary A
           |
     Replication
           |
      Secondary B
```

Everything works correctly.

---

### Network partition

The replication link breaks.

```text
Primary A      X      Secondary B
```

Server B assumes A has failed.

It promotes itself.

```text
Primary A

Primary B
```

Now both servers accept writes.

---

### Conflicting writes

Customer 1:

```text
Withdraw ₹500

→ Primary A
```

Customer 2:

```text
Deposit ₹1000

→ Primary B
```

Both operations succeed independently.

Later, when connectivity is restored:

```text
Balance on A = ₹500

Balance on B = ₹2000
```

The system now has conflicting versions of the data.

---

# Why does split-brain happen?

Common causes include:

### 1. Network partition

The most common cause.

```text
A ----X---- B
```

Both nodes assume the other is down.

---

### 2. Incorrect failover

A standby becomes primary too quickly without verifying the original primary is actually unavailable.

---

### 3. Delayed heartbeats

Temporary network congestion delays heartbeat messages, causing healthy nodes to be mistakenly declared dead.

---

### 4. Configuration errors

Improper quorum or clustering configuration can allow multiple leaders.

---

# Why is it dangerous?

Split-brain can cause:

- Data corruption
- Duplicate transactions
- Lost writes
- Conflicting records
- Inconsistent reads
- Service outages during recovery

Example:

```text
User updates profile

Replica A:
Phone = 1111

Replica B:
Phone = 9999
```

Both values cannot be correct simultaneously.

---

# How to prevent split-brain

## 1. Leader Election

Only one node is allowed to become leader.

Consensus algorithms such as Raft or Paxos ensure that a leader is elected only after receiving votes from a majority (quorum) of nodes.

---

## 2. Quorum

Require a majority before accepting writes.

Example:

```text
5 nodes

Need 3 votes
```

If a network partition creates groups of 2 and 3:

```text
Group A = 2 nodes

Group B = 3 nodes
```

Only the group with **3 nodes** can elect a leader and continue processing writes.

The smaller partition becomes read-only or unavailable for writes.

---

## 3. Fencing Tokens

Each newly elected leader receives a **monotonically increasing token**.

Example:

```text
Leader A

Token = 10
```

New leader:

```text
Leader B

Token = 11
```

Storage systems reject operations from older tokens.

Even if Leader A is still running, its writes are ignored because it has an outdated token.

---

## 4. Distributed Locking

Before acting as leader, a node acquires a distributed lock.

If it loses the lock, it must stop accepting writes.

---

## 5. STONITH (Shoot The Other Node In The Head)

In high-availability clusters, the newly promoted primary forcibly powers off or isolates the old primary before serving writes.

This ensures only one active primary exists.

---

# Split-brain vs Network Partition

| Network Partition                          | Split-Brain                                         |
| ------------------------------------------ | --------------------------------------------------- |
| Communication between nodes is interrupted | Multiple nodes incorrectly become leaders           |
| May or may not cause problems              | Always leads to inconsistency if both accept writes |
| A network issue                            | A failure in cluster coordination                   |

A network partition **can lead to** split-brain if the system lacks proper coordination mechanisms.

---

# Trade-offs

| Prevention Technique | Advantages                          | Disadvantages                                |
| -------------------- | ----------------------------------- | -------------------------------------------- |
| Leader election      | Reliable, well-understood           | Requires consensus overhead                  |
| Quorum               | Prevents multiple leaders           | Some partitions become unavailable           |
| Fencing tokens       | Protects storage from stale leaders | Requires token-aware infrastructure          |
| Distributed locks    | Simple leader control               | Lock service becomes critical infrastructure |
| STONITH              | Strong protection                   | Operationally more complex                   |

---

# Real-world examples

- **Primary-replica databases:** Prevent dual primaries through leader election and quorum.
- **Distributed coordination services:** Systems like Apache ZooKeeper and etcd use quorum-based consensus to avoid split-brain.
- **Container orchestration:** Kubernetes relies on its control plane's consensus mechanisms so only one leader manages cluster state.

---

# Interview-ready summary

> "A split-brain problem occurs when multiple nodes in a distributed system mistakenly believe they are the leader and simultaneously accept writes, usually after a network partition or faulty failover. This can result in conflicting updates and data corruption. Production systems prevent split-brain using quorum-based leader election, consensus protocols like Raft or Paxos, fencing tokens, and, in some HA clusters, STONITH to ensure only one primary can process writes at any time."

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
