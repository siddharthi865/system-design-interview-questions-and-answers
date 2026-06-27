# Set 15

| S.No. | Question                                                                                                                                                       |
| ----- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you design a secure password storage system?](#question-1-how-do-you-design-a-secure-password-storage-system)                                          |
| 2.    | [What is token revocation in authentication?](#question-2-what-is-token-revocation-in-authentication)                                                          |
| 3.    | [What is a man-in-the-middle (MITM) attack?](#question-3-what-is-a-man-in-the-middle-mitm-attack)                                                              |
| 4.    | [What is CSRF and how do you prevent it?](#question-4-what-is-csrf-and-how-do-you-prevent-it)                                                                  |
| 5.    | [How do you design a secure multi-tenant system?](#question-5-how-do-you-design-a-secure-multi-tenant-system)                                                  |
| 6.    | [What is TLS termination at load balancers?](#question-6-what-is-tls-termination-at-load-balancers)                                                            |
| 7.    | [How do you secure inter-service communication in microservices?](#question-7-how-do-you-secure-inter-service-communication-in-microservices)                  |
| 8.    | [What is database encryption key rotation?](#question-8-what-is-database-encryption-key-rotation)                                                              |
| 9.    | [How do you implement two-factor authentication in system design?](#question-9-how-do-you-implement-two-factor-authentication-in-system-design)                |
| 10.   | [What is differential privacy?](#question-10-what-is-differential-privacy)                                                                                     |
| 11.   | [How do you design a blockchain wallet?](#question-11-how-do-you-design-a-blockchain-wallet)                                                                   |
| 12.   | [How do you design a distributed ledger system?](#question-12-how-do-you-design-a-distributed-ledger-system)                                                   |
| 13.   | [How do you design a real-time fraud detection system?](#question-13-how-do-you-design-a-real-time-fraud-detection-system)                                     |
| 14.   | [How do you design a global stock ticker system?](#question-14-how-do-you-design-a-global-stock-ticker-system)                                                 |
| 15.   | [How do you design an API throttling (rate-limiting) system for partners?](#question-15-how-do-you-design-an-api-throttling-rate-limiting-system-for-partners) |
| 16.   | [How do you design a podcast recommendation engine?](#question-16-how-do-you-design-a-podcast-recommendation-engine)                                           |
| 17.   | [How do you design a ride-sharing pooling algorithm?](#question-17-how-do-you-design-a-ride-sharing-pooling-algorithm)                                         |
| 18.   | [How do you design a telemedicine platform?](#question-18-how-do-you-design-a-telemedicine-platform)                                                           |
| 19.   | [How do you design a digital classroom system?](#question-19-how-do-you-design-a-digital-classroom-system)                                                     |
| 20.   | [How do you design a global payment settlement system?](#question-20-how-do-you-design-a-global-payment-settlement-system)                                     |

## Question 1. How do you design a secure password storage system?

# How do you design a secure password storage system?

## Direct answer

A secure password storage system should **never store plaintext passwords or use reversible encryption**. Instead, store a **unique random salt** and a **slow, adaptive password hash** (such as Argon2, bcrypt, or scrypt). During login, hash the supplied password with the same parameters and compare it securely with the stored hash.

---

# Requirements / Problem Framing

### Functional Requirements

- User registration
- User login
- Password reset
- Password change
- Password verification

### Non-functional Requirements

- Strong resistance against database compromise
- Protection from brute-force and rainbow table attacks
- Scalable authentication
- Low latency for users while remaining computationally expensive for attackers

---

# High-Level Architecture

```text
                 Registration

Client
   |
   | Password
   v
Authentication Service
   |
Generate Random Salt
   |
Hash(password + salt)
   |
Store:
   - User ID
   - Salt
   - Hash
   - Hash algorithm/version
   |
Database
```

### Login Flow

```text
Client
   |
Username + Password
   |
Authentication Service
   |
Fetch salt + stored hash
   |
Hash(input password + salt)
   |
Constant-time comparison
   |
Success / Failure
```

---

# Database Schema

```text
Users
------
user_id
email
password_hash
salt
hash_algorithm
work_factor
created_at
updated_at
```

Example:

```text
user_id: 123
salt: x7T9...
algorithm: Argon2id
memory_cost: 64MB
iterations: 3
parallelism: 2
password_hash: 5af9...
```

Many modern password hash formats encode the salt and parameters within the stored hash string itself, so storing them in separate columns is optional.

---

# Password Hashing Process

### Registration

```text
User Password
      |
Generate Random Salt
      |
Password + Salt
      |
Argon2 / bcrypt / scrypt
      |
Password Hash
      |
Store Hash (+ salt/parameters)
```

---

### Login

```text
Entered Password
        |
Stored Salt
        |
Hash Again
        |
Compare
        |
Authenticated
```

---

# Why Salting Is Important

Suppose two users choose:

```text
Password = password123
```

Without salt:

```text
Hash(password123)
= ABC123
```

Both users have identical hashes.

An attacker immediately knows they use the same password.

---

With random salts:

```text
Salt1 + password123
→ Hash = X92AF

Salt2 + password123
→ Hash = P81KD
```

Same password, completely different hashes.

Benefits:

- Prevents rainbow table attacks
- Prevents identical hashes
- Forces attackers to crack passwords individually

---

# Choosing the Right Hash Algorithm

| Algorithm | Suitable?        | Notes                                                 |
| --------- | ---------------- | ----------------------------------------------------- |
| MD5       | ❌ No            | Extremely fast and broken for password storage        |
| SHA-1     | ❌ No            | Too fast and outdated                                 |
| SHA-256   | ❌ Alone, no     | Fast hashes are unsuitable for passwords              |
| bcrypt    | ✅ Yes           | Widely used and battle-tested                         |
| scrypt    | ✅ Yes           | Memory-hard, good GPU resistance                      |
| Argon2id  | ✅ Best practice | Memory-hard and currently recommended for new systems |

---

# Additional Security Measures

### 1. Pepper

A pepper is a secret value stored outside the database (for example, in a secret manager).

```text
Hash(password + salt + pepper)
```

If the database is stolen but the pepper is not, cracking becomes significantly harder.

---

### 2. Adaptive Cost

Increase hashing cost over time.

Example:

```text
bcrypt cost = 10
```

Later:

```text
bcrypt cost = 12
```

or increase Argon2 memory and iteration parameters.

This keeps pace with faster hardware.

---

### 3. Constant-Time Comparison

Avoid:

```text
hash1 == hash2
```

because naive comparisons may leak timing information.

Use constant-time comparison functions provided by cryptographic libraries.

---

### 4. Strong Password Policy

Require:

- Minimum length
- Common-password detection
- Password strength checks
- Support for long passphrases

---

### 5. Rate Limiting

Prevent brute-force attacks.

Example:

```text
5 failed logins
      ↓
Temporary lock

OR

Exponential backoff
```

---

### 6. Multi-Factor Authentication (MFA)

Even if a password is compromised:

```text
Password
      +
OTP / Authenticator App / Security Key
```

adds another layer of protection.

---

# Password Reset Flow

Never email passwords.

Instead:

```text
User requests reset
      |
Generate secure random token
      |
Store hashed token with expiration
      |
Email reset link
      |
User sets new password
      |
Generate new salt
      |
Hash new password
      |
Replace old hash
```

---

# Scalability Considerations

Authentication servers should be stateless.

```text
        Load Balancer
          /      \
      Auth      Auth
        |          |
      Shared Database
```

Benefits:

- Horizontal scaling
- No session affinity required
- Easy deployment

---

# Trade-offs

| Choice                | Advantages                               | Disadvantages                          |
| --------------------- | ---------------------------------------- | -------------------------------------- |
| Fast hashes (SHA-256) | Very fast                                | Easy for attackers to brute-force      |
| bcrypt                | Mature and widely supported              | Less memory-hard than newer algorithms |
| scrypt                | Good GPU resistance                      | More complex tuning                    |
| Argon2id              | Strong protection against modern attacks | Higher CPU and memory usage            |

---

# Security Considerations

- Never store plaintext passwords.
- Never encrypt passwords for later decryption.
- Always use a unique random salt.
- Prefer memory-hard password hashing.
- Protect peppers in a secrets manager.
- Enforce TLS so passwords are encrypted in transit.
- Log authentication events, but never log passwords or password hashes.
- Rotate hash parameters over time and rehash passwords upon successful login when older parameters are detected.

---

# Interview-ready Summary

> A secure password storage system stores only **salted, one-way password hashes** using adaptive algorithms like **Argon2id**, **bcrypt**, or **scrypt**. Each password gets a unique random salt, optional pepper protection, and configurable work factors. During login, the password is rehashed and compared using a constant-time comparison. Combined with rate limiting, MFA, secure password resets, and periodic parameter upgrades, this design remains resilient even if the database is compromised.

## Question 2. What is token revocation in authentication?

# What is token revocation in authentication?

## Direct answer

**Token revocation** is the process of **invalidating an authentication token before its natural expiration time**. It ensures that a token can no longer be used, even if it is otherwise valid.

This is important when:

- A user logs out
- A device is lost or stolen
- Credentials are compromised
- An administrator disables an account
- Permissions or roles change

---

# Why is Token Revocation Needed?

Consider a user with a JSON Web Token (JWT) that expires in 24 hours.

```text
9:00 AM  Login
        ↓
JWT issued (valid for 24 hours)

11:00 AM User logs out

Without revocation:
Token remains usable until next day
```

Even after logout, anyone holding the token can continue accessing protected resources until it expires.

Token revocation solves this by invalidating the token immediately.

---

# Common Revocation Strategies

## 1. Blacklisting Tokens

Maintain a list of revoked tokens.

```text
Request
    |
JWT
    |
Authentication Service
    |
Check blacklist
    |
Revoked?
  /      \
Yes      No
 |         |
Reject   Accept
```

### Database

```text
Revoked Tokens
---------------
token_id (jti)
expires_at
reason
revoked_at
```

### Pros

- Immediate revocation
- Simple to understand

### Cons

- Every request requires a blacklist lookup
- Additional storage and maintenance

---

## 2. Short-Lived Access Tokens (Most Common)

Issue:

- Access token: 10–15 minutes
- Refresh token: several days or weeks

```text
Access Token
15 min
      ↓
Expires quickly

Refresh Token
30 days
```

If a user logs out:

- Revoke the refresh token.
- Existing access token expires soon.

### Pros

- No blacklist lookup for every request
- Highly scalable
- Common in modern systems

### Cons

- Revocation is not immediate; access tokens remain valid until they expire.

---

## 3. Refresh Token Revocation

Only refresh tokens are stored server-side.

```text
Login
   |
Issue:
Access Token
Refresh Token
        |
Store Refresh Token
```

Logout:

```text
Delete Refresh Token
```

The current access token continues until expiration, but no new access tokens can be issued.

This is the most common approach for JWT-based authentication.

---

## 4. Token Versioning

Store a version number for each user.

```text
User
------
user_id
token_version = 5
```

JWT payload:

```json
{
  "userId": 123,
  "version": 5
}
```

On each request:

```text
JWT version == DB version ?
        |
      Yes → Allow
       No → Reject
```

To revoke all active tokens for a user:

```text
token_version++
```

All previously issued tokens become invalid.

### Pros

- Revokes all sessions instantly
- Simple implementation

### Cons

- Requires a database lookup (or cache lookup) on authenticated requests

---

## 5. Session Store (Opaque Tokens)

Instead of self-contained JWTs, issue a random session ID.

```text
Client
   |
Session ID
   |
Server
   |
Redis / Database
```

Logout:

```text
Delete Session
```

The token immediately becomes invalid because the server no longer recognizes it.

### Pros

- Immediate revocation
- Easy session management

### Cons

- Requires server-side state
- Less scalable than fully stateless JWTs

---

# Comparison

| Strategy                 | Immediate Revocation | Stateless          | Scalable  |
| ------------------------ | -------------------- | ------------------ | --------- |
| Blacklist                | ✅ Yes               | ❌ No              | Medium    |
| Short-lived JWT          | ❌ No                | ✅ Yes             | Excellent |
| Refresh token revocation | Partial              | Mostly             | Excellent |
| Token versioning         | ✅ Yes               | ❌ Requires lookup | Good      |
| Session store            | ✅ Yes               | ❌ No              | Good      |

---

# When to Revoke Tokens

Typical events include:

- User logs out
- Password is changed
- Password reset occurs
- Account is disabled
- User is deleted
- Suspicious login detected
- Device is reported lost
- User permissions or roles change

---

# Scalability Considerations

Large-scale systems typically:

- Use **short-lived access tokens** (5–15 minutes)
- Store **refresh tokens** in a database or cache like Redis
- Revoke refresh tokens on logout or compromise
- Use distributed caches for fast revocation checks when immediate invalidation is required

This minimizes database lookups while allowing efficient session management.

---

# Trade-offs

| Approach                            | Advantages                                     | Disadvantages                                                 |
| ----------------------------------- | ---------------------------------------------- | ------------------------------------------------------------- |
| Stateless JWTs                      | High performance, no server-side session state | Difficult to revoke immediately                               |
| Server-side sessions                | Easy revocation and fine-grained control       | Requires shared session storage                               |
| Blacklisting                        | Immediate invalidation of specific tokens      | Lookup overhead on each request                               |
| Short-lived access + refresh tokens | Excellent balance of security and scalability  | Small window where an access token remains valid after logout |

---

# Interview-ready Summary

> Token revocation is the ability to invalidate an authentication token before it expires. While self-contained JWTs are difficult to revoke immediately, common production systems address this by issuing **short-lived access tokens** and **server-managed refresh tokens**. Depending on security requirements, systems may also use **blacklists**, **token versioning**, or **server-side session stores** to support immediate revocation after logout, credential compromise, or account changes.

## Question 3. What is a man-in-the-middle (MITM) attack?

## Question 4. What is CSRF and how do you prevent it?

## Question 5. How do you design a secure multi-tenant system?

## Question 6. What is TLS termination at load balancers?

## Question 7. How do you secure inter-service communication in microservices?

## Question 8. What is database encryption key rotation?

## Question 9. How do you implement two-factor authentication in system design?

## Question 10. What is differential privacy?

## Question 11. How do you design a blockchain wallet?

## Question 12. How do you design a distributed ledger system?

## Question 13. How do you design a real-time fraud detection system?

## Question 14. How do you design a global stock ticker system?

## Question 15. How do you design an API throttling (rate-limiting) system for partners?

## Question 16. How do you design a podcast recommendation engine?

## Question 17. How do you design a ride-sharing pooling algorithm?

## Question 18. How do you design a telemedicine platform?

## Question 19. How do you design a digital classroom system?

## Question 20. How do you design a global payment settlement system?
