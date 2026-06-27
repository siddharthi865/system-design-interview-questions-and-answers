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

## Question 4. How do you detect failures in distributed systems?

## Question 5. What is a watchdog timer in reliability?

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
