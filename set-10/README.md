# Set 10

| S.No. | Question                                                                                                                                                    |
| ----- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What is data masking and when is it used?](#question-1-what-is-data-masking-and-when-is-it-used)                                                           |
| 2.    | [How do you design a secure session management system?](#question-2-how-do-you-design-a-secure-session-management-system)                                   |
| 3.    | [What is secret management in microservices?](#question-3-what-is-secret-management-in-microservices)                                                       |
| 4.    | [How do you prevent SQL injection at scale?](#question-4-how-do-you-prevent-sql-injection-at-scale)                                                         |
| 5.    | [What is a DDoS attack and how do you prevent it?](#question-5-what-is-a-ddos-attack-and-how-do-you-prevent-it)                                             |
| 6.    | [What is token-based authentication vs session-based?](#question-6-what-is-token-based-authentication-vs-session-based)                                     |
| 7.    | [How do you design a system for GDPR compliance?](#question-7-how-do-you-design-a-system-for-gdpr-compliance)                                               |
| 8.    | [How do you design a stock trading platform like Zerodha/Robinhood?](#question-8-how-do-you-design-a-stock-trading-platform-like-zerodharobinhood)          |
| 9.    | [How do you design a blockchain-based system?](#question-9-how-do-you-design-a-blockchain-based-system)                                                     |
| 10.   | [How do you design a cryptocurrency exchange (like Binance/Coinbase)?](#question-10-how-do-you-design-a-cryptocurrency-exchange-like-binancecoinbase)       |
| 11.   | [How do you design an IoT device management platform?](#question-11-how-do-you-design-an-iot-device-management-platform)                                    |
| 12.   | [How do you design a ride-hailing surge pricing system (Uber/Ola style)?](#question-12-how-do-you-design-a-ride-hailing-surge-pricing-system-uberola-style) |
| 13.   | [How do you design a global CDN from scratch?](#question-13-how-do-you-design-a-global-cdn-from-scratch)                                                    |
| 14.   | [How do you design a disaster recovery system for a bank?](#question-14-how-do-you-design-a-disaster-recovery-system-for-a-bank)                            |
| 15.   | [How do you design a fraud detection system for payments?](#question-15-how-do-you-design-a-fraud-detection-system-for-payments)                            |
| 16.   | [How do you design a voice assistant system like Alexa/Siri?](#question-16-how-do-you-design-a-voice-assistant-system-like-alexasiri)                       |
| 17.   | [How do you design a healthcare records management system?](#question-17-how-do-you-design-a-healthcare-records-management-system)                          |

## Question 1. What is data masking and when is it used?

## Direct answer

**Data masking** is the process of hiding or replacing sensitive data with realistic but non-sensitive values while preserving the original data's format and usability. It allows developers, testers, analysts, or third parties to work with data without exposing confidential information.

The goal is to **protect sensitive data while maintaining the usefulness of the dataset**.

---

## Why data masking is needed

Production databases often contain sensitive information such as:

- Personal information (PII)
- Credit card numbers
- Bank account details
- Passwords
- Medical records
- Business secrets

Giving direct access to production data increases the risk of:

- Data breaches
- Insider threats
- Compliance violations
- Accidental exposure

Instead, organizations create a masked copy of the data.

Example:

| Original                                | Masked                                      |
| --------------------------------------- | ------------------------------------------- |
| John Smith                              | David Brown                                 |
| [john@gmail.com](mailto:john@gmail.com) | [user123@test.com](mailto:user123@test.com) |
| 9876-5432-1234-5678                     | XXXX-XXXX-XXXX-5678                         |
| Salary = ₹2,50,000                      | Salary = ₹2,40,500                          |

The data still looks realistic but no longer reveals the original values.

---

## Common data masking techniques

| Technique         | Description                        | Example                      |
| ----------------- | ---------------------------------- | ---------------------------- |
| **Substitution**  | Replace with realistic fake values | Alice → Emma                 |
| **Shuffling**     | Rearrange values within a column   | Swap customer names          |
| **Redaction**     | Hide part or all of the value      | XXXX-XXXX-1234               |
| **Encryption**    | Encrypt the data                   | Sensitive value → Ciphertext |
| **Tokenization**  | Replace with a random token        | Card → TKN-874321            |
| **Nulling**       | Replace with NULL or blanks        | Phone → NULL                 |
| **Randomization** | Modify numbers randomly            | Salary ₹80k → ₹82k           |

---

## Types of data masking

### 1. Static Data Masking (SDM)

A copy of the production database is created, and sensitive data is permanently masked.

```
Production DB
      |
Copy Database
      |
Mask Sensitive Fields
      |
Testing / Development
```

**Used for:**

- Development
- QA testing
- Training environments

**Advantages:**

- Production remains untouched
- Safe for sharing
- Fast access

---

### 2. Dynamic Data Masking (DDM)

Data remains unchanged in the database. Masking is applied only when users query it, based on their permissions.

Example:

**Admin sees**

```
Card Number:
1234-5678-9876-5432
```

**Support Engineer sees**

```
XXXX-XXXX-XXXX-5432
```

The stored data is identical; only the displayed value changes.

---

### 3. On-the-fly Data Masking

Sensitive data is masked during migration, ETL, or replication between systems.

```
Production DB
      |
Data Pipeline
      |
Mask Data
      |
Data Warehouse
```

---

## When is data masking used?

### 1. Development and testing

Developers often need production-like data to reproduce bugs, but should not access real customer information.

Example:

- Testing an e-commerce checkout flow using masked customer data.

---

### 2. QA environments

Quality assurance teams validate business workflows without viewing sensitive user information.

---

### 3. Analytics and reporting

Business analysts can analyze trends without exposing personal identifiers.

Example:

- Revenue by city instead of customer names.

---

### 4. Third-party vendors

External consultants or offshore teams may need access to datasets. Masking enables collaboration while protecting confidential information.

---

### 5. Compliance

Many regulations require protection of sensitive data, including:

- GDPR
- HIPAA
- PCI DSS

Data masking helps organizations reduce compliance risk.

---

## Data masking vs encryption

| Feature                | Data Masking                            | Encryption                            |
| ---------------------- | --------------------------------------- | ------------------------------------- |
| Purpose                | Hide sensitive information for safe use | Protect data from unauthorized access |
| Reversible             | Usually no                              | Yes (with the key)                    |
| Used in production     | Sometimes (dynamic masking)             | Yes                                   |
| Used for testing       | Yes                                     | Usually no                            |
| Data remains realistic | Yes                                     | No (ciphertext)                       |

Example:

**Masked**

```
9876-XXXX-XXXX-4321
```

**Encrypted**

```
A9F8C2D14E8B...
```

Masked data is readable and useful for testing, whereas encrypted data is unreadable until decrypted.

---

## Data masking vs tokenization

| Data Masking                                | Tokenization                                           |
| ------------------------------------------- | ------------------------------------------------------ |
| Creates fake but realistic values           | Replaces values with meaningless tokens                |
| Original value is generally not recoverable | Original can be retrieved through a secure token vault |
| Common in testing and analytics             | Common in payment and financial systems                |

Example:

```
Original:
4111-1111-1111-1111

Masked:
XXXX-XXXX-XXXX-1111

Tokenized:
TKN_98AF72CD
```

---

## Best practices

- Mask only the fields containing sensitive data.
- Preserve data formats so applications continue to function correctly.
- Maintain consistency (the same input should map to the same masked value when needed for testing relationships).
- Never expose production data in development or test environments.
- Apply role-based access control alongside masking.
- Audit access to sensitive datasets and masking policies.

---

## Interview-ready summary

> **Data masking is a technique for protecting sensitive information by replacing it with realistic but non-sensitive values while preserving the data's format and usefulness. It is commonly used in development, testing, analytics, and third-party data sharing to prevent exposure of production data and meet compliance requirements. Unlike encryption, masking is typically irreversible and intended for safe data usage rather than secure storage or transmission.**

## Question 2. How do you design a secure session management system?

# Direct answer

A **secure session management system** authenticates users, issues secure session tokens, validates them on every request, supports session expiration and revocation, and protects against attacks such as session hijacking, fixation, replay, and CSRF.

In a system design interview, focus on:

- Secure authentication
- Secure session/token generation
- Session storage
- Session validation
- Expiration and renewal
- Logout and revocation
- Scalability
- Security best practices

---

# Requirements / Problem framing

## Functional requirements

- User login
- Create authenticated session
- Validate session on every request
- Logout from current or all devices
- Session expiration
- Remember-me support (optional)
- Support multiple concurrent devices

## Non-functional requirements

- High availability
- Low latency
- Horizontal scalability
- Strong security
- Easy revocation
- Auditability

---

# High-level architecture

```text
                +----------------+
                |     Client     |
                +-------+--------+
                        |
                 Login Request
                        |
                +-------v--------+
                | Authentication |
                |    Service     |
                +-------+--------+
                        |
            Verify username/password
                        |
                +-------v--------+
                | User Database  |
                +----------------+

After successful login

                Generate Session ID
                        |
                Store Session
                        |
                +-------v--------+
                | Redis / Session|
                |     Store      |
                +-------+--------+
                        |
               Return Secure Cookie
                        |
                Client stores cookie
                        |
            Every authenticated request
                        |
                +-------v--------+
                | API Gateway    |
                +-------+--------+
                        |
             Validate Session ID
                        |
                +-------v--------+
                | Session Store  |
                +----------------+
```

---

# Session lifecycle

## Step 1: Login

User submits:

```text
Email
Password
```

Authentication service:

- verifies credentials
- checks MFA (if enabled)
- generates session ID
- stores session

Example:

```text
Session ID:
8f93ab18d9ef...
```

---

## Step 2: Store session

Store:

```text
SessionID
UserID
CreatedTime
LastAccessTime
ExpiryTime
DeviceInfo
IP (optional)
UserAgent
RefreshToken(optional)
```

Example:

```text
Session {
    sessionId
    userId
    createdAt
    expiresAt
    lastSeen
}
```

Redis is commonly used because:

- fast lookup
- TTL support
- distributed
- replicated

---

## Step 3: Return cookie

Never expose session IDs in URLs.

Return:

```http
Set-Cookie:

session=abc123

HttpOnly
Secure
SameSite=Lax
```

Security flags:

| Flag     | Purpose                                        |
| -------- | ---------------------------------------------- |
| HttpOnly | Prevent JavaScript access (reduces XSS impact) |
| Secure   | HTTPS only                                     |
| SameSite | Helps prevent CSRF                             |

---

## Step 4: Validate session

Every request:

```text
Client
    |
Cookie
    |
API
    |
Redis Lookup
    |
Valid?
```

If valid:

- load user
- continue request

Else:

```text
401 Unauthorized
```

---

# Session storage options

## Option 1: Server-side sessions (recommended)

```text
Client
     |
Session ID
     |
Server
     |
Redis
```

Advantages

- easy logout
- immediate revocation
- session tracking
- simple expiration

Disadvantages

- requires distributed cache

---

## Option 2: Stateless sessions using tokens

Store signed tokens (e.g., JWTs) on the client.

Advantages

- no session lookup
- horizontally scalable
- good for microservices

Disadvantages

- difficult to revoke before expiry
- larger attack surface if leaked
- often paired with short-lived access tokens and refresh tokens

---

# Session expiration

Two common expiration strategies:

### Absolute expiration

Example:

```text
Expires after 8 hours
```

Even if active.

---

### Idle timeout

Example:

```text
Logout after 30 minutes of inactivity
```

Reset:

```text
lastSeen = currentTime
```

Many systems combine both:

```text
Absolute: 24 hours

Idle: 30 minutes
```

---

# Session renewal

Instead of forcing users to log in frequently:

```text
Access Token

15 min
```

Refresh token:

```text
30 days
```

Flow:

```text
Expired Access Token

↓

Refresh Token

↓

New Access Token

↓

Continue
```

This limits the impact of stolen access tokens.

---

# Logout

User clicks Logout:

```text
Delete session from Redis
```

Future requests:

```text
Invalid session

↓

401
```

For "Logout from all devices":

```text
Delete all sessions

WHERE UserID = X
```

---

# Defending against common attacks

## 1. Session hijacking

Attack:

Attacker steals the session cookie.

Mitigations:

- HTTPS everywhere
- `Secure` cookies
- `HttpOnly`
- Short session lifetime
- Rotate session IDs after login or privilege changes
- Detect unusual activity (e.g., impossible travel)

---

## 2. Session fixation

Attack:

Attacker forces the victim to use a known session ID.

Mitigation:

Always generate a **new** session ID after successful authentication.

```text
Old Session

↓

Login

↓

New Session ID
```

---

## 3. Cross-Site Scripting (XSS)

Attack:

Malicious JavaScript steals cookies.

Mitigations:

- `HttpOnly` cookies
- Output encoding
- Content Security Policy (CSP)
- Input validation

---

## 4. Cross-Site Request Forgery (CSRF)

Attack:

A malicious website causes a victim's browser to perform unwanted actions.

Mitigations:

- `SameSite=Lax` or `SameSite=Strict`
- CSRF tokens for state-changing requests
- Validate `Origin` or `Referer` headers where appropriate

---

## 5. Replay attacks

Attack:

Captured requests are replayed.

Mitigations:

- HTTPS
- Short-lived tokens
- Nonces or request signatures for high-risk operations

---

# Scaling the session system

## Redis Cluster

```text
         API Servers

        /     |      \

     Redis Cluster
```

Benefits:

- horizontal scaling
- replication
- automatic failover
- TTL support

---

## Load balancing

Use a load balancer in front of stateless application servers.

Since sessions are stored centrally:

```text
Any server

↓

Redis
```

No sticky sessions are required, making scaling and failover simpler.

---

# Security / observability

**Authentication & authorization**

- Use strong password hashing (e.g., bcrypt or Argon2).
- Support MFA for sensitive accounts.
- Validate user permissions on every protected request.

**Monitoring**

- Track active sessions per user.
- Detect repeated failed logins.
- Alert on abnormal session creation rates or suspicious geographic changes.
- Log login, logout, session renewal, and revocation events with audit trails.

**Data protection**

- Encrypt data in transit with HTTPS/TLS.
- Encrypt sensitive session metadata at rest if required.
- Never store passwords or sensitive secrets in session data.

---

# Trade-offs

| Approach                    | Advantages                                | Disadvantages                        |
| --------------------------- | ----------------------------------------- | ------------------------------------ |
| Server-side sessions        | Easy revocation, logout, session tracking | Requires shared session store        |
| JWT-only                    | Stateless, no database lookup             | Hard to revoke before expiry         |
| Access + Refresh Tokens     | Good security and scalability balance     | More implementation complexity       |
| Sticky Sessions             | Simple to implement initially             | Poor scalability and failover        |
| Central Redis Session Store | Highly scalable, no sticky sessions       | Adds operational dependency on Redis |

---

# Interview-ready summary

> "I would authenticate the user, generate a cryptographically secure random session ID, and store the session in a distributed store like Redis with a TTL. The client receives only the session ID in an `HttpOnly`, `Secure`, and `SameSite` cookie. Every request validates the session against Redis. I'd implement idle and absolute expiration, rotate session IDs after login, support immediate logout by deleting sessions, and defend against session hijacking, fixation, XSS, and CSRF. For scale, application servers remain stateless behind a load balancer, while Redis provides centralized, replicated session storage."

## Question 3. What is secret management in microservices?

# Direct answer

**Secret management** is the practice of securely storing, distributing, rotating, and controlling access to sensitive credentials (called _secrets_) used by microservices.

Secrets include:

- Database passwords
- API keys
- Encryption keys
- OAuth client secrets
- TLS certificates
- Cloud credentials
- Service account tokens

The goal is to **ensure services can securely access the secrets they need without hardcoding or exposing them**.

---

# Why secret management is important

In a microservices architecture, dozens or hundreds of services communicate with databases, message queues, cloud services, and third-party APIs.

Each service may require multiple secrets.

Without proper secret management, teams often:

- Hardcode secrets in source code
- Store credentials in configuration files
- Commit secrets to Git
- Share credentials manually
- Reuse the same password across services

These practices increase the risk of credential leaks and make rotation difficult.

---

# High-level architecture

```text
                +----------------------+
                | Secret Management    |
                |      System          |
                | (Vault/KMS/Cloud)    |
                +----------+-----------+
                           ^
                           |
                 Authenticate & Request
                           |
        +------------------+------------------+
        |                  |                  |
+-------+-------+  +-------+-------+  +-------+-------+
| User Service  |  | Order Service |  | Payment Service|
+-------+-------+  +-------+-------+  +-------+-------+
        |                  |                  |
        +------------------+------------------+
                           |
                    Databases / APIs
```

Instead of embedding passwords, each service authenticates itself and retrieves secrets securely when needed.

---

# What qualifies as a secret?

Common examples include:

- Database usernames and passwords
- API keys
- JWT signing keys
- TLS private keys
- SSH keys
- OAuth client secrets
- Cloud IAM credentials
- Encryption keys

Generally, **any value that grants access or must remain confidential should be treated as a secret**.

---

# How secret management works

### Step 1: Service authenticates

When a service starts, it proves its identity using mechanisms such as:

- Kubernetes Service Account
- Cloud IAM role
- Mutual TLS (mTLS)
- Machine identity
- Short-lived bootstrap token

---

### Step 2: Request a secret

```text
User Service
      |
      | Authenticate
      |
Secret Manager
      |
Return DB Password
```

The service requests only the secrets it is authorized to access.

---

### Step 3: Use the secret

Example:

```text
Database Password

↓

Connect to Database

↓

Never expose password to users
```

The secret is used in memory and should not be logged or exposed.

---

# Where secrets are stored

A centralized secret management system typically provides:

- Encrypted storage
- Access control
- Audit logging
- Secret versioning
- Automatic rotation
- High availability

Popular solutions include:

- HashiCorp Vault
- AWS Secrets Manager
- Azure Key Vault
- Google Cloud Secret Manager

---

# Secret rotation

Secrets should not remain valid indefinitely.

Example:

```text
Old Password

↓

Generate New Password

↓

Update Database

↓

Notify Services

↓

Old Password Expires
```

Automatic rotation reduces the impact of compromised credentials.

---

# Dynamic secrets

Instead of storing a permanent database password, the secret manager can generate **temporary credentials**.

Example:

```text
Service

↓

Request DB Credentials

↓

Secret Manager

↓

Username: app_83921

Password: x72A...

TTL: 1 hour
```

After the TTL expires, the credentials become invalid automatically.

**Benefits:**

- Reduced exposure window
- No shared long-lived passwords
- Easier revocation

---

# Access control

Apply the **principle of least privilege**.

Example:

| Service              | Allowed Secrets        |
| -------------------- | ---------------------- |
| User Service         | User DB password       |
| Payment Service      | Payment API key        |
| Notification Service | Email provider API key |

Each service receives only the secrets it needs.

---

# Common mistakes

### ❌ Hardcoding secrets

```text
const PASSWORD = "admin123";
```

If the code is leaked, the secret is exposed.

---

### ❌ Storing secrets in Git

```text
config.json

password = abc123
```

Secrets committed to version control are difficult to remove completely.

---

### ❌ Logging secrets

```text
Connecting using password=abcd1234
```

Logs should never contain secrets.

---

### ❌ Sharing one credential across services

If one service is compromised, all services using that credential are at risk.

---

# Best practices

- Never hardcode secrets in source code.
- Store secrets in a centralized secret manager.
- Encrypt secrets at rest and in transit.
- Use short-lived credentials whenever possible.
- Rotate secrets automatically and regularly.
- Authenticate services using strong machine identities.
- Enforce least-privilege access policies.
- Audit every secret access.
- Avoid storing secrets in environment variables longer than necessary if more secure retrieval mechanisms are available.
- Never expose secrets in logs, error messages, or monitoring systems.

---

# Trade-offs

| Approach                   | Advantages                               | Disadvantages                                                         |
| -------------------------- | ---------------------------------------- | --------------------------------------------------------------------- |
| Hardcoded secrets          | Very simple                              | Highly insecure, difficult to rotate                                  |
| Configuration files        | Easy to manage initially                 | Risk of accidental exposure                                           |
| Environment variables      | Common in containerized deployments      | Can still be exposed through process inspection or misconfigured logs |
| Centralized secret manager | Secure, auditable, supports rotation     | Additional infrastructure and operational complexity                  |
| Dynamic secrets            | Strongest security, automatic expiration | More complex integration and lifecycle management                     |

---

# Interview-ready summary

> **Secret management is the secure lifecycle management of sensitive credentials used by microservices. Instead of embedding passwords or API keys in code or configuration files, services authenticate to a centralized secret manager to retrieve only the secrets they are authorized to use. A robust solution supports encryption, fine-grained access control, auditing, automatic rotation, and dynamic short-lived credentials, significantly reducing the risk of credential leakage and simplifying operational security.**

## Question 4. How do you prevent SQL injection at scale?

# Direct answer

**SQL injection is prevented by ensuring user input is never interpreted as SQL code.** At scale, this means combining **parameterized queries**, **input validation**, **least-privilege database access**, **secure APIs/ORMs**, **monitoring**, and **defense-in-depth** across all services—not relying on input sanitization alone.

---

# Requirements / Problem framing

The system should:

- Prevent SQL injection across all services.
- Scale to hundreds of microservices and developers.
- Enforce consistent security practices.
- Detect and respond to attempted attacks.
- Minimize performance overhead.

---

# High-level architecture

```text
                 Client
                    |
             API Gateway / WAF
                    |
        +-----------+-----------+
        |           |           |
   User Service  Order Service  Payment Service
        |           |           |
   Parameterized Queries / ORM
        |           |           |
        +-----------+-----------+
                    |
              Database Cluster
```

Every service interacts with the database through secure data access layers that enforce parameterized queries.

---

# Primary defense: Parameterized queries (Prepared Statements)

The database treats user input as **data**, not executable SQL.

### ❌ Vulnerable

```sql
SELECT * FROM users
WHERE username = '" + username + "'
```

If `username` is:

```text
' OR 1=1 --
```

The resulting query becomes:

```sql
SELECT * FROM users
WHERE username='' OR 1=1 --
```

This can return every row.

---

### ✅ Safe

```sql
SELECT * FROM users
WHERE username = ?
```

The parameter is transmitted separately from the SQL statement, so it cannot change the query structure.

This is the **most important defense**.

---

# Use ORMs carefully

Modern ORMs generate parameterized queries automatically.

Examples:

- Prisma
- TypeORM
- Sequelize
- Hibernate
- Entity Framework

However:

```text
ORM

↓

Raw SQL

↓

Potential SQL Injection
```

If developers execute raw SQL by concatenating strings, the protection is lost.

---

# Input validation

Validate inputs before they reach the database.

Examples:

- Integer IDs should contain only digits.
- Email addresses should match expected formats.
- Dates should follow valid date formats.
- Enforce maximum input lengths.

Validation improves security but **does not replace parameterized queries**.

---

# Stored procedures

Properly written stored procedures can reduce injection risks.

Safe:

```sql
CALL GetUser(?)
```

Unsafe:

```sql
EXEC('SELECT * FROM users WHERE name=''' + @name + '''')
```

Dynamic SQL inside stored procedures remains vulnerable.

---

# Principle of least privilege

The database account used by a service should have only the permissions it needs.

Example:

| Service           | Permissions              |
| ----------------- | ------------------------ |
| User Service      | SELECT, UPDATE on users  |
| Order Service     | SELECT, INSERT on orders |
| Reporting Service | Read-only                |

If SQL injection occurs, the attacker's capabilities are limited by the account's privileges.

---

# API design

Never expose arbitrary SQL execution through APIs.

Bad:

```text
POST /query

{
   "sql": "SELECT * FROM users"
}
```

Good:

```text
GET /users/{id}
```

The backend constructs queries internally using parameters.

---

# Escaping is not enough

Escaping quotes can reduce some attacks, but:

- Different databases have different escaping rules.
- Mistakes are common.
- New edge cases appear over time.

**Parameterized queries are the preferred solution.**

---

# Web Application Firewall (WAF)

A WAF can detect common SQL injection patterns before requests reach your application.

Examples of suspicious payloads:

```text
UNION SELECT
```

```text
' OR 1=1
```

```text
DROP TABLE
```

A WAF is a useful **additional layer**, not a replacement for secure coding.

---

# Monitoring and detection

Track:

- Failed SQL queries
- Database syntax errors
- Unusual spikes in query failures
- Repeated suspicious request patterns
- WAF alerts
- High-frequency login failures

Alert on anomalies that may indicate probing or exploitation attempts.

---

# Security in microservices

In large organizations:

```text
Developer

↓

Repository

↓

Static Security Scan

↓

CI/CD

↓

Production
```

Automated checks help ensure:

- No string-concatenated SQL
- Approved database libraries
- ORM usage follows secure patterns
- Security tests pass before deployment

This keeps security consistent across many teams.

---

# Additional best practices

- Use parameterized queries everywhere.
- Avoid dynamic SQL whenever possible.
- Keep database drivers and ORM libraries up to date.
- Store database credentials securely using a secret management system.
- Disable verbose database error messages in production.
- Apply rate limiting to reduce automated attack attempts.
- Perform regular penetration testing and code reviews.

---

# Trade-offs

| Approach              | Advantages                             | Disadvantages                                               |
| --------------------- | -------------------------------------- | ----------------------------------------------------------- |
| Parameterized queries | Strongest and simplest defense         | Requires consistent developer adoption                      |
| ORM                   | Reduces risk and improves productivity | Raw SQL APIs can bypass protections                         |
| Stored procedures     | Can centralize business logic          | Unsafe if they build dynamic SQL                            |
| WAF                   | Blocks many known attack patterns      | Can produce false positives/negatives; not sufficient alone |
| Input validation      | Improves data quality and security     | Cannot prevent SQL injection by itself                      |

---

# Interview-ready summary

> "To prevent SQL injection at scale, I'd enforce parameterized queries across all services and discourage raw SQL concatenation. I'd use ORMs or database libraries that default to prepared statements, validate inputs, apply least-privilege database permissions, and secure secrets through a centralized secret manager. In production, I'd add a WAF, monitor suspicious query patterns, integrate static analysis into CI/CD, and conduct regular security reviews. The key principle is ensuring user input is always treated as data, never as executable SQL."

## Question 5. What is a DDoS attack and how do you prevent it?

# Direct answer

A **Distributed Denial of Service (DDoS) attack** is an attack where **thousands or millions of compromised devices (a botnet) simultaneously send traffic to a target system**, overwhelming its resources and making the service unavailable to legitimate users.

The primary defense is **a layered strategy** that combines **CDNs, DDoS protection services, rate limiting, load balancing, auto-scaling, Web Application Firewalls (WAFs), and traffic filtering**. No single mechanism is sufficient.

---

# Why DDoS attacks are dangerous

The attacker attempts to exhaust one or more critical resources:

- Network bandwidth
- CPU
- Memory
- Database connections
- Application threads
- Load balancer capacity

Example:

```text
Normal Traffic

10,000 users
      |
      v
   Web Server
      |
    Responds


DDoS Attack

10 million requests/sec
        |
        v
   Web Server
        |
     Overloaded
```

Legitimate users experience:

- Slow responses
- Timeouts
- Service outages

---

# Types of DDoS attacks

## 1. Volumetric attacks

Goal: Saturate the network bandwidth.

Examples:

- UDP floods
- ICMP floods
- DNS amplification

Characteristics:

- Massive traffic volume
- Measured in Gbps or Tbps

---

## 2. Protocol attacks

Target weaknesses in network protocols or connection handling.

Examples:

- SYN floods
- TCP connection exhaustion

Instead of consuming bandwidth, these attacks exhaust server or firewall resources.

---

## 3. Application-layer attacks (Layer 7)

Target the application itself.

Examples:

```http
GET /search?q=laptop
```

or

```http
POST /login
```

Thousands of expensive requests per second can overload CPUs, databases, or caches, even with relatively low bandwidth.

These are often the hardest attacks to distinguish from legitimate traffic.

---

# High-level architecture

```text
                    Internet
                        |
                 DDoS Protection
                        |
                      CDN
                        |
                      WAF
                        |
                 Load Balancer
                        |
          +-------------+-------------+
          |             |             |
      Web Server    Web Server    Web Server
          |             |             |
          +-------------+-------------+
                        |
                    Cache/Database
```

Each layer absorbs or filters malicious traffic before it reaches the application.

---

# Prevention techniques

## 1. Content Delivery Network (CDN)

A CDN serves static content from edge locations around the world.

Benefits:

- Absorbs large traffic spikes
- Reduces origin server load
- Distributes traffic geographically

If attackers request static assets, edge servers often handle the requests without contacting the origin.

---

## 2. DDoS protection service

Specialized providers operate large global networks capable of absorbing and filtering attack traffic before forwarding legitimate requests.

Capabilities include:

- Traffic scrubbing
- Anycast routing
- Automatic attack detection
- Rate-based filtering

---

## 3. Rate limiting

Limit how many requests a client can make.

Example:

```text
100 requests/minute/IP
```

If exceeded:

```http
429 Too Many Requests
```

Common algorithms:

- Token Bucket
- Leaky Bucket
- Sliding Window

Rate limiting is especially effective against Layer 7 attacks.

---

## 4. Load balancing

Distribute requests across multiple servers.

```text
             Load Balancer
          /        |        \
      Server1   Server2   Server3
```

Benefits:

- Prevents a single server from becoming a bottleneck
- Improves availability
- Supports horizontal scaling

---

## 5. Auto-scaling

Automatically add servers during traffic spikes.

```text
Traffic Spike

↓

Auto Scaling

↓

More Servers
```

Auto-scaling helps with legitimate surges and moderate attacks but **cannot withstand unlimited attack traffic on its own**.

---

## 6. Web Application Firewall (WAF)

A WAF analyzes HTTP requests and blocks suspicious patterns.

Examples:

- SQL injection attempts
- Cross-site scripting (XSS)
- Requests from known malicious IPs
- Abnormal request rates

---

## 7. Caching

Serve frequently requested data from cache instead of repeatedly hitting databases or application servers.

```text
User

↓

Cache

↓

Application (only on cache miss)
```

Caching reduces the cost of repeated requests.

---

## 8. Traffic filtering

Filter based on:

- IP reputation
- Geographic origin (when appropriate)
- User-Agent patterns
- Known bot signatures
- Network behavior

Suspicious traffic can be blocked or challenged before reaching the application.

---

## 9. CAPTCHAs and bot detection

For endpoints like login or signup:

```text
User

↓

CAPTCHA

↓

Application
```

This makes automated attacks more expensive while allowing human users to proceed.

---

# Monitoring and detection

Monitor metrics such as:

- Requests per second (RPS)
- Bandwidth usage
- Error rates
- Latency
- CPU and memory utilization
- Active connections
- Geographic traffic distribution

Anomalies such as sudden spikes or traffic from unexpected regions can trigger alerts and mitigation.

---

# Trade-offs

| Technique               | Advantages                                        | Limitations                                     |
| ----------------------- | ------------------------------------------------- | ----------------------------------------------- |
| CDN                     | Excellent for static content, global distribution | Doesn't fully protect dynamic endpoints         |
| Rate limiting           | Simple and effective against abuse                | May affect legitimate high-volume clients       |
| Load balancing          | Improves scalability and availability             | Doesn't block malicious traffic                 |
| Auto-scaling            | Handles traffic growth                            | Can increase costs during attacks               |
| WAF                     | Filters many application-layer attacks            | Cannot stop very large volumetric attacks alone |
| DDoS protection service | Best defense against large attacks                | Additional cost and operational dependency      |

---

# Security / observability

- Maintain real-time dashboards for traffic volume, request rates, and error rates.
- Set alerts for unusual spikes in RPS, bandwidth, or connection counts.
- Collect logs from CDNs, WAFs, load balancers, and application servers for correlation.
- Use distributed tracing to identify bottlenecks during Layer 7 attacks.
- Regularly test incident response and DDoS mitigation procedures.

---

# Interview-ready summary

> **A DDoS attack attempts to make a service unavailable by overwhelming it with traffic from many distributed machines. I would defend against it using multiple layers: a CDN and dedicated DDoS protection service to absorb and filter traffic, a WAF to block malicious HTTP requests, rate limiting to control abusive clients, load balancing and auto-scaling for resilience, caching to reduce backend load, and continuous monitoring to detect and respond to attacks quickly. This defense-in-depth approach protects against volumetric, protocol, and application-layer DDoS attacks.**

## Question 6. What is token-based authentication vs session-based?

## Question 7. How do you design a system for GDPR compliance?

## Question 8. How do you design a stock trading platform like Zerodha/Robinhood?

## Question 9. How do you design a blockchain-based system?

## Question 10. How do you design a cryptocurrency exchange (like Binance/Coinbase)?

## Question 11. How do you design an IoT device management platform?

## Question 12. How do you design a ride-hailing surge pricing system (Uber/Ola style)?

## Question 13. How do you design a global CDN from scratch?

## Question 14. How do you design a disaster recovery system for a bank?

## Question 15. How do you design a fraud detection system for payments?

## Question 16. How do you design a voice assistant system like Alexa/Siri?

## Question 17. How do you design a healthcare records management system?
