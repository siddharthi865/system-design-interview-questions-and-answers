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

## Direct answer

**Mutual TLS (mTLS) authentication** is a security mechanism where **both the client and the server authenticate each other using TLS certificates** during the TLS handshake. Unlike standard TLS (where only the server is verified), mTLS ensures **bidirectional identity verification**, providing strong protection against impersonation and man-in-the-middle attacks in service-to-service communication.

---

## Requirements / problem framing

In distributed systems, especially microservices:

- Services need to securely communicate over untrusted networks
- We must ensure:
  - The client is who it claims to be
  - The server is legitimate

- Passwords or API keys alone are not sufficient at scale

mTLS solves this by using **cryptographic identity (X.509 certificates)** instead of shared secrets.

---

## High-level architecture

### Key components

- **Certificate Authority (CA)** – issues and signs certificates
- **Client Service** – presents a client certificate
- **Server Service** – presents a server certificate
- **TLS stack (e.g., Envoy, Nginx, service mesh like Istio)**

### Flow diagram

```
Client                      Server
  |                           |
  |---- TLS handshake ------->|
  |     (ClientHello)         |
  |<--- Server Certificate ---|
  |                           |
  |--- Client Certificate --->|
  |                           |
  |--- Key exchange + verify->|
  |                           |
  |==== Secure channel =======|
```

---

## How mTLS works (step-by-step)

### 1. Server authentication (standard TLS behavior)

- Server sends its certificate
- Client verifies:
  - Signed by trusted CA
  - Domain matches certificate
  - Certificate not expired/revoked

---

### 2. Client authentication (extra step in mTLS)

- Server requests client certificate
- Client sends its certificate
- Server verifies:
  - Signed by trusted CA
  - Certificate is valid for identity (service/user)
  - Not revoked

---

### 3. Session establishment

- Both parties derive session keys
- All subsequent communication is encrypted

---

## Deep design considerations

### 1. Certificate management

You need a full PKI system:

- Root CA (offline, highly secure)
- Intermediate CA (issues certs)
- Automated certificate issuance (short-lived certs preferred)

Common tools:

- SPIFFE/SPIRE for identity management
- Service mesh (Istio, Linkerd)

---

### 2. Identity model

Instead of usernames:

- Identity = certificate subject / SPIFFE ID
  Example:

```
spiffe://company.com/service/payment-service
```

---

### 3. Rotation & revocation

Critical for security:

- Short-lived certs (hours/days)
- Automatic rotation via agents
- Revocation via:
  - CRL (Certificate Revocation List)
  - OCSP (Online Certificate Status Protocol)

---

### 4. Where mTLS is enforced

#### Option A: Service mesh (preferred)

- Sidecar proxy (Envoy)
- Transparent mTLS between services

#### Option B: Application layer

- App handles TLS directly
- More flexible but harder to manage

---

### 5. Performance trade-offs

mTLS adds:

- TLS handshake overhead
- Certificate validation cost

Mitigations:

- Session reuse (TLS session resumption)
- Connection pooling (keep-alive)
- Hardware acceleration (rare cases)

---

## Trade-offs

| Approach          | Pros                                                          | Cons                                     |
| ----------------- | ------------------------------------------------------------- | ---------------------------------------- |
| mTLS              | Strong authentication, no shared secrets, zero trust friendly | Operational complexity (PKI management)  |
| API keys          | Simple                                                        | Weak security, hard rotation             |
| OAuth/JWT         | Flexible, stateless                                           | Token leakage risk, needs secure storage |
| mTLS + JWT hybrid | Strong + flexible identity                                    | More complex architecture                |

---

## Security benefits

mTLS provides:

- Strong service identity (cryptographic)
- Protection against MITM attacks
- No reliance on passwords or static secrets
- Ideal for zero-trust architectures

---

## Real-world usage examples

- Microservices in Kubernetes (via Istio)
- Banking/payment systems internal APIs
- Service-to-service authentication in cloud providers (AWS, GCP internal systems)

---

## Interview-ready summary

Mutual TLS (mTLS) is an extension of TLS where **both client and server authenticate each other using X.509 certificates during the handshake**. It provides strong, cryptographic identity verification and is widely used in microservices and zero-trust architectures. While it significantly improves security by eliminating reliance on shared secrets, it introduces operational complexity around certificate issuance, rotation, and revocation, which is often managed using a PKI system or service mesh.

## Question 3. How do you implement secure inter-datacenter communication?

## Direct answer

Secure inter–datacenter (DC-to-DC) communication is implemented using a **layered security + private networking approach**, typically combining:

- **Private backbone or encrypted tunnels (IPSec / WireGuard / MPLS)**
- **Mutual TLS (mTLS) at service layer**
- **Strong identity + certificate-based authentication (PKI)**
- **Network segmentation + zero-trust policies**
- **End-to-end encryption + key management system (KMS)**

The core idea is:

> Even if the network is compromised, every packet is still encrypted, authenticated, and authorized.

---

## Requirements / problem framing

### Functional requirements

- Secure service-to-service communication across datacenters
- Data confidentiality + integrity in transit
- Strong authentication between services/DCs
- Controlled routing between regions
- Failover between datacenters without security degradation

### Non-functional requirements

- High availability (multi-region resilience)
- Low latency (avoid excessive encryption overhead)
- Scalability across thousands of services
- Zero-trust security model
- Compliance (SOC2, PCI, HIPAA)

---

## High-level architecture

### Typical layered design

```id="dc-secure-arch"
        +----------------------+
        |   Service Layer      |
        |   (mTLS / SPIFFE)    |
        +----------+-----------+
                   |
        +----------v-----------+
        | Service Mesh (Envoy) |
        | Policy + AuthZ       |
        +----------+-----------+
                   |
        +----------v-----------+
        | Encrypted Network    |
        | IPSec / WireGuard    |
        +----------+-----------+
                   |
        +----------v-----------+
        | Private Backbone     |
        | MPLS / SD-WAN / BGP  |
        +----------+-----------+
                   |
        +----------v-----------+
        | Remote Datacenter    |
        +----------------------+
```

---

## Deep design considerations

### 1. Network layer security (first line of defense)

You typically avoid public internet entirely:

#### Option A: Private backbone (preferred)

- MPLS or cloud backbone (AWS Global Accelerator, Google Cloud Interconnect)
- Low latency, high reliability
- Physically isolated from public internet

#### Option B: Encrypted tunnels over internet

- IPSec tunnels between DCs
- WireGuard (modern, lightweight alternative)
- Used when private links are unavailable

✔ Pros: strong encryption at network layer
✖ Cons: operational overhead, key rotation complexity

---

### 2. Service-to-service security (mTLS)

Even inside secure network tunnels, you still enforce:

- **mTLS between services**
- Each service has identity (certificates)
- Certificates issued by internal PKI

Benefits:

- Prevents lateral movement if network is compromised
- Eliminates trust in “network perimeter”

---

### 3. Identity & PKI system

You need a centralized identity system:

- Root CA (offline, highly secure)
- Intermediate CAs per region or cluster
- Short-lived certificates (hours/days)
- Automated rotation

Identity model:

```id="pk3id"
spiffe://company.com/datacenter/us-east/service/payment
```

---

### 4. Authorization layer (beyond encryption)

Encryption alone is not enough.

You enforce:

- Service-to-service ACLs
- Policy engine (OPA / custom IAM)
- Context-aware rules:
  - region restrictions
  - service identity
  - request type

Example:

- “Payment service in DC1 can call fraud service in DC2 but not vice versa”

---

### 5. Routing and failover

Cross-DC communication relies on:

- BGP-based routing between datacenters
- Health-aware traffic routing (failover regions)
- DNS-based or service discovery-based routing

Key design:

- Avoid hardcoding DC endpoints
- Use global service discovery layer

---

### 6. Data protection in transit

All layers must encrypt:

| Layer              | Mechanism              |
| ------------------ | ---------------------- |
| Network            | IPSec / WireGuard      |
| Transport          | TLS 1.3                |
| Service            | mTLS                   |
| Payload (optional) | Field-level encryption |

In high-security systems (banking):

- Double encryption (network + application layer)

---

### 7. Key management (critical)

All encryption depends on KMS:

- Central or federated KMS per region
- Keys never leave HSM
- Automatic rotation
- Cross-region key replication (encrypted only)

Failure scenario handling:

- If KMS in one DC fails → fallback to replica DC

---

### 8. Observability & audit

You must track:

- Cross-DC request traces
- TLS handshake failures
- Certificate validation errors
- Latency between DCs

Pipeline:

```id="obs1"
Services → tracing (OpenTelemetry) → central observability stack
         → logs → immutable storage
         → metrics → Prometheus/Grafana
```

---

## Trade-offs

| Approach                          | Pros                             | Cons                                      |
| --------------------------------- | -------------------------------- | ----------------------------------------- |
| Private backbone only             | Fast, secure, low overhead       | Expensive, limited flexibility            |
| VPN/IPSec over internet           | Flexible, cheaper                | Higher latency, key management complexity |
| mTLS only (no network encryption) | Strong app-layer security        | Vulnerable to network-level attacks       |
| Multi-layer (VPN + mTLS)          | Defense in depth (best practice) | Operational complexity                    |

---

## Common real-world architecture (best practice)

Most large-scale systems use:

1. **Private network backbone between DCs**
2. **IPSec or WireGuard encryption at network layer**
3. **mTLS at service layer**
4. **Central PKI + KMS for key lifecycle**
5. **Service mesh for policy enforcement**
6. **Global load balancing + failover routing**

This creates a **zero-trust, defense-in-depth architecture**.

---

## Security considerations

Key threats and mitigations:

- **MITM attacks → mTLS + IPSec**
- **Compromised DC → strict segmentation + revocation**
- **Replay attacks → TLS nonces + timestamps**
- **Certificate theft → short-lived certs + HSM-backed keys**
- **Insider threats → IAM + audit logs + least privilege**

---

## Interview-ready summary

Secure inter-datacenter communication is implemented using a **defense-in-depth model**: a private or encrypted network layer (MPLS/IPSec/WireGuard) ensures transport security, while **mTLS provides service-level authentication and encryption**. A centralized **PKI and KMS system manages certificates and keys**, and a service mesh enforces identity-based access control. Combined with strong observability, routing, and zero-trust policies, this ensures that even if the network is compromised, services remain authenticated, authorized, and encrypted end-to-end.

## Question 4. What is role-based access control (RBAC)?

## Direct answer

**Role-Based Access Control (RBAC)** is an authorization model where **access permissions are assigned to roles, and users are assigned to those roles**. Instead of giving permissions directly to users, you define roles (like _Admin_, _Developer_, _Viewer_) and map them to allowed actions.

So the rule is:

> **User → Role → Permissions → Resources**

---

## Requirements / problem framing

In large systems, we need a scalable way to control:

- Who can access what resource
- What actions they can perform (read, write, delete, etc.)
- How to manage permissions without assigning them individually per user

Direct user-permission mapping does not scale, so RBAC solves this by introducing abstraction via roles.

---

## Core concept

### Key entities

- **User**: actual identity (Alice, Bob, service account)
- **Role**: collection of permissions (Admin, Editor, Viewer)
- **Permission**: action on a resource (e.g., `read:invoice`, `delete:user`)
- **Resource**: system object (DB record, API, file, service)

---

### Simple mapping

```
User ───> Role ───> Permissions ───> Resources
```

Example:

- Alice → Admin → [create_user, delete_user, read_all_data]
- Bob → Viewer → [read_reports]

---

## High-level architecture

```id="rbac_arch"
        +----------------------+
        |      Auth System     |
        | (Login / Identity)   |
        +----------+-----------+
                   |
                   v
        +----------------------+
        |   RBAC Engine       |
        | (Policy Evaluation) |
        +----------+-----------+
                   |
     +-------------+-------------+
     |                           |
     v                           v
+-----------+             +----------------+
| User DB   |             | Role/Policy DB |
+-----------+             +----------------+
```

---

## How RBAC works (step-by-step)

### 1. User logs in

- System authenticates user (password, OAuth, etc.)

### 2. Fetch roles

- System retrieves roles assigned to the user

### 3. Resolve permissions

- Roles are mapped to permissions

### 4. Authorization check

- On each request:
  - Check if role allows the requested action on resource

---

### Example flow

User: `Bob`

- Role: `Viewer`
- Permission: `read:dashboard`

Request:

```
GET /admin/dashboard
```

RBAC engine:

- Does Viewer have `read:dashboard`? → YES → allow
- Does Viewer have `delete:user`? → NO → deny

---

## Deep design considerations

### 1. Role hierarchy (optional enhancement)

Roles can inherit permissions:

```
Admin
  ↑
Manager
  ↑
Viewer
```

- Admin inherits Manager + Viewer permissions
- Reduces duplication

---

### 2. RBAC vs ACL vs ABAC

| Model                                 | Concept                                            | Pros             | Cons                    |
| ------------------------------------- | -------------------------------------------------- | ---------------- | ----------------------- |
| RBAC                                  | Role-based permissions                             | Simple, scalable | Not fine-grained        |
| ACL (Access Control List)             | Per-user permissions per resource                  | Very granular    | Hard to manage at scale |
| ABAC (Attribute-Based Access Control) | Rules based on attributes (time, location, device) | Highly flexible  | Complex                 |

---

### 3. Fine-grained RBAC (common in real systems)

Instead of just “role = admin”, modern systems use:

```
permission = resource + action
```

Examples:

- `invoice:read`
- `invoice:write`
- `user:delete`

---

### 4. Multi-tenant RBAC

In SaaS systems:

- Roles are scoped per tenant
- Same user can have different roles in different organizations

Example:

- Alice = Admin in Org A
- Alice = Viewer in Org B

---

### 5. Caching for performance

RBAC checks happen on every request, so:

- Cache user → roles → permissions mapping
- Use JWT claims for stateless authorization (with caveats)

---

### 6. Security considerations

- Principle of least privilege (only required permissions)
- Avoid role explosion (too many roles becomes unmanageable)
- Regular role audits
- Strong separation between admin role management and application logic

---

## Trade-offs

| Approach | Pros                             | Cons                                |
| -------- | -------------------------------- | ----------------------------------- |
| RBAC     | Simple, scalable, widely adopted | Not flexible for dynamic conditions |
| ACL      | Precise control                  | Hard to maintain at scale           |
| ABAC     | Very flexible (context-aware)    | Complex policy evaluation           |

---

## Real-world usage examples

- AWS IAM roles (RBAC + policy hybrid)
- Kubernetes RBAC (cluster, namespace roles)
- Enterprise SaaS admin dashboards
- Internal microservices authorization

---

## Interview-ready summary

Role-Based Access Control (RBAC) is an authorization model where **permissions are assigned to roles, and users inherit permissions through those roles**. It simplifies access management by decoupling users from direct permission assignments, making it highly scalable for enterprise systems. While RBAC is simple and widely used, modern systems often extend it with hierarchical roles or combine it with ABAC for more fine-grained, context-aware access control.

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
