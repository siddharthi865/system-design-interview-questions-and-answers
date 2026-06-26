# Set 20

| S.No. | Question                                                                                                                                                   |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you design a secure key management system?](#question-1-how-do-you-design-a-secure-key-management-system)                                          |
| 2.    | [What is mutual TLS authentication?](#question-2-what-is-mutual-tls-authentication)                                                                        |
| 3.    | [How do you implement secure inter-datacenter communication?](#question-3-how-do-you-implement-secure-inter-datacenter-communication)                      |
| 4.    | [What is role-based access control (RBAC)?](#question-4-what-is-role-based-access-control-rbac)                                                            |
| 5.    | [How do you design an attribute-based access control system (ABAC)?](#question-5-how-do-you-design-an-attribute-based-access-control-system-abac)          |
| 6.    | [How do you prevent data leaks in system logs?](#question-6-how-do-you-prevent-data-leaks-in-system-logs)                                                  |
| 7.    | [How do you design a fraud detection pipeline for e-commerce?](#question-7-how-do-you-design-a-fraud-detection-pipeline-for-e-commerce)                    |
| 8.    | [What is secure multi-party computation?](#question-8-what-is-secure-multi-party-computation)                                                              |
| 9.    | [How do you design secure API authentication across regions?](#question-9-how-do-you-design-secure-api-authentication-across-regions)                      |
| 10.   | [What is the principle of least privilege in system design?](#question-10-what-is-the-principle-of-least-privilege-in-system-design)                       |
| 11.   | [How do you design a microfinance lending platform?](#question-11-how-do-you-design-a-microfinance-lending-platform)                                       |
| 12.   | [How do you design a crypto staking platform?](#question-12-how-do-you-design-a-crypto-staking-platform)                                                   |
| 13.   | [How do you design a drone traffic control system?](#question-13-how-do-you-design-a-drone-traffic-control-system)                                         |
| 14.   | [How do you design a global ticket booking system with seat selection?](#question-14-how-do-you-design-a-global-ticket-booking-system-with-seat-selection) |
| 15.   | [How do you design a podcast live-streaming platform?](#question-15-how-do-you-design-a-podcast-live-streaming-platform)                                   |
| 16.   | [How do you design a smart city IoT traffic management system?](#question-16-how-do-you-design-a-smart-city-iot-traffic-management-system)                 |
| 17.   | [How do you design a global news aggregation system?](#question-17-how-do-you-design-a-global-news-aggregation-system)                                     |
| 18.   | [How do you design a fitness wearable data sync system?](#question-18-how-do-you-design-a-fitness-wearable-data-sync-system)                               |
| 19.   | [How do you design a distributed image recognition system?](#question-19-how-do-you-design-a-distributed-image-recognition-system)                         |
| 20.   | [How do you design a scalable AI/ML model training platform?](#question-20-how-do-you-design-a-scalable-aiml-model-training-platform)                      |

## Question 1. How do you design a secure key management system?

## Direct answer

A **secure key management system (KMS)** is a centralized service that securely **generates, stores, distributes, rotates, and revokes cryptographic keys** while enforcing strict access control, auditability, and hardware-backed protection (HSMs). The goal is to ensure keys are never exposed in plaintext outside trusted boundaries and all key usage is tightly controlled and traceable.

---

## Requirements / problem framing

### Functional requirements

- Generate cryptographic keys (symmetric + asymmetric)
- Secure storage of keys (never expose raw keys unnecessarily)
- Encrypt/decrypt, sign/verify operations via APIs
- Key versioning and rotation
- Key revocation and destruction
- Access control per tenant/service/user
- Audit logging of all key usage

### Non-functional requirements

- **Strong security guarantees** (no plaintext key leakage)
- High availability (keys needed for production systems)
- Low latency for crypto operations
- Strong auditability and compliance (PCI-DSS, HIPAA, SOC2)
- Multi-tenant isolation
- Durability and backup of encrypted key material

---

## High-level architecture

### Core components

- **KMS API Gateway**
- **Auth & Policy Engine (IAM)**
- **Key Management Service (KMS Core)**
- **HSM cluster (Hardware Security Module)**
- **Metadata Store (keys, versions, policies)**
- **Audit Logging Pipeline**
- **Envelope Encryption Service**
- **Key Cache (limited TTL, optional)**

### Architecture flow (simplified)

```
Client Service
     |
     v
API Gateway + Auth (IAM)
     |
     v
KMS Core Service  -----> Policy Engine
     |
     +-----> HSM Cluster (key generation + crypto ops)
     |
     +-----> Metadata DB (key metadata, versions)
     |
     +-----> Audit Log Stream (Kafka / event pipeline)
```

---

## Deep design considerations

### 1. Key storage strategy (critical)

You never store plaintext keys in DB.

Two common models:

#### Option A: HSM-backed storage (preferred)

- Keys generated inside HSM
- Key never leaves HSM
- API requests trigger crypto operations inside HSM

✔ Highest security
✖ Higher cost, slightly higher latency

---

#### Option B: Envelope encryption (scalable approach)

- Data Encryption Keys (DEKs) used for actual encryption
- DEKs are encrypted by a Key Encryption Key (KEK) stored in HSM

Flow:

```
Data -> encrypted with DEK
DEK -> encrypted with KEK (stored in DB)
KEK -> resides in HSM
```

✔ Scalable
✔ Lower latency
✔ Common in AWS KMS-style systems

---

### 2. Key lifecycle management

Each key has states:

- **Created → Enabled → Disabled → Scheduled deletion → Destroyed**

Key operations:

- Rotation policy (time-based or event-based)
- Versioning (v1, v2, v3 of same key ID)
- Backward compatibility for decrypting old data

---

### 3. Access control model

Use **fine-grained IAM policies**:

Example:

- Service A can only encrypt using key K1
- Service B can decrypt but not export keys

Enforcement:

- Policy engine checks:
  - Identity (service/user)
  - Action (encrypt/decrypt/sign)
  - Key ARN
  - Context (region, time, IP, workload)

---

### 4. Encryption operations flow

#### Encrypt flow

```
Client → KMS API → Auth check → HSM encrypt or DEK generation → return ciphertext
```

#### Decrypt flow

```
Client → KMS → Auth check → HSM decrypt or unwrap DEK → return plaintext (or partial use)
```

Important:

- Prefer returning **ciphertext or wrapped keys**, not plaintext data when possible.

---

### 5. High availability & scalability

- Multi-region active-active KMS (with replication of metadata only)
- HSM clusters per region
- Stateless API layer (horizontal scaling)
- Strong consistency for key metadata (use distributed DB like Spanner/DynamoDB w/ strong reads)
- Cache only **public metadata**, not keys

---

### 6. Auditability & compliance

Every operation must generate immutable audit logs:

- Who accessed which key
- What operation (encrypt/decrypt/sign)
- When and where
- Result (success/failure)

Pipeline:

```
KMS → Event bus (Kafka/Kinesis) → Immutable storage (S3/WORM) → SIEM
```

---

### 7. Key rotation strategy

Two approaches:

- **Automatic rotation (recommended)**: every 90/180 days
- **Manual rotation for sensitive keys**

Important:

- Keep old key versions for decrypt compatibility
- New encrypt operations use latest version automatically

---

### 8. Caching strategy

Only safe caching:

- Key metadata (enabled/disabled, version info)
- NOT raw keys or unwrapped secrets

Use:

- Short TTL cache
- Per-region cache invalidation on key rotation

---

### 9. Threat model considerations

Protect against:

- Insider attacks → strict IAM + HSM isolation
- Key exfiltration → no plaintext key export
- Replay attacks → request signing + timestamps
- Compromised service → per-service key policies
- MITM → TLS everywhere + mutual auth

---

## Trade-offs

| Approach                                 | Pros                     | Cons                            |
| ---------------------------------------- | ------------------------ | ------------------------------- |
| Full HSM-based KMS                       | Maximum security         | High cost, higher latency       |
| Envelope encryption                      | Scalable, cost-efficient | Slightly more complex           |
| Centralized KMS                          | Simple governance        | Potential bottleneck            |
| Federated KMS (multi-region independent) | Resilient, low latency   | Harder consistency + governance |

---

## Capacity / sizing (rough intuition)

Assumptions:

- 100K QPS encryption requests globally
- 10–20% decrypt traffic

Design implications:

- HSM throughput is bottleneck → must shard HSM clusters
- Cache metadata heavily
- Batch audit logs

---

## Security / observability

### Security

- Hardware-backed root of trust (HSM)
- TLS + mTLS between services
- Strict IAM policies
- Key non-exportability enforced at hardware level

### Observability

- Metrics:
  - Encrypt/decrypt latency
  - HSM utilization
  - Key rotation failures

- Logs:
  - All key access events

- Tracing:
  - End-to-end request tracing for crypto operations

---

## Interview-ready summary

A secure Key Management System is built around **hardware-backed key storage (HSMs), strict IAM-based access control, and envelope encryption for scalability**. It ensures keys are never exposed in plaintext outside trusted boundaries, supports lifecycle management (rotation, versioning, revocation), and provides full auditability for compliance. The main design challenge is balancing **security (HSM isolation) with scalability (metadata services + envelope encryption)** while maintaining low-latency cryptographic operations.

## Question 2. What is mutual TLS authentication?

## Question 3. How do you implement secure inter-datacenter communication?

## Question 4. What is role-based access control (RBAC)?

## Question 5. How do you design an attribute-based access control system (ABAC)?

## Question 6. How do you prevent data leaks in system logs?

## Question 7. How do you design a fraud detection pipeline for e-commerce?

## Question 8. What is secure multi-party computation?

## Question 9. How do you design secure API authentication across regions?

## Question 10. What is the principle of least privilege in system design?

## Question 11. How do you design a microfinance lending platform?

## Question 12. How do you design a crypto staking platform?

## Question 13. How do you design a drone traffic control system?

## Question 14. How do you design a global ticket booking system with seat selection?

## Question 15. How do you design a podcast live-streaming platform?

## Question 16. How do you design a smart city IoT traffic management system?

## Question 17. How do you design a global news aggregation system?

## Question 18. How do you design a fitness wearable data sync system?

## Question 19. How do you design a distributed image recognition system?

## Question 20. How do you design a scalable AI/ML model training platform?
