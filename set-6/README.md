# Set 6

| S.No. | Question                                                                                                                                                              |
| ----- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What is the difference between throughput and bandwidth?](#question-1-what-is-the-difference-between-throughput-and-bandwidth)                                       |
| 2.    | [What are bottlenecks in a system? How do you identify and fix them?](#question-2-what-are-bottlenecks-in-a-system-how-do-you-identify-and-fix-them)                  |
| 3.    | [What is a Single Point of Failure (SPOF)?](#question-3-what-is-a-single-point-of-failure-spof)                                                                       |
| 4.    | [What is the difference between synchronous and asynchronous communication?](#question-4-what-is-the-difference-between-synchronous-and-asynchronous-communication)   |
| 5.    | [What is the difference between online (OLTP) and analytical (OLAP) databases?](#question-5-what-is-the-difference-between-online-oltp-and-analytical-olap-databases) |
| 6.    | [What is a quorum in distributed systems?](#question-6-what-is-a-quorum-in-distributed-systems)                                                                       |
| 7.    | [What is data locality and why is it important?](#question-7-what-is-data-locality-and-why-is-it-important)                                                           |
| 8.    | [What is a proxy server? How does it help in scaling?](#question-8-what-is-a-proxy-server-how-does-it-help-in-scaling)                                                |
| 9.    | [Difference between reverse proxy and forward proxy](#question-9-difference-between-reverse-proxy-and-forward-proxy)                                                  |
| 10.   | [What is edge computing? How does it affect system design?](#question-10-what-is-edge-computing-how-does-it-affect-system-design)                                     |
| 11.   | [How do you design an online multiplayer game system?](#question-11-how-do-you-design-an-online-multiplayer-game-system)                                              |
| 12.   | [How do you design an online exam/test-taking platform?](#question-12-how-do-you-design-an-online-examtest-taking-platform)                                           |
| 13.   | [How do you design a real-time bidding system (like Google Ads)?](#question-13-how-do-you-design-a-real-time-bidding-system-like-google-ads)                          |
| 14.   | [How do you design a digital wallet system like Paytm/PhonePe?](#question-14-how-do-you-design-a-digital-wallet-system-like-paytmphonepe)                             |
| 15.   | [How do you design a microblogging platform like Twitter?](#question-15-how-do-you-design-a-microblogging-platform-like-twitter)                                      |
| 16.   | [How do you design a logging system for billions of events per day?](#question-16-how-do-you-design-a-logging-system-for-billions-of-events-per-day)                  |
| 17.   | [How do you design a recommendation system like Netflix?](#question-17-how-do-you-design-a-recommendation-system-like-netflix)                                        |
| 18.   | [How do you design a content moderation system for social media?](#question-18-how-do-you-design-a-content-moderation-system-for-social-media)                        |
| 19.   | [How do you design a search engine like Google?](#question-19-how-do-you-design-a-search-engine-like-google)                                                          |
| 20.   | [How do you design a collaborative whiteboard tool?](#question-20-how-do-you-design-a-collaborative-whiteboard-tool)                                                  |

## Question 1. What is the difference between throughput and bandwidth?

## Direct answer

**Bandwidth** is the **maximum amount of data that can be transmitted over a network or communication channel per unit time**, while **throughput** is the **actual amount of data successfully transmitted per unit time**.

Think of it this way:

- **Bandwidth = Capacity**
- **Throughput = Actual achieved performance**

Throughput is almost always **less than or equal to bandwidth** due to protocol overhead, congestion, packet loss, latency, and hardware limitations.

---

## Intuition

Imagine a highway:

- **Bandwidth** = Number of lanes on the highway.
- **Throughput** = Number of cars that actually reach the destination every minute.

Even with an 8-lane highway (high bandwidth), traffic jams or accidents can reduce the number of cars arriving (lower throughput).

---

## Comparison

| Aspect     | Bandwidth                    | Throughput                                           |
| ---------- | ---------------------------- | ---------------------------------------------------- |
| Meaning    | Maximum theoretical capacity | Actual successful data transfer rate                 |
| Nature     | Theoretical/Configured       | Measured in real-world conditions                    |
| Unit       | bps (Mbps, Gbps)             | bps (Mbps, Gbps)                                     |
| Depends on | Link capacity                | Network conditions, congestion, latency, packet loss |
| Value      | Upper limit                  | Always ≤ Bandwidth                                   |

---

## Example

Suppose you have:

- Network link = **1 Gbps bandwidth**

### Ideal case

- No congestion
- No packet loss
- Efficient protocol

```
Bandwidth = 1 Gbps
Throughput ≈ 980 Mbps
```

(The small difference is due to protocol headers and overhead.)

### Congested network

```
Bandwidth = 1 Gbps
Throughput = 350 Mbps
```

Although the link can carry 1 Gbps, congestion limits the actual transfer rate.

---

## What affects throughput?

Several factors reduce throughput:

- **Network congestion**
- **Packet loss and retransmissions**
- **High latency (especially TCP connections)**
- **Protocol overhead (TCP/IP headers, TLS, etc.)**
- **Server processing capacity**
- **Client processing speed**
- **Disk read/write speed**
- **Load balancer or firewall bottlenecks**

---

## System design relevance

In system design interviews, understanding the distinction helps identify bottlenecks:

- **Bandwidth bottleneck:** The network link itself cannot carry more data.
- **Throughput bottleneck:** The system is unable to utilize the available bandwidth due to software, infrastructure, or network inefficiencies.

For example:

- A storage service may have a **10 Gbps network interface** (high bandwidth).
- But if the database can only read data at **2 Gbps**, the overall throughput is limited to **2 Gbps**.

---

## Related concept: Throughput vs Latency

These are often discussed together but measure different things.

| Metric         | Measures                                                     |
| -------------- | ------------------------------------------------------------ |
| **Latency**    | Time taken for a single request to complete                  |
| **Throughput** | Number of requests or amount of data processed per unit time |

Example:

- API latency = **50 ms/request**
- Throughput = **20,000 requests/second**

A system can have low latency but low throughput (fast individual requests, limited parallelism), or high throughput but higher latency (many concurrent requests with longer wait times).

---

## Interview-ready summary

> **Bandwidth is the maximum theoretical capacity of a communication channel, whereas throughput is the actual amount of data successfully transferred over that channel. Throughput is always less than or equal to bandwidth because of factors like congestion, latency, protocol overhead, and packet loss. In system design, bandwidth represents the network's potential, while throughput reflects the system's real-world performance.**

## Question 2. What are bottlenecks in a system? How do you identify and fix them?

## Question 3. What is a Single Point of Failure (SPOF)?

## Question 4. What is the difference between synchronous and asynchronous communication?

## Question 5. What is the difference between online (OLTP) and analytical (OLAP) databases?

## Question 6. What is a quorum in distributed systems?

## Question 7. What is data locality and why is it important?

## Question 8. What is a proxy server? How does it help in scaling?

## Question 9. Difference between reverse proxy and forward proxy

## Question 10. What is edge computing? How does it affect system design?

## Question 11. How do you design an online multiplayer game system?

## Question 12. How do you design an online exam/test-taking platform?

## Question 13. How do you design a real-time bidding system (like Google Ads)?

## Question 14. How do you design a digital wallet system like Paytm/PhonePe?

## Question 15. How do you design a microblogging platform like Twitter?

## Question 16. How do you design a logging system for billions of events per day?

## Question 17. How do you design a recommendation system like Netflix?

## Question 18. How do you design a content moderation system for social media?

## Question 19. How do you design a search engine like Google?

## Question 20. How do you design a collaborative whiteboard tool?
