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

# What is a Man-in-the-Middle (MITM) Attack?

## Direct answer

A **Man-in-the-Middle (MITM) attack** is a cyberattack where an attacker secretly intercepts and possibly modifies communication between two parties who believe they are communicating directly.

The attacker can:

- Eavesdrop on sensitive data (passwords, credit card numbers, session tokens)
- Modify messages in transit
- Inject malicious content
- Impersonate either party

---

# How a MITM Attack Works

### Normal Communication

```text
Client -----------------------> Server
         Secure communication
```

---

### During a MITM Attack

```text
Client ---------> Attacker ---------> Server
         <---------         <---------
```

The attacker sits between the client and server, relaying traffic so that neither side realizes the communication has been intercepted.

---

# Example Scenario

Suppose a user connects to public Wi-Fi.

```text
Laptop
   |
Public Wi-Fi
   |
Attacker
   |
Bank Server
```

If the connection is not properly secured:

1. The user sends login credentials.
2. The attacker intercepts them.
3. The attacker forwards the request to the bank.
4. The user logs in successfully and notices nothing unusual.
5. Meanwhile, the attacker has stolen the credentials.

---

# Common Types of MITM Attacks

### 1. Wi-Fi Eavesdropping

An attacker creates a fake public hotspot.

```text
Free Airport Wi-Fi
```

Users connect, and all their traffic passes through the attacker's device.

---

### 2. ARP Spoofing

On a local network, the attacker sends forged Address Resolution Protocol (ARP) messages to associate their device with the gateway's IP address.

```text
Victim
   |
Attacker
   |
Router
```

The victim unknowingly sends network traffic to the attacker first.

---

### 3. DNS Spoofing

The attacker manipulates Domain Name System (DNS) responses.

```text
bank.com
      ↓

Attacker redirects to

fake-bank.com
```

The victim may believe they are on the legitimate site.

---

### 4. SSL/TLS Stripping

The attacker downgrades a secure HTTPS connection to unencrypted HTTP.

```text
Browser
    |
HTTPS
    |

Attacker

    |
HTTP
    |
Server
```

Without encryption, the attacker can read and modify traffic.

---

### 5. Session Hijacking

Instead of stealing passwords, the attacker steals a session cookie or authentication token.

```text
Login
    |
Session Cookie
    |
Attacker steals cookie
    |
Attacker impersonates user
```

---

# What Can an Attacker Steal?

- Usernames
- Passwords
- Session cookies
- Authentication tokens
- Credit card details
- Personal information
- API keys
- Private messages

---

# How to Prevent MITM Attacks

## 1. Use HTTPS Everywhere

All communication should be encrypted using Transport Layer Security (TLS).

```text
Client
    |
Encrypted
    |
Server
```

Even if traffic is intercepted, the attacker cannot easily read its contents.

---

## 2. Validate TLS Certificates

Clients should verify that the server's certificate:

- Is signed by a trusted Certificate Authority (CA)
- Matches the domain name
- Has not expired or been revoked

Browsers warn users when certificate validation fails.

---

## 3. Enable HSTS

HTTP Strict Transport Security (HSTS) forces browsers to use HTTPS and prevents downgrade attacks such as SSL stripping.

---

## 4. Certificate Pinning (When Appropriate)

Applications can be configured to trust only specific server certificates or public keys, making forged certificates ineffective. While useful in some controlled environments, it requires careful operational management.

---

## 5. Secure Wi-Fi

- Use encrypted Wi-Fi (WPA2/WPA3)
- Avoid untrusted public hotspots
- Verify the network before connecting

---

## 6. Use a VPN on Untrusted Networks

A VPN encrypts traffic between the user's device and the VPN server, reducing the risk of interception on public networks.

---

## 7. Multi-Factor Authentication (MFA)

Even if an attacker steals a password, MFA makes it much harder to access the account without the second authentication factor.

---

# MITM in System Design

When designing secure distributed systems:

- Enforce HTTPS/TLS for all APIs and internal service communication where appropriate.
- Encrypt sensitive data in transit.
- Validate certificates on both clients and servers.
- Secure session tokens (e.g., `Secure` and `HttpOnly` cookies).
- Use short-lived access tokens and support token revocation.
- Monitor for suspicious login activity and unusual network behavior.

---

# Trade-offs

| Security Measure    | Advantages                                    | Limitations                                                |
| ------------------- | --------------------------------------------- | ---------------------------------------------------------- |
| TLS/HTTPS           | Encrypts data in transit                      | Requires proper certificate management                     |
| HSTS                | Prevents HTTPS downgrade attacks              | Only effective after browser support/policy is established |
| VPN                 | Protects traffic on untrusted networks        | Doesn't protect against compromised endpoints              |
| MFA                 | Limits damage from stolen credentials         | Adds some user friction                                    |
| Certificate Pinning | Strong protection against forged certificates | Operational complexity during certificate rotation         |

---

# Interview-ready Summary

> A Man-in-the-Middle (MITM) attack occurs when an attacker secretly intercepts communication between a client and a server, allowing them to read or modify data without either party's knowledge. The primary defense is enforcing TLS/HTTPS with proper certificate validation, complemented by HSTS, secure session management, MFA, and encrypted communication across all network paths. In system design, protecting data **in transit** is a fundamental security requirement to mitigate MITM attacks.

## Question 4. What is CSRF and how do you prevent it?

# What is CSRF and how do you prevent it?

## Direct answer

**Cross-Site Request Forgery (CSRF)** is a web security attack in which a malicious website tricks a user's browser into sending an unwanted request to another website where the user is already authenticated.

The attack succeeds because the browser **automatically includes authentication credentials** (such as cookies) with requests, causing the target website to believe the request came from the legitimate user.

---

# How CSRF Works

Suppose a user is logged into a banking website.

```text
User Login
      |
Browser stores session cookie
      |
Cookie automatically sent
with every request
```

The user then visits a malicious website.

```text
Victim Browser
       |
Malicious Website
       |
Hidden request
       |
Bank Website
```

The malicious site silently submits a request like:

```http
POST /transfer
amount=10000
to=attacker
```

Since the browser automatically attaches the bank's session cookie, the bank sees the request as coming from the authenticated user.

---

# Example Attack

### Step 1: User logs in

```text
bank.com
```

Browser stores:

```text
Session Cookie
```

---

### Step 2: User visits a malicious site

```text
evil.com
```

Hidden HTML:

```html
<form action="https://bank.com/transfer" method="POST">
  <input type="hidden" name="amount" value="10000" />
  <input type="hidden" name="to" value="attacker" />
</form>

<script>
  document.forms[0].submit();
</script>
```

The browser sends:

```text
POST /transfer
Cookie: SESSION=abc123
```

The bank receives a valid session cookie and may process the transfer unless CSRF protections are in place.

---

# Why CSRF Happens

CSRF relies on two conditions:

1. The application uses **cookie-based authentication**.
2. The browser automatically sends those cookies with requests.

The attacker **cannot read** the response because of the browser's same-origin policy, but they can often trigger state-changing requests.

---

# How to Prevent CSRF

## 1. CSRF Tokens (Primary Defense)

Generate a unique, unpredictable token for each user session (or each form).

```text
Server
   |
Generate CSRF Token
   |
Store in session
   |
Send with HTML form
```

Form:

```html
<input type="hidden" name="csrf_token" value="X7A91BC" />
```

When the form is submitted:

```text
Browser
   |
POST
csrf_token=X7A91BC
   |
Server
   |
Validate token
```

If the token is missing or invalid, reject the request.

### Why it works

A malicious website cannot read the legitimate site's pages, so it cannot obtain the correct CSRF token to include in the forged request.

---

## 2. SameSite Cookies

Modern browsers support the `SameSite` cookie attribute.

```http
Set-Cookie:
SESSION=abc123;
SameSite=Lax
```

Options:

| Setting  | Protection                                                                              |
| -------- | --------------------------------------------------------------------------------------- |
| `Strict` | Strongest CSRF protection; cookies are not sent on cross-site requests.                 |
| `Lax`    | Good default; protects most state-changing requests while preserving common navigation. |
| `None`   | Cookies are sent cross-site; must also use `Secure`.                                    |

This significantly reduces CSRF risk.

---

## 3. Check Origin or Referer Headers

For sensitive requests, verify the `Origin` (preferred) or `Referer` header.

Example:

```text
Origin: https://bank.com
```

Reject requests from unexpected origins.

---

## 4. Use Custom Headers for APIs

For AJAX requests:

```http
X-CSRF-Token: abc123
```

or

```http
X-Requested-With: XMLHttpRequest
```

A malicious website cannot generally add arbitrary custom headers in a simple cross-site form submission.

---

## 5. Avoid GET for State Changes

Never use `GET` for operations like:

```text
/deleteUser
/transferMoney
/updateProfile
```

Use:

```text
POST
PUT
PATCH
DELETE
```

GET requests should be safe and idempotent.

---

# CSRF vs XSS

| CSRF                                                        | XSS                                                                                          |
| ----------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| Tricks a user's browser into sending authenticated requests | Injects malicious JavaScript into a trusted website                                          |
| Exploits browser trust in the target site                   | Exploits user trust in the website                                                           |
| Doesn't require reading the response                        | Can read and modify page content                                                             |
| Mitigated with CSRF tokens and `SameSite` cookies           | Mitigated with output encoding, input validation, and a strong Content Security Policy (CSP) |

---

# CSRF in JWT-Based Authentication

If a JWT is stored in:

- **Cookies** → CSRF is still a concern because cookies are sent automatically.
- **Authorization header** (e.g., `Bearer <token>`) → Traditional CSRF is generally **not** a concern because the browser does not automatically attach the header.

However, storing tokens in browser-accessible storage (like `localStorage`) introduces different risks, particularly from XSS attacks.

---

# System Design Considerations

In production systems:

- Enable `SameSite=Lax` or `Strict` for session cookies where possible.
- Use `Secure` and `HttpOnly` cookie attributes.
- Protect all state-changing endpoints with CSRF tokens if using cookie-based authentication.
- Validate `Origin` headers for sensitive operations.
- Avoid state-changing GET endpoints.
- Use short session lifetimes and support session/token revocation.

---

# Trade-offs

| Technique                 | Advantages                          | Limitations                                          |
| ------------------------- | ----------------------------------- | ---------------------------------------------------- |
| CSRF Tokens               | Strong, widely adopted protection   | Requires server-side validation and token management |
| SameSite Cookies          | Built into browsers, easy to deploy | May affect legitimate cross-site workflows           |
| Origin/Referer Validation | Simple additional defense           | Headers may be absent in some situations             |
| Custom Headers            | Effective for AJAX APIs             | Not applicable to traditional HTML form submissions  |

---

# Interview-ready Summary

> CSRF is an attack where a malicious website causes a victim's browser to send authenticated requests to another site using automatically attached cookies. The primary defense is **CSRF tokens**, supplemented by **`SameSite` cookies**, **Origin/Referer validation**, and ensuring that **GET requests never perform state-changing actions**. In modern systems using cookie-based sessions, combining these defenses provides robust protection against CSRF attacks.

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
