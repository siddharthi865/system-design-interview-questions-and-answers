# Set 14

| S.No. | Question                                                                                                                                   |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| 1.    | [What is checkpointing in distributed systems?](#question-1-what-is-checkpointing-in-distributed-systems)                                  |
| 2.    | [What is log replication?](#question-2-what-is-log-replication)                                                                            |
| 3.    | [What is data reconciliation?](#question-3-what-is-data-reconciliation)                                                                    |
| 4.    | [How do you detect failures in distributed systems?](#question-4-how-do-you-detect-failures-in-distributed-systems)                        |
| 5.    | [What is a watchdog timer in reliability?](#question-5-what-is-a-watchdog-timer-in-reliability)                                            |
| 6.    | [How do you handle phantom reads in transactions?](#question-6-how-do-you-handle-phantom-reads-in-transactions)                            |
| 7.    | [What is a write quorum?](#question-7-what-is-a-write-quorum)                                                                              |
| 8.    | [What is geo-replication and what are its challenges?](#question-8-what-is-geo-replication-and-what-are-its-challenges)                    |
| 9.    | [How do you design an active geo-distributed database?](#question-9-how-do-you-design-an-active-geo-distributed-database)                  |
| 10.   | [How do you handle inconsistent states in distributed systems?](#question-10-how-do-you-handle-inconsistent-states-in-distributed-systems) |
| 11.   | [What is log sampling?](#question-11-what-is-log-sampling)                                                                                 |
| 12.   | [What is real user monitoring (RUM)?](#question-12-what-is-real-user-monitoring-rum)                                                       |
| 13.   | [How do you design synthetic monitoring?](#question-13-how-do-you-design-synthetic-monitoring)                                             |
| 14.   | [How do you design anomaly detection in metrics?](#question-14-how-do-you-design-anomaly-detection-in-metrics)                             |
| 15.   | [What is log rotation?](#question-15-what-is-log-rotation)                                                                                 |
| 16.   | [What is rate sampling in observability?](#question-16-what-is-rate-sampling-in-observability)                                             |
| 17.   | [How do you monitor latency percentiles (P50, P95, P99)?](#question-17-how-do-you-monitor-latency-percentiles-p50-p95-p99)                 |
| 18.   | [What is red/black monitoring?](#question-18-what-is-redblack-monitoring)                                                                  |
| 19.   | [What is log enrichment?](#question-19-what-is-log-enrichment)                                                                             |
| 20.   | [How do you monitor microservices dependencies?](#question-20-how-do-you-monitor-microservices-dependencies)                               |

## Question 1. What is checkpointing in distributed systems?

# Checkpointing in Distributed Systems

## Direct answer

**Checkpointing** is the process of periodically saving the state of a distributed application so that it can recover from failures without restarting from the beginning. If a node crashes, the system restores the latest saved checkpoint and resumes execution from that point.

It is a fundamental technique for **fault tolerance**, **disaster recovery**, and **long-running distributed computations**.

---

## Why is checkpointing needed?

Distributed systems often execute tasks that can run for hours or days. If one machine fails and there are no checkpoints:

- All progress may be lost.
- The computation must restart from scratch.
- Recovery time becomes very high.

Checkpointing minimizes lost work by allowing recovery from the latest saved state.

---

## Simple example

Suppose you're processing a **10 TB dataset**.

```
0% ------------------------------ 100%

After 8 hours:
Processed = 70%
Checkpoint saved

Node crashes

Without checkpoint:
Restart from 0%

With checkpoint:
Resume from 70%
```

Instead of losing 8 hours of work, only the work done after the last checkpoint is lost.

---

## What does a checkpoint contain?

A checkpoint typically stores:

- Current program state
- Memory contents (or application state)
- CPU registers (if required)
- Open file positions
- Pending messages or offsets
- Transaction progress
- Metadata

For distributed applications, it may also include communication state between nodes.

---

## Types of checkpointing

### 1. Coordinated checkpointing (Synchronous)

All nodes coordinate to take a checkpoint at the same logical time.

```
Node A ---- Checkpoint
Node B ---- Checkpoint
Node C ---- Checkpoint
```

### Advantages

- Simple recovery
- Consistent global state
- No orphan messages

### Disadvantages

- Requires synchronization
- Temporary pause in execution
- Higher coordination overhead

---

### 2. Uncoordinated checkpointing (Independent)

Each node checkpoints independently.

```
Node A -> every 5 min

Node B -> every 8 min

Node C -> every 10 min
```

### Advantages

- No synchronization
- Lower runtime overhead

### Disadvantages

- Recovery is difficult
- Can lead to inconsistent states
- Suffers from the **domino effect**

---

### 3. Communication-induced checkpointing

Nodes normally checkpoint independently but are forced to checkpoint when communication patterns require it to maintain consistency.

This is a compromise between coordinated and uncoordinated checkpointing.

---

## The domino effect

This is the biggest problem with uncoordinated checkpointing.

Example:

```
Time →

A: C1 --------- C2 -------- Crash

B: C1 ---- Msg ---- C2

C: C1 -------- C2
```

If node A rolls back to C1:

- Message sent after C1 no longer exists.
- B may have processed that message.
- B must also roll back.
- That rollback may force C to roll back.

Eventually, every node may need to roll back to the very first checkpoint.

```
A rollback
   ↓
B rollback
   ↓
C rollback
   ↓
Entire system restarts
```

This cascading rollback is called the **domino effect**.

---

## Checkpoint frequency trade-off

Choosing how often to checkpoint is important.

### Frequent checkpoints

**Pros**

- Faster recovery
- Less work lost

**Cons**

- High storage usage
- More CPU and I/O overhead
- Performance degradation

---

### Infrequent checkpoints

**Pros**

- Lower runtime overhead
- Better performance

**Cons**

- More work lost after failure
- Longer recovery time

The optimal interval balances checkpoint cost against expected recovery cost.

---

## Recovery process

When a failure occurs:

1. Detect failure.
2. Identify the latest valid checkpoint.
3. Restore application state.
4. Restore communication state if needed.
5. Resume execution.

```
Failure
   │
Restore checkpoint
   │
Reload state
   │
Resume execution
```

---

## Where checkpointing is used

### Distributed data processing

- Apache Spark
- Apache Flink
- Hadoop MapReduce

Recover long-running jobs after failures.

---

### Stream processing

Checkpoint consumer offsets and processing state.

Example:

```
Kafka Offset = 1,250,000
Operator state
Window state

↓ Crash

Recover from checkpoint

Continue at offset 1,250,000
```

---

### Databases

Distributed databases checkpoint memory state to reduce crash recovery time.

---

### Virtual machines and containers

Hypervisors can checkpoint entire virtual machines so they can be resumed later.

---

### Machine learning

Long-running training jobs periodically save model weights and optimizer state so training can resume after interruptions.

---

## Checkpointing vs Logging

| Checkpointing                    | Logging                              |
| -------------------------------- | ------------------------------------ |
| Saves complete application state | Records individual operations/events |
| Faster recovery                  | May require replaying logs           |
| Larger storage requirement       | Smaller incremental storage          |
| Periodic snapshots               | Continuous recording                 |

Many production systems combine both:

- **Checkpoint** = periodic snapshot
- **Log** = replay changes since the snapshot

This provides both efficient recovery and minimal data loss.

---

## Trade-offs

| Advantage                  | Disadvantage                                    |
| -------------------------- | ----------------------------------------------- |
| Faster recovery            | Storage overhead                                |
| Improves fault tolerance   | Additional I/O                                  |
| Reduces recomputation      | Coordination cost (for coordinated checkpoints) |
| Supports long-running jobs | Choosing checkpoint intervals is challenging    |

---

## Interview-ready summary

> **Checkpointing is the periodic saving of a distributed application's state so it can recover from failures without restarting from scratch. There are three main approaches: coordinated, uncoordinated, and communication-induced checkpointing. Coordinated checkpointing is the most common because it creates a consistent global state and avoids the domino effect, although it introduces synchronization overhead. In practice, systems like Apache Spark, Apache Flink, distributed databases, and machine learning platforms use checkpointing to achieve fault tolerance and fast recovery, often combining checkpoints with write-ahead logs for efficient restoration.**

## Question 2. What is log replication?

# Log Replication in Distributed Systems

## Direct answer

**Log replication** is the process of copying a sequence of operations (a log) from a **leader node** to one or more **follower nodes** so that all replicas execute the same operations in the same order and remain consistent.

Instead of directly copying the database state, distributed systems replicate the **operations (commands)** that modify the state. This approach is the foundation of consensus algorithms like **Raft** and **Paxos**.

---

## Why do we replicate logs instead of data?

Suppose a banking service receives this transaction:

```text
Transfer ₹500 from A to B
```

Instead of sending the updated account balances to every replica, the leader appends this command to its log:

```text
Entry #101:
Transfer ₹500 from A to B
```

This log entry is then replicated to followers.

Once every replica applies the same log entry in the same order:

```text
Account A -= 500
Account B += 500
```

All replicas reach the same final state.

**Key idea:** Replicate **operations**, not the resulting data.

---

## High-level architecture

```text
                 Client
                    │
                    ▼
              +-------------+
              |   Leader    |
              +-------------+
                    │
         Append log entry
                    │
      ┌─────────────┼─────────────┐
      ▼             ▼             ▼
+-----------+ +-----------+ +-----------+
| Follower1 | | Follower2 | | Follower3 |
+-----------+ +-----------+ +-----------+
      │             │             │
 Apply entry    Apply entry   Apply entry
```

The leader is responsible for accepting writes and ensuring followers receive log entries.

---

## How log replication works

### Step 1: Client sends a write request

```text
Client:
Deposit ₹1000
```

---

### Step 2: Leader appends to its log

```text
Leader Log

1
2
3
4 Deposit ₹1000
```

The entry is not yet considered committed.

---

### Step 3: Leader replicates the log

```text
Leader
   │
AppendEntries
   │
────────────► Followers
```

Followers append the same entry to their logs.

---

### Step 4: Majority acknowledges

Example with five replicas:

```text
Leader      ✓

Follower1   ✓

Follower2   ✓

Follower3   ✗

Follower4   ✗
```

Three out of five nodes acknowledge the entry, forming a majority.

---

### Step 5: Commit

The leader marks the entry as committed and informs followers.

```text
Committed

1
2
3
4 Deposit ₹1000
```

Now every node applies the operation to its local database.

---

## Why is ordering important?

Consider two operations:

```text
Withdraw ₹500
Deposit ₹1000
```

Correct order:

```text
Balance = 1000

Withdraw 500 → 500

Deposit 1000 → 1500
```

If another replica executes:

```text
Deposit 1000

Withdraw 500
```

It may temporarily observe a different state, and with more complex operations the final result can diverge.

Log replication guarantees that **every replica executes the exact same ordered sequence of operations**, ensuring deterministic state.

---

## Components of a log entry

A log entry typically contains:

```text
Index
Term (or Version)
Command
Timestamp (optional)
Checksum (optional)
```

Example:

```text
Index : 52

Term  : 8

Command :
Transfer(A,B,500)
```

---

## Benefits of log replication

- Keeps replicas consistent
- Enables fault tolerance
- Supports automatic leader failover
- Preserves operation ordering
- Simplifies recovery after crashes
- Forms the basis of distributed consensus

---

## Failure handling

Suppose the leader crashes after replicating to two followers.

```text
Leader (crashed)

Follower A ✓

Follower B ✓

Follower C
```

If the entry was replicated to a **majority**, one of those followers can become the new leader, and the committed log entry is preserved.

If the entry never reached a majority, it is discarded to maintain consistency.

---

## Log replication vs State replication

| Log Replication              | State Replication                               |
| ---------------------------- | ----------------------------------------------- |
| Replicates operations        | Replicates entire data                          |
| Lower network bandwidth      | Higher bandwidth                                |
| Preserves operation order    | Focuses on final state                          |
| Easier recovery and replay   | Simpler but less efficient for frequent updates |
| Used by consensus algorithms | Used for snapshots or backups                   |

---

## Log replication vs Checkpointing

| Log Replication                    | Checkpointing                                        |
| ---------------------------------- | ---------------------------------------------------- |
| Replicates every write operation   | Saves a snapshot of the system state                 |
| Ensures replicas stay synchronized | Enables recovery after failures                      |
| Continuous process                 | Periodic process                                     |
| Supports high availability         | Reduces recovery time                                |
| Usually combined with snapshots    | Often restores a snapshot and replays logs afterward |

In many systems, recovery works like this:

```text
Latest Snapshot
        │
        ▼
Replay Log Entries
        │
        ▼
Current State
```

---

## Real-world examples

- Apache Kafka replicates partition logs across brokers for durability and availability.
- etcd uses the Raft consensus algorithm to replicate logs across cluster members.
- Apache ZooKeeper uses a replicated transaction log to keep all servers synchronized.
- Many distributed SQL and NoSQL databases use write-ahead logs combined with log replication to ensure consistency and durability.

---

## Trade-offs

| Advantages         | Disadvantages                                         |
| ------------------ | ----------------------------------------------------- |
| Strong consistency | Additional network overhead                           |
| High availability  | Increased write latency due to replication            |
| Fault tolerance    | More complex leader election and recovery             |
| Ordered execution  | Storage required for logs until they can be compacted |

---

## Interview-ready summary

> **Log replication is the process of copying an ordered sequence of write operations from a leader to follower replicas. Instead of replicating the entire database state, systems replicate commands, ensuring every replica executes the same operations in the same order. Once a majority of replicas acknowledge a log entry, it is committed and applied, providing consistency, durability, and fault tolerance. Consensus algorithms like Raft and Paxos rely on log replication as their core mechanism, and production systems often combine replicated logs with periodic snapshots for efficient recovery.**

## Question 3. What is data reconciliation?

# Data Reconciliation in Distributed Systems

## Direct answer

**Data reconciliation** is the process of detecting and resolving differences between multiple copies of data so that all replicas eventually become consistent.

It is commonly used in distributed systems where temporary inconsistencies can occur due to network failures, replication delays, node crashes, or concurrent updates.

---

## Why is data reconciliation needed?

In a distributed system, data is often replicated across multiple nodes.

Example:

```text
        User updates profile
               │
               ▼
          Primary Database
          Name = "Alice"

        /                 \
       /                   \
Replica A              Replica B
Name = Alice          Name = Alicia
```

Because of replication delay or a network partition, replicas can temporarily hold different values.

**Data reconciliation** identifies these differences and synchronizes the replicas.

---

## Common causes of inconsistency

- Network partitions
- Replication lag
- Node failures
- Concurrent writes
- Lost or duplicated messages
- Software bugs
- Clock synchronization issues

---

## How data reconciliation works

A typical reconciliation process involves four steps:

### 1. Detect differences

Compare data across replicas using techniques such as:

- Checksums
- Hashes
- Version numbers
- Timestamps
- Merkle Trees

Example:

```text
Replica A

User 101
Balance = ₹500

Replica B

User 101
Balance = ₹700
```

The mismatch is detected.

---

### 2. Identify the correct version

The system determines which copy should be considered authoritative.

Possible strategies include:

- Latest timestamp (Last Write Wins)
- Highest version number
- Leader's copy
- Consensus protocol
- Business rules
- Manual review (for financial systems)

---

### 3. Resolve conflicts

Common conflict resolution methods:

#### Last Write Wins (LWW)

The newest update replaces older ones.

```text
10:01 Alice

10:05 Alicia

Result:
Alicia
```

**Pros:** Simple and fast.

**Cons:** Can lose valid updates.

---

#### Version-based reconciliation

Each update increments a version.

```text
Replica A

Version 8

Replica B

Version 10

Result:
Version 10 wins
```

---

#### Merge changes

Combine updates instead of overwriting.

Example:

```text
Replica A
Phone updated

Replica B
Address updated
```

Result:

```text
Phone updated
Address updated
```

Both changes are preserved.

---

#### Application-specific rules

For example, in banking:

```text
Deposit ₹100

Withdraw ₹50
```

Both operations are applied instead of selecting one winner.

---

### 4. Synchronize replicas

After resolving conflicts, the correct data is propagated to all replicas.

```text
Before

Replica A = 500

Replica B = 700

↓

Reconciliation

↓

Replica A = 700

Replica B = 700
```

---

## Techniques used for reconciliation

### 1. Checksums

Each node computes a checksum for its data.

```text
Replica A

Checksum = 8F21

Replica B

Checksum = 8F21
```

If checksums match, the data is likely identical.

---

### 2. Merkle Trees

Instead of comparing the entire dataset, compare hashes in a tree structure.

```text
          Root Hash
         /         \
     Hash1       Hash2
     /   \        /   \
   D1    D2     D3    D4
```

Only branches with mismatched hashes need further comparison, making reconciliation efficient for large datasets.

---

### 3. Version vectors (Vector Clocks)

Track updates from multiple replicas.

Example:

```text
Replica A

[A:5, B:2]

Replica B

[A:5, B:4]
```

Vector clocks help determine whether one version supersedes another or if updates occurred concurrently.

---

### 4. Anti-entropy protocols

Replicas periodically exchange data to repair inconsistencies.

```text
Replica A
      ⇄
Replica B
      ⇄
Replica C
```

Over time, all replicas converge to the same state.

---

## Real-world examples

- Apache Cassandra uses anti-entropy repair and Merkle Trees to reconcile replica differences.
- Amazon DynamoDB uses versioning and conflict resolution to handle replicated data.
- Apache Kafka consumers can reconcile state by replaying event logs after failures.
- Distributed file systems reconcile metadata and file blocks after node recovery.

---

## Data reconciliation vs Data replication

| Data Replication                | Data Reconciliation                 |
| ------------------------------- | ----------------------------------- |
| Copies data to replicas         | Detects and fixes inconsistencies   |
| Happens during normal writes    | Happens after inconsistencies occur |
| Focuses on distributing updates | Focuses on repairing divergent data |
| Continuous process              | Periodic or event-triggered process |

---

## Data reconciliation vs Checkpointing

| Data Reconciliation           | Checkpointing                                                     |
| ----------------------------- | ----------------------------------------------------------------- |
| Synchronizes replicas         | Saves application state                                           |
| Repairs inconsistent data     | Enables crash recovery                                            |
| Works across multiple nodes   | Usually concerns a single application or coordinated global state |
| Supports eventual consistency | Supports fault tolerance and restart                              |

---

## Trade-offs

| Advantages                    | Disadvantages                                  |
| ----------------------------- | ---------------------------------------------- |
| Restores consistency          | Consumes network and CPU resources             |
| Repairs missed updates        | Can be expensive for very large datasets       |
| Supports eventual consistency | Conflict resolution can be complex             |
| Improves system reliability   | Poor conflict resolution may lead to data loss |

---

## Interview-ready summary

> **Data reconciliation is the process of detecting and resolving inconsistencies between replicated copies of data in a distributed system. Differences can arise due to replication lag, network partitions, or node failures. Reconciliation typically involves detecting mismatches (using checksums, hashes, or Merkle Trees), resolving conflicts (using timestamps, version numbers, vector clocks, or application-specific rules), and synchronizing replicas. Systems like Apache Cassandra use anti-entropy repair with Merkle Trees to efficiently reconcile data while maintaining eventual consistency.**

## Question 4. How do you detect failures in distributed systems?

# How do you detect failures in distributed systems?

## Direct answer

Failure detection in distributed systems is typically done using **heartbeats**, **timeouts**, and **health checks**. Since there is no global clock or perfect way to distinguish a slow node from a failed one, distributed systems use **failure detectors** that provide the _best possible estimate_ of whether a node has failed.

A node is generally considered failed if it stops responding within a configured timeout period.

---

## Why is failure detection challenging?

Unlike a single machine, distributed systems cannot directly determine whether another node has crashed.

If a node doesn't respond, it could be because:

- The node crashed.
- The network is partitioned.
- The node is overloaded.
- The response is delayed.
- The message was lost.

This uncertainty is known as the **partial failure problem**.

---

## Common failure detection techniques

### 1. Heartbeats (Most common)

Nodes periodically send small "I'm alive" messages.

```text
        Heartbeat
Node A -----------> Node B
        every 2 sec
```

If Node B stops receiving heartbeats for a configured interval:

```text
Node A

Heartbeat ✓
Heartbeat ✓
Heartbeat ✗
Heartbeat ✗

↓

Node B suspects Node A has failed.
```

**Pros**

- Simple
- Low overhead
- Widely used

**Cons**

- Choosing the heartbeat interval is a trade-off between detection speed and false positives.

---

### 2. Timeout-based detection

Each request has a timeout.

```text
Client
   │
Request
   ▼
Server

Wait 5 sec

↓

No response

↓

Failure suspected
```

If the timeout expires, the requester assumes the node is unavailable.

**Pros**

- Easy to implement
- Works well for RPC systems

**Cons**

- A slow server may be incorrectly marked as failed.

---

### 3. Health checks

A monitoring service periodically checks whether a node is healthy.

Checks may include:

- Process running
- CPU usage
- Memory availability
- Database connectivity
- Disk health
- Network connectivity

Example:

```text
Load Balancer

      │
Health Check

      ▼

Server

HTTP 200 OK
```

If health checks fail repeatedly, the node is removed from service.

---

### 4. Gossip protocol

Nodes randomly exchange status information with each other.

```text
      A
    / | \
   B  C  D
      |
      E
```

If Node A detects that Node C has failed, it gossips that information to other nodes. Eventually, the entire cluster learns about the failure.

**Pros**

- Highly scalable
- No central coordinator
- Fault tolerant

**Cons**

- Failure information propagates gradually.

Used in systems like Apache Cassandra and HashiCorp Consul.

---

### 5. Phi Accrual Failure Detector

Instead of declaring a node simply "alive" or "dead," it computes a **suspicion level (φ)** based on heartbeat history.

```text
φ = 0.5  → Healthy

φ = 3.2  → Possibly slow

φ = 8.5  → Consider failed
```

Advantages:

- Adapts to changing network conditions
- Reduces false positives
- Works well in cloud environments with variable latency

Used by systems like Apache Cassandra and Akka.

---

## Active vs Passive failure detection

| Active Detection          | Passive Detection               |
| ------------------------- | ------------------------------- |
| Periodically probes nodes | Waits for requests to fail      |
| Faster detection          | Simpler implementation          |
| Higher network overhead   | Lower overhead                  |
| Common in clusters        | Common in client-server systems |

---

## What happens after detecting a failure?

Once a node is suspected to have failed, the system may:

1. Stop routing traffic to it.
2. Elect a new leader (if needed).
3. Fail over to replicas.
4. Redistribute tasks.
5. Recover data from replicas or logs.
6. Restart the failed node when it comes back.

Example:

```text
Leader crashes
      │
      ▼
Failure detected
      │
      ▼
Leader election
      │
      ▼
New leader starts serving requests
```

---

## Trade-offs

| Faster Detection                   | Slower Detection             |
| ---------------------------------- | ---------------------------- |
| Faster recovery                    | Fewer false positives        |
| Higher network overhead            | Longer failover time         |
| More sensitive to temporary delays | Better for unstable networks |

Choosing heartbeat intervals and timeout values requires balancing detection speed against the risk of incorrectly suspecting healthy nodes.

---

## Real-world examples

- Kubernetes uses liveness and readiness probes to determine pod health and restart or remove unhealthy instances.
- Apache ZooKeeper uses session timeouts and heartbeats to detect client and server failures.
- etcd uses periodic heartbeats within the Raft consensus protocol to detect leader failures.
- Apache Cassandra combines gossip with the Phi Accrual Failure Detector for scalable, adaptive failure detection.

---

## Interview-ready summary

> **Distributed systems cannot distinguish a crashed node from a slow or partitioned one with certainty, so they rely on failure detectors based on heartbeats, timeouts, health checks, gossip protocols, or adaptive mechanisms like the Phi Accrual Failure Detector. These detectors provide a suspicion of failure rather than absolute certainty. Once a failure is detected, the system can trigger actions such as removing the node from service, electing a new leader, failing over to replicas, or redistributing work, ensuring high availability and fault tolerance.**

## Question 5. What is a watchdog timer in reliability?

# What is a watchdog timer in reliability?

## Direct answer

A **watchdog timer (WDT)** is a reliability mechanism that monitors whether a system, process, or service is still functioning correctly. If the monitored component fails to periodically signal ("kick" or "feed") the watchdog within a specified timeout, the watchdog assumes it has become unresponsive and triggers a recovery action, such as restarting the process, rebooting the machine, or initiating failover.

It is widely used in **embedded systems, operating systems, distributed systems, and cloud infrastructure** to recover automatically from hangs or deadlocks.

---

## Why is a watchdog timer needed?

Some failures do not cause a process to crash—they cause it to **hang**.

For example:

- Infinite loop
- Deadlock
- Resource starvation
- Memory corruption
- Waiting indefinitely on I/O

Without a watchdog:

```text
Application starts

↓

Deadlock occurs

↓

Application hangs forever
```

With a watchdog:

```text
Application starts

↓

Feeds watchdog every 5 sec

↓

Application hangs

↓

Watchdog timeout expires

↓

Restart application
```

The watchdog detects that the application is no longer making progress.

---

## How a watchdog timer works

1. The watchdog starts with a timeout (e.g., 10 seconds).
2. The application periodically resets ("feeds") the timer.
3. As long as the timer is reset before it expires, nothing happens.
4. If the timer expires, the watchdog performs a recovery action.

```text
Time →

Feed   Feed   Feed        (No Feed)

|------|------|-------------X Timeout

                      ↓

              Restart service
```

---

## Types of watchdog timers

### 1. Hardware watchdog

Implemented in hardware (often inside a CPU or microcontroller).

```text
+----------------+
| CPU            |
|                |
|  Feed WDT ---> |
+----------------+
        │
        ▼
Hardware Watchdog
        │
Timeout?
        │
        ▼
System Reset
```

**Advantages**

- Works even if the operating system crashes.
- Highly reliable.
- Common in embedded and automotive systems.

---

### 2. Software watchdog

Implemented by another process or service.

Example:

```text
Supervisor Process
       │
Checks heartbeat
       │
       ▼
Application
```

If the application stops responding, the supervisor restarts it.

Examples include:

- Process supervisors
- Service managers
- Container orchestrators

---

## Recovery actions

When a watchdog detects a timeout, it may:

- Restart the process
- Restart a container
- Reboot the machine
- Trigger leader election
- Switch traffic to a healthy replica
- Generate alerts and logs

---

## Real-world examples

### Embedded systems

A car's braking controller periodically feeds a hardware watchdog.

If the software freezes:

```text
Brake Controller

↓

Stops responding

↓

Hardware Watchdog

↓

Reboot Controller
```

This prevents the controller from remaining permanently stuck.

---

### Cloud services

A monitoring service checks an application every few seconds.

```text
Health Check

↓

Application healthy?

↓

No

↓

Restart container
```

This is effectively a software watchdog.

---

### Distributed systems

A leader node periodically sends heartbeats.

If heartbeats stop:

```text
Leader

(No heartbeat)

↓

Followers detect timeout

↓

Leader election
```

Although implemented using heartbeats and timeouts rather than a traditional watchdog, the concept is similar: detect lack of progress and initiate recovery.

---

## Watchdog timer vs Heartbeat

| Watchdog Timer                                     | Heartbeat                                    |
| -------------------------------------------------- | -------------------------------------------- |
| Monitors a local process or system                 | Monitors another node or service             |
| Detects hangs or lack of progress                  | Detects remote node failures                 |
| Usually triggers restart                           | Usually triggers failover or leader election |
| Common in embedded systems and service supervisors | Common in distributed systems                |

---

## Watchdog timer vs Health check

| Watchdog Timer               | Health Check                                                             |
| ---------------------------- | ------------------------------------------------------------------------ |
| Monitors continuous progress | Checks current health/status                                             |
| Timeout-based                | Request/response based                                                   |
| Often restarts automatically | May remove the instance from service or alert operators                  |
| Usually local                | Often performed by external systems like load balancers or orchestrators |

---

## Best practices

- Choose a timeout longer than the application's normal execution time.
- Feed the watchdog only after completing critical work—not merely because the process is running.
- Log watchdog-triggered restarts for debugging.
- Combine watchdogs with health checks, monitoring, and alerting.
- Avoid overly aggressive timeouts that cause unnecessary restarts.

---

## Trade-offs

| Advantages                  | Disadvantages                                     |
| --------------------------- | ------------------------------------------------- |
| Automatic recovery          | Incorrect timeout values can cause false restarts |
| Detects hangs and deadlocks | Frequent restarts may mask underlying bugs        |
| Improves availability       | Does not identify the root cause of failures      |
| Reduces manual intervention | Recovery may temporarily interrupt service        |

---

## Interview-ready summary

> **A watchdog timer is a reliability mechanism that monitors whether a process or system continues to make progress. The monitored component must periodically reset the watchdog before its timeout expires. If it fails to do so—because of a hang, deadlock, or crash—the watchdog automatically triggers recovery actions such as restarting the process or rebooting the system. Hardware watchdogs are common in embedded systems, while software watchdogs are widely used in cloud infrastructure and distributed systems to improve availability and enable self-healing.**

## Question 6. How do you handle phantom reads in transactions?

## Question 7. What is a write quorum?

## Question 8. What is geo-replication and what are its challenges?

## Question 9. How do you design an active geo-distributed database?

## Question 10. How do you handle inconsistent states in distributed systems?

## Question 11. What is log sampling?

## Question 12. What is real user monitoring (RUM)?

## Question 13. How do you design synthetic monitoring?

## Question 14. How do you design anomaly detection in metrics?

## Question 15. What is log rotation?

## Question 16. What is rate sampling in observability?

## Question 17. How do you monitor latency percentiles (P50, P95, P99)?

## Question 18. What is red/black monitoring?

## Question 19. What is log enrichment?

## Question 20. How do you monitor microservices dependencies?
