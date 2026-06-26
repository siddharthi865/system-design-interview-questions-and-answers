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

## Question 3. What is secret management in microservices?

## Question 4. How do you prevent SQL injection at scale?

## Question 5. What is a DDoS attack and how do you prevent it?

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
