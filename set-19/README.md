# Set 19

| S.No. | Question                                                                                                                                                    |
| ----- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you design a retry queue with exponential backoff?](#question-1-how-do-you-design-a-retry-queue-with-exponential-backoff)                           |
| 2.    | [How do you design a self-healing system?](#question-2-how-do-you-design-a-self-healing-system)                                                             |
| 3.    | [How do you handle zombie processes in distributed systems?](#question-3-how-do-you-handle-zombie-processes-in-distributed-systems)                         |
| 4.    | [What is replication lag and how do you mitigate it?](#question-4-what-is-replication-lag-and-how-do-you-mitigate-it)                                       |
| 5.    | [What is write-behind caching?](#question-5-what-is-write-behind-caching)                                                                                   |
| 6.    | [How do you design a strongly consistent distributed cache?](#question-6-how-do-you-design-a-strongly-consistent-distributed-cache)                         |
| 7.    | [What is the difference between sync checkpoints and async checkpoints?](#question-7-what-is-the-difference-between-sync-checkpoints-and-async-checkpoints) |
| 8.    | [How do you design a transactional outbox pattern?](#question-8-how-do-you-design-a-transactional-outbox-pattern)                                           |
| 9.    | [What is a saga pattern in distributed transactions?](#question-9-what-is-a-saga-pattern-in-distributed-transactions)                                       |
| 10.   | [How do you design dual writes handling in distributed systems?](#question-10-how-do-you-design-dual-writes-handling-in-distributed-systems)                |
| 11.   | [How do you design log ingestion pipelines at scale?](#question-11-how-do-you-design-log-ingestion-pipelines-at-scale)                                      |
| 12.   | [How do you store logs efficiently for 1 year?](#question-12-how-do-you-store-logs-efficiently-for-1-year)                                                  |
| 13.   | [What is histogram-based monitoring?](#question-13-what-is-histogram-based-monitoring)                                                                      |
| 14.   | [How do you monitor microservices dependency graphs?](#question-14-how-do-you-monitor-microservices-dependency-graphs)                                      |
| 15.   | [What is span sampling in tracing?](#question-15-what-is-span-sampling-in-tracing)                                                                          |
| 16.   | [How do you design an error reporting system like Sentry?](#question-16-how-do-you-design-an-error-reporting-system-like-sentry)                            |
| 17.   | [What is structured logging vs unstructured logging?](#question-17-what-is-structured-logging-vs-unstructured-logging)                                      |
| 18.   | [How do you implement health checks in microservices?](#question-18-how-do-you-implement-health-checks-in-microservices)                                    |
| 19.   | [What is blue/green monitoring?](#question-19-what-is-bluegreen-monitoring)                                                                                 |
| 20.   | [How do you design real-time metrics dashboards?](#question-20-how-do-you-design-real-time-metrics-dashboards)                                              |

## Question 1. How do you design a retry queue with exponential backoff?

## Direct answer

A **retry queue with exponential backoff** is a message-processing system that re-attempts failed tasks with increasing delays between retries (e.g., 1s → 2s → 4s → 8s), to prevent overwhelming downstream services and improve reliability under transient failures.

---

## Requirements / Problem Framing

### Functional requirements

- Accept failed tasks/messages for retry
- Retry automatically based on policy
- Apply exponential backoff with jitter
- Respect max retry limits
- Move permanently failing messages to a **dead-letter queue (DLQ)**

### Non-functional requirements

- High reliability (no message loss)
- Horizontal scalability
- Low contention under retry storms
- Idempotent processing support
- Observability (retry count, failure reasons)

---

## High-Level Architecture

```
          +-------------------+
Producer → | Main Queue        | → Worker Service
          +-------------------+         |
                                         | failure
                                         v
                                +-------------------+
                                | Retry Queue System|
                                +-------------------+
                                   |      |      |
                                   v      v      v
                         Delay Buckets / Scheduler / Timer Wheel
                                   |
                                   v
                          +------------------+
                          | Retry Worker     |
                          +------------------+
                                   |
                     success ------+------ failure
                                   |              |
                                   v              v
                              Success       Dead Letter Queue
```

---

## Core Design

### 1. Retry Queue Structure

Instead of a single queue, we typically use:

- **Main queue** → first attempt
- **Retry queues (delayed buckets)** → scheduled retries
- **DLQ** → permanent failures

Two common implementations:

| Approach                   | Description                                     | Trade-off                  |
| -------------------------- | ----------------------------------------------- | -------------------------- |
| Multiple delayed queues    | One queue per delay level (1s, 2s, 4s...)       | Simple but coarse-grained  |
| Timestamp-based scheduling | Single queue + scheduler checks visibility time | More scalable and flexible |

---

### 2. Exponential Backoff Logic

For each failed message:

```
delay = base * (2 ^ retry_count)
```

Example:

- retry 0 → 1s
- retry 1 → 2s
- retry 2 → 4s
- retry 3 → 8s

### Add jitter (critical in production)

To avoid thundering herd:

```
final_delay = random(0, delay)
or
final_delay = delay * (0.5 + random())
```

---

### 3. Message Schema

```json
{
  "id": "msg_123",
  "payload": {...},
  "retry_count": 2,
  "next_retry_at": 1710000000,
  "last_error": "timeout"
}
```

---

### 4. Retry Scheduler (Core Component)

Responsible for:

- Scanning retry queue
- Checking `next_retry_at <= now`
- Moving message to worker queue

Implementation options:

- Cron-based polling (simple)
- Priority queue (min-heap by timestamp)
- Redis sorted sets (common in production)

Example (Redis ZSET):

```
ZADD retry_queue <timestamp> <message_id>
```

---

### 5. Worker Flow

1. Consume message
2. Process task
3. If success → ACK
4. If failure:
   - increment retry_count
   - compute backoff delay
   - push to retry queue with updated timestamp
   - if retry_count > MAX → send to DLQ

---

## Deep Design Considerations

### 1. Avoiding Retry Storms

- Add jitter
- Spread retry buckets
- Rate limit retry workers

---

### 2. Idempotency (Critical)

Since retries can duplicate execution:

- Use idempotency keys per task
- Ensure downstream operations are safe to repeat

---

### 3. Retry Policies

Different strategies per failure type:

| Error type       | Strategy           |
| ---------------- | ------------------ |
| Network timeout  | retry with backoff |
| 4xx validation   | no retry → DLQ     |
| 5xx server error | retry              |
| Rate limit (429) | longer backoff     |

---

### 4. Priority Handling

You may want:

- High priority retries bypassing queue delays
- Separate queues per priority class

---

### 5. Storage Choice

| Storage                   | Pros                 | Cons                     |
| ------------------------- | -------------------- | ------------------------ |
| Redis ZSET                | fast, simple         | memory bound             |
| Kafka + delay topic       | durable, scalable    | complex scheduling       |
| RabbitMQ delayed exchange | native delay support | vendor-specific          |
| DB + scheduler            | simple               | not scalable at high QPS |

---

### 6. Failure Safety

- At-least-once delivery model
- Persistent retry state
- Crash-safe scheduler (checkpointing processed timestamps)

---

## Trade-offs

| Design Choice              | Pros              | Cons                            |
| -------------------------- | ----------------- | ------------------------------- |
| Fixed delay buckets        | simple            | inefficient backoff granularity |
| Timestamp scheduling       | precise, scalable | more complex                    |
| Immediate retries          | faster recovery   | risk of overload                |
| Strict exponential backoff | protects system   | slower recovery                 |

---

## Capacity / Scaling (Example)

Assume:

- 100K msgs/sec ingestion
- 5% failure rate → 5K/sec retries

Implications:

- Retry system must handle burst amplification
- Scheduler must be horizontally scalable
- Use partitioned retry queues (by hash of message_id)

---

## Security / Observability

### Observability

- retry_count histogram
- failure reason metrics
- DLQ rate alerts
- retry latency distribution

### Logging

- structured logs with message_id + attempt count

### Monitoring

- retry backlog size
- oldest retry age
- DLQ growth rate

---

## Interview-ready Summary

A retry queue with exponential backoff is a fault-tolerant messaging pattern where failed tasks are retried with increasing delays to avoid system overload. It typically consists of a main queue, a retry scheduling mechanism (often using timestamp-based delayed queues or Redis sorted sets), and a dead-letter queue for permanent failures. Exponential backoff combined with jitter prevents retry storms, while idempotency ensures safe reprocessing. The key design challenges are scalable scheduling, failure classification, and avoiding cascading overload during outages.

## Question 2. How do you design a self-healing system?

## Direct answer

A **self-healing system** is a distributed system that can **detect failures, isolate them, and automatically recover** without human intervention. It achieves this through **health monitoring, automated remediation, redundancy, and safe fallback mechanisms**.

At a high level, it continuously answers three questions:

- _Is something broken?_ (detection)
- _How bad is it?_ (diagnosis)
- _What should we do about it automatically?_ (remediation)

---

## Requirements / Problem Framing

### Functional requirements

- Detect service/component failures automatically
- Restart or replace unhealthy instances
- Route traffic away from failed nodes
- Recover from transient and persistent failures
- Maintain service continuity during failures

### Non-functional requirements

- High availability (minimal downtime)
- Fast detection + recovery (low MTTR)
- Safe automated remediation (no cascading failures)
- Scalability across many services/nodes
- Fault isolation

---

## High-Level Architecture

```
                 +----------------------+
                 |  Monitoring System   |
                 | (metrics/logs/traces|
                 +----------+----------+
                            |
                            v
                 +----------------------+
                 |  Health Evaluator    |
                 | (rules + anomaly     |
                 |  detection)          |
                 +----------+----------+
                            |
                unhealthy    | healthy
                            v
                 +----------------------+
                 | Self-Healing Engine  |
                 | (automation brain)   |
                 +----------+----------+
                            |
        +-------------------+-------------------+
        |                   |                   |
        v                   v                   v
 Restart instance     Replace node       Traffic re-routing
 (process manager)   (orchestrator)     (load balancer)

                            |
                            v
                 +----------------------+
                 | Service Registry /   |
                 | Load Balancer        |
                 +----------------------+
```

---

## Core Building Blocks

### 1. Health Monitoring Layer

Collects signals:

- CPU, memory, disk
- Request latency, error rates
- Heartbeats / liveness probes
- Dependency health (DB, cache, queues)

Types of checks:

| Type        | Meaning                |
| ----------- | ---------------------- |
| Liveness    | Is process alive?      |
| Readiness   | Can it serve traffic?  |
| Deep health | Is full stack working? |

---

### 2. Failure Detection System

You need **fast + accurate detection**:

Techniques:

- Heartbeat timeout detection
- Threshold-based alerting (e.g., error rate > 5%)
- Sliding window anomaly detection
- Circuit breaker triggers

Trade-off:

- Fast detection → false positives risk
- Slow detection → longer downtime

---

### 3. Self-Healing Engine (Core Brain)

This is the automation layer that executes recovery actions.

#### Common actions:

- Restart service/container
- Recreate VM/pod
- Scale out replicas
- Kill stuck processes
- Rollback deployment
- Switch traffic to healthy region

Example policy:

```
IF instance unhealthy for > 3 failures
THEN restart instance
IF restart fails 2 times
THEN replace node
```

---

### 4. Orchestration Layer

Usually handled by:

- Kubernetes-style control plane
- VM orchestrators
- Custom cluster managers

Responsibilities:

- Maintain desired state
- Reconcile actual vs desired state
- Recreate failed components

This is the key to “self-healing by design”.

---

### 5. Load Balancing + Traffic Shifting

Self-healing requires **automatic isolation**:

- Remove unhealthy nodes from LB
- Use health checks in routing decisions
- Gradual traffic shifting (canary fallback)
- Circuit breaker integration

---

## Deep Design Considerations

### 1. Avoiding Cascading Failures

A major risk: healing actions themselves overload system.

Mitigation:

- Rate-limit healing actions
- Backoff between restarts
- Bulkhead isolation (limit impact per service)
- Circuit breakers

---

### 2. State vs Stateless Systems

| Type              | Healing Strategy               |
| ----------------- | ------------------------------ |
| Stateless service | restart/recreate freely        |
| Stateful service  | careful recovery + replication |

Stateful systems require:

- Replication (primary/secondary)
- Snapshot recovery
- WAL/log replay

---

### 3. Idempotent Recovery Actions

All healing actions must be safe to retry:

- Restart multiple times safely
- No duplicate side effects
- Safe rollback mechanisms

---

### 4. Multi-layer Healing Strategy

Self-healing is not one layer; it’s hierarchical:

| Layer     | Action               |
| --------- | -------------------- |
| App       | retry, fallback      |
| Container | restart              |
| Node      | reboot               |
| Cluster   | reschedule workloads |
| Region    | failover             |

---

### 5. Time-based escalation model

```
t + 0s   → retry request
t + 10s  → restart container
t + 30s  → replace instance
t + 2m   → failover region
```

---

### 6. Observability is critical

Without observability, healing is blind.

- Metrics: latency, error rate, saturation
- Logs: structured error logs
- Tracing: distributed request paths
- Alerts: anomaly + threshold-based

---

## Trade-offs

| Approach             | Pros              | Cons                         |
| -------------------- | ----------------- | ---------------------------- |
| Aggressive healing   | fast recovery     | false positives              |
| Conservative healing | stable system     | slower recovery              |
| Fully automated      | zero ops overhead | risk of bad automation loops |
| Human-in-the-loop    | safe              | slow                         |

---

## Failure Scenarios to Handle

- Zombie processes (alive but non-responsive)
- Split brain (multiple primaries)
- Dependency failure (DB down)
- Network partitions
- Cascading overload
- Bad deploy rollout

---

## Interview-ready Summary

A self-healing system is a distributed architecture that automatically detects failures through monitoring, diagnoses severity, and executes corrective actions like restart, replacement, or traffic rerouting. It relies heavily on orchestration systems, health checks, circuit breakers, and automated remediation policies. The key design challenge is balancing fast recovery with avoiding false positives and cascading failures, while ensuring idempotent and safe healing actions across stateless and stateful components.

## Question 3. How do you handle zombie processes in distributed systems?

## Direct answer

A **zombie process in distributed systems** is a worker/service instance that appears alive but is actually **stuck, unresponsive, or no longer correctly participating in the system**, often still holding resources or processing state incorrectly.

You handle zombie processes using a combination of:

- **Heartbeats + lease/TTL mechanisms**
- **Watchdogs / supervisors**
- **Fencing (to prevent split-brain actions)**
- **Forced termination + re-spawn**
- **Idempotent task execution + safe re-assignment**

The core idea is:

> “If a process cannot continuously prove it is healthy and authorized, the system assumes it is dead and replaces it safely.”

---

## Requirements / Problem Framing

### What “zombie” means in distributed systems

Not just OS-level zombies (defunct processes), but:

- Process is alive but **not making progress**
- Process is **partitioned from cluster but still running**
- Process is **hung due to deadlock / GC pause / IO stall**
- Process is **split-brain actor still performing writes**

### Goals

- Detect stuck/unresponsive workers
- Prevent duplicate or unsafe work execution
- Recover quickly without corruption
- Avoid split-brain side effects

---

## High-Level Architecture

```
            +----------------------+
            |   Coordinator        |
            | (Scheduler/Control)  |
            +----------+-----------+
                       |
        leases/assign  |  heartbeat/renew
                       v
            +----------------------+
            | Worker Process      |
            +----------------------+
                       |
             heartbeat |
                       v
            +----------------------+
            | Health Monitor      |
            | (watchdog system)   |
            +----------+-----------+
                       |
          expired lease| no heartbeat
                       v
            +----------------------+
            | Zombie Detector     |
            +----------+-----------+
                       |
        revoke lease   | kill + reassign
                       v
     +----------------------+     +----------------------+
     | Fencing Service      | --> | New Worker Instance  |
     +----------------------+     +----------------------+
```

---

## Core Strategies to Handle Zombie Processes

### 1. Heartbeat + Lease (Primary mechanism)

Each worker must periodically renew a **lease**:

- Worker gets task with lease: `lease_id, expiry_time`
- Must heartbeat before expiry
- If heartbeat stops → lease expires → worker considered zombie

Example:

```
Lease duration: 30s
Heartbeat interval: 10s
If no heartbeat in 30s → reassign task
```

This is widely used in:

- Kubernetes kubelet liveness
- Distributed job queues
- Microservice schedulers

---

### 2. Watchdog Process (Local recovery)

Each node runs a **watchdog/supervisor**:

- systemd / supervisord / kubelet
- monitors process health
- restarts hung processes

Detects:

- no CPU progress
- thread deadlock
- memory freeze / GC stall

Action:

- SIGTERM → graceful shutdown
- SIGKILL → force kill
- restart process

---

### 3. Fencing (Prevents split-brain corruption)

Even after detecting a zombie, the real danger is:

> old process still writing data

So we use **fencing tokens**:

- Each lease has a monotonic version number
- Only latest token is allowed to write

Example:

```
Worker A gets lease token = 100
Worker A becomes zombie
Worker B gets token = 101

If A tries writing → rejected (stale token)
```

This is critical in:

- distributed databases
- leader election systems
- file systems

---

### 4. Task Reassignment with Idempotency

Once zombie is detected:

- Task is reassigned to another worker
- System assumes **at-least-once execution**
- Must ensure idempotent processing

Pattern:

- Use idempotency keys
- Deduplicate writes
- Use transactional outbox patterns

---

### 5. Visibility Timeout (Queue-based systems)

Used in systems like SQS-style queues:

```
Message → Worker receives it → hidden for T seconds
If not ACKed → becomes visible again
```

If worker dies or becomes zombie:

- message automatically re-queued

---

### 6. Health Signals (Beyond heartbeat)

To detect “false alive” processes:

| Signal               | Meaning                |
| -------------------- | ---------------------- |
| heartbeat            | basic liveness         |
| progress metrics     | is work advancing?     |
| request success rate | is service functional? |
| dependency checks    | is DB/cache reachable? |

A zombie often still sends heartbeat but:

- makes no progress
- fails downstream work

So systems use **multi-dimensional health checks**.

---

### 7. Kill + Replace Strategy

Once confirmed zombie:

1. Stop accepting work
2. Force terminate process
3. Clean resources
4. Restart new instance

In container systems:

- kubelet kills pod
- scheduler recreates it elsewhere

---

## Deep Design Considerations

### 1. Zombie detection is probabilistic

You never know with certainty → you rely on:

- timeouts
- missing heartbeats
- lack of progress

Trade-off:

- shorter timeout → faster recovery, more false positives
- longer timeout → safer, slower recovery

---

### 2. Split-brain prevention is critical

Worst-case scenario:

- zombie + new worker both act on same data

Solution:

- fencing tokens
- leader leases
- single-writer enforcement

---

### 3. Network partitions complicate everything

A worker might:

- be alive locally
- but isolated from coordinator

System must assume:

> “If it cannot communicate, it is unsafe.”

---

### 4. Stateful vs Stateless workers

| Type      | Handling                               |
| --------- | -------------------------------------- |
| Stateless | kill + recreate                        |
| Stateful  | failover + replication + recovery logs |

---

### 5. Avoiding cascading restarts

If many nodes are marked zombie at once:

- throttle restarts
- stagger recovery
- avoid “restart storms”

---

## Trade-offs

| Approach                   | Pros          | Cons                         |
| -------------------------- | ------------- | ---------------------------- |
| Aggressive timeouts        | fast recovery | false positives              |
| Conservative timeouts      | safe          | slow recovery                |
| Heartbeat-only             | simple        | misses stuck-but-alive cases |
| Multi-signal health checks | accurate      | complex                      |

---

## Interview-ready Summary

Zombie processes in distributed systems are handled by combining **heartbeat-based leases, watchdog supervision, and fencing mechanisms**. The system detects unresponsive or non-progressing workers using timeouts and health signals, then safely reassigns work while preventing duplicate side effects through fencing tokens and idempotent execution. The key challenge is balancing fast failure detection with avoiding false positives and ensuring correctness under network partitions and split-brain scenarios.

## Question 4. What is replication lag and how do you mitigate it?

## Question 5. What is write-behind caching?

## Question 6. How do you design a strongly consistent distributed cache?

## Question 7. What is the difference between sync checkpoints and async checkpoints?

## Question 8. How do you design a transactional outbox pattern?

## Question 9. What is a saga pattern in distributed transactions?

## Question 10. How do you design dual writes handling in distributed systems?

## Question 11. How do you design log ingestion pipelines at scale?

## Question 12. How do you store logs efficiently for 1 year?

## Question 13. What is histogram-based monitoring?

## Question 14. How do you monitor microservices dependency graphs?

## Question 15. What is span sampling in tracing?

## Question 16. How do you design an error reporting system like Sentry?

## Question 17. What is structured logging vs unstructured logging?

## Question 18. How do you implement health checks in microservices?

## Question 19. What is blue/green monitoring?

## Question 20. How do you design real-time metrics dashboards?
