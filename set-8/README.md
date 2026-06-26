# Set 8

| S.No. | Question                                                                                                                                          |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What is DNS and how does it work in system design?](#question-1-what-is-dns-and-how-does-it-work-in-system-design)                               |
| 2.    | [How do you design a DNS resolution system?](#question-2-how-do-you-design-a-dns-resolution-system)                                               |
| 3.    | [What is load balancing at Layer 4 vs Layer 7?](#question-3-what-is-load-balancing-at-layer-4-vs-layer-7)                                         |
| 4.    | [What is sticky session in load balancing?](#question-4-what-is-sticky-session-in-load-balancing)                                                 |
| 5.    | [How do you design a system with WebRTC support?](#question-5-how-do-you-design-a-system-with-webrtc-support)                                     |
| 6.    | [What is multicast vs unicast vs broadcast?](#question-6-what-is-multicast-vs-unicast-vs-broadcast)                                               |
| 7.    | [How do you design a distributed logging pipeline?](#question-7-how-do-you-design-a-distributed-logging-pipeline)                                 |
| 8.    | [How do you design a system for flash sales (like Big Billion Day)?](#question-8-how-do-you-design-a-system-for-flash-sales-like-big-billion-day) |
| 9.    | [How do you handle high write throughput in databases?](#question-9-how-do-you-handle-high-write-throughput-in-databases)                         |
| 10.   | [How do you reduce read latency in distributed systems?](#question-10-how-do-you-reduce-read-latency-in-distributed-systems)                      |
| 11.   | [What is read-through vs write-through caching?](#question-11-what-is-read-through-vs-write-through-caching)                                      |
| 12.   | [How do you prevent cache stampede?](#question-12-how-do-you-prevent-cache-stampede)                                                              |
| 13.   | [What is cache invalidation and why is it hard?](#question-13-what-is-cache-invalidation-and-why-is-it-hard)                                      |
| 14.   | [How do you design a system with auto-scaling?](#question-14-how-do-you-design-a-system-with-auto-scaling)                                        |
| 15.   | [What are strategies for database sharding?](#question-15-what-are-strategies-for-database-sharding)                                              |
| 16.   | [How do you design a system for geo-distributed users?](#question-16-how-do-you-design-a-system-for-geo-distributed-users)                        |
| 17.   | [What is leader-follower replication?](#question-17-what-is-leader-follower-replication)                                                          |
| 18.   | [What is chaos engineering?](#question-18-what-is-chaos-engineering)                                                                              |
| 19.   | [How do you test fault tolerance in a distributed system?](#question-19-how-do-you-test-fault-tolerance-in-a-distributed-system)                  |
| 20.   | [How do you ensure data durability in storage systems?](#question-20-how-do-you-ensure-data-durability-in-storage-systems)                        |

## Question 1. What is DNS and how does it work in system design?

## Direct answer

**DNS (Domain Name System)** is the internet's **phonebook**. It translates a human-readable domain name (e.g., `google.com`) into an IP address (e.g., `142.x.x.x`) that computers use to communicate.

In system design, DNS is a critical component because it enables:

- Service discovery
- Global traffic routing
- Load balancing
- High availability
- Disaster recovery

Without DNS, users would need to remember IP addresses instead of domain names.

---

# How DNS works

When a user enters `www.example.com` into a browser, the following steps occur:

```
User
  │
  ▼
Browser Cache
  │
  ▼
OS DNS Cache
  │
  ▼
Recursive DNS Resolver (ISP/Google/Cloudflare)
  │
  ├── Root DNS Server
  │
  ├── TLD Server (.com)
  │
  └── Authoritative DNS Server
             │
             ▼
       Returns IP Address
             │
             ▼
Browser connects to Server
```

---

## Step-by-step lookup

### 1. User enters a URL

```
https://www.example.com
```

The browser needs the IP address before it can send an HTTP request.

---

### 2. Browser checks local cache

Browsers cache DNS results.

```
www.example.com
→ 192.168.10.20
```

If found and not expired, no network request is needed.

---

### 3. OS checks DNS cache

If the browser cache misses, the operating system checks its DNS cache.

---

### 4. Recursive DNS Resolver

If the IP is still unknown, the request goes to a recursive resolver.

Examples:

- Google DNS (8.8.8.8)
- Cloudflare DNS (1.1.1.1)
- ISP DNS servers

The resolver performs the lookup on behalf of the client.

---

### 5. Root DNS Server

The resolver asks:

> "Where can I find `.com`?"

The root server doesn't know the IP but responds with the location of the `.com` TLD servers.

---

### 6. TLD (Top-Level Domain) Server

The resolver asks:

> "Where is `example.com`?"

The `.com` server responds with the authoritative DNS server for that domain.

---

### 7. Authoritative DNS Server

The resolver asks:

> "What is the IP address of `www.example.com`?"

The authoritative server returns:

```
www.example.com
→ 203.0.113.10
```

---

### 8. Browser connects to the server

Now the browser knows the IP address.

```
Browser
      │
      ▼
203.0.113.10
      │
      ▼
HTTP/HTTPS Request
```

The webpage loads.

---

# DNS caching

DNS uses caching heavily to reduce lookup latency.

| Cache Location     | Purpose                          |
| ------------------ | -------------------------------- |
| Browser            | Avoid repeated lookups           |
| Operating System   | Shared by applications           |
| Recursive Resolver | Speeds up lookups for many users |
| CDN/Proxy          | Can cache DNS internally         |

Each DNS record has a **TTL (Time To Live)**.

Example:

```
TTL = 300 seconds
```

The resolver can reuse the IP for 5 minutes before querying again.

---

# Common DNS record types

| Record | Purpose                 | Example                   |
| ------ | ----------------------- | ------------------------- |
| A      | Domain → IPv4           | example.com → 192.0.2.1   |
| AAAA   | Domain → IPv6           | example.com → 2001:db8::1 |
| CNAME  | Alias to another domain | www → app.example.com     |
| MX     | Mail server             | mail.example.com          |
| TXT    | Verification/SPF/DKIM   | Google verification       |
| NS     | Name servers            | ns1.example.com           |

---

# DNS in system design

DNS is much more than name resolution—it plays a major role in routing and reliability.

## 1. Load balancing

One domain can resolve to multiple IP addresses.

```
example.com

↓

10.0.0.1
10.0.0.2
10.0.0.3
```

DNS can rotate the returned IPs (**Round Robin DNS**), distributing requests across servers.

---

## 2. Geographic routing

Users are directed to the nearest data center.

```
India
    ↓
Mumbai DC

Europe
    ↓
Frankfurt DC

US
    ↓
Virginia DC
```

This reduces latency and improves user experience.

---

## 3. High availability

If one data center fails, DNS can redirect users to another.

```
Primary
    ↓
Unavailable

DNS

↓

Backup Region
```

This supports disaster recovery.

---

## 4. CDN integration

When users access:

```
images.example.com
```

DNS can resolve it to the nearest CDN edge server instead of the origin server.

```
User

↓

Nearest CDN

↓

Origin (if needed)
```

This reduces latency and origin load.

---

## 5. Service migration

During infrastructure changes:

```
Old Server

↓

DNS Update

↓

New Server
```

Clients automatically start reaching the new infrastructure once cached records expire.

---

# DNS-based load balancing vs Load Balancer

| DNS Load Balancing            | Hardware/Software Load Balancer |
| ----------------------------- | ------------------------------- |
| Happens before connection     | Happens after connection        |
| Returns server IP             | Routes individual requests      |
| Simple                        | Intelligent                     |
| Can't detect instant failures | Performs active health checks   |
| Uses TTL                      | Real-time decisions             |

Large systems often use both:

```
User
   │
DNS
   │
Regional Load Balancer
   │
Application Servers
```

---

# Challenges with DNS

### DNS caching delays

If an IP changes:

```
Old IP
↓

Cached for TTL

↓

Traffic continues to old server
```

Propagation isn't instantaneous.

---

### Health checking limitations

Basic DNS doesn't know immediately whether a server is down unless combined with health-aware DNS services.

---

### DDoS attacks

DNS infrastructure is a common target for attacks.

Mitigations include:

- Anycast routing
- Distributed authoritative DNS
- Rate limiting
- Multiple DNS providers

---

# Interview-ready summary

> **DNS is a distributed, hierarchical naming system that translates domain names into IP addresses. A DNS lookup typically goes from local caches to a recursive resolver, then to the root server, TLD server, and finally the authoritative name server. In system design, DNS is essential not only for name resolution but also for global traffic routing, load balancing, CDN integration, failover, and high availability. Because DNS responses are cached using TTLs, changes are not reflected instantly, so DNS is commonly combined with load balancers and health checks to build highly available, scalable systems.**

## Question 2. How do you design a DNS resolution system?

## Question 3. What is load balancing at Layer 4 vs Layer 7?

## Question 4. What is sticky session in load balancing?

## Question 5. How do you design a system with WebRTC support?

## Question 6. What is multicast vs unicast vs broadcast?

## Question 7. How do you design a distributed logging pipeline?

## Question 8. How do you design a system for flash sales (like Big Billion Day)?

## Question 9. How do you handle high write throughput in databases?

## Question 10. How do you reduce read latency in distributed systems?

## Question 11. What is read-through vs write-through caching?

## Question 12. How do you prevent cache stampede?

## Question 13. What is cache invalidation and why is it hard?

## Question 14. How do you design a system with auto-scaling?

## Question 15. What are strategies for database sharding?

## Question 16. How do you design a system for geo-distributed users?

## Question 17. What is leader-follower replication?

## Question 18. What is chaos engineering?

## Question 19. How do you test fault tolerance in a distributed system?

## Question 20. How do you ensure data durability in storage systems?
