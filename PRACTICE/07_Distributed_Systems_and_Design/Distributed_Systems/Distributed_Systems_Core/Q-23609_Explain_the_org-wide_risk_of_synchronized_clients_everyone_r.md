---
type: "practice"
difficulty: "Deep"
---

# Explain the org-wide risk of synchronized clients: everyone refreshing tokens on the hour, every cron at midnight, all TTLs expiring together. Give the general thundering-herd pattern and the standards (deterministic jitter, staggered TTLs, request smoothing) you'd mandate to prevent it by construction.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Gossip and Deduplication](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/Gossip_and_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In distributed systems, a common risk is the synchronized client behavior where multiple clients attempt to perform critical operations simultaneously, often leading to a "thundering herd" scenario. This can cause significant load spikes and resource contention, particularly when dealing with token expiration and refresh. For an org-wide system, ensuring that this doesn't happen requires careful design and implementation of strategies such as deterministic jitter, staggered TTLs, and request smoothing.

#### Triage

The question is essentially asking about the potential for a thundering herd problem in distributed systems, specifically when multiple clients refresh their tokens at the same time due to scheduled cron jobs. The interviewer is probing your understanding of synchronization issues in distributed environments and how to mitigate them using established practices and standards.

#### First-Principles Solution
Understanding the Thundering Herd Pattern

When all clients attempt to perform a critical operation simultaneously, such as refreshing tokens at a specific time (e.g., every hour or midnight), it can overwhelm the system. This pattern is known as the "thundering herd." In our context, this could lead to a sudden spike in load on the token server, database, and other services.

Mitigation Strategies
Deterministic Jitter: Clients introduce slight delays before performing critical operations based on their ID or some deterministic factor. This ensures that no two clients will perform the same operation at exactly the same time.
Staggered TTLs: Instead of having all tokens expire at the same time, tokens can have staggered expiration times. For example, if there are 10 users, each token could be set to expire after a different number of minutes (e.g., User 1: 35m, User 2: 40m, etc.), thus spreading out their refresh times.
Request Smoothing: Implement rate limiting or queuing mechanisms at the client level to ensure that tokens are refreshed in a controlled and smooth manner rather than all at once.
Design Considerations
Load Distribution: Ensure that token servers can handle spikes by having adequate resources and possibly load balancing.
Database Stress Testing: Simulate high traffic scenarios to understand how the system performs under stress.
Monitoring and Alerts: Set up monitoring for abnormal traffic patterns and alerting mechanisms to quickly address any issues.

#### Closing Summary

To prevent the thundering herd pattern in a distributed system, you should mandate strategies like deterministic jitter, staggered TTLs, and request smoothing. These practices help spread out critical operations over time, reducing the likelihood of overwhelming the system with simultaneous requests.

#### Common Mistakes/Follow-Ups
Ignoring Edge Cases: Not considering scenarios where multiple clients might inadvertently synchronize their refresh times due to network latency or clock synchronization issues.
Overlooking Monitoring: Failing to set up robust monitoring and alerting for detecting and addressing high traffic conditions before they become problematic.
Not Considering the Impact on Other Services: Assuming that only token servers are affected, neglecting the broader impact on databases, APIs, and other critical services.

**Comparison:**

| Client ID | Expiry Time (Zulu)         |
|-----------|---------------------------|
| 1         | 2023-10-01T18:37:00Z      |
| 2         | 2023-10-01T18:40:00Z      |
| 3         | 2023-10-01T18:42:00Z      |

**Code:**

```python
# Example of deterministic jitter in Python
import time

def refresh_token(jitter):
    # Simulate a client refreshing its token with some delay
    time.sleep(jitter)
    print(f"Token refreshed after {jitter} seconds")

for i in range(10):  # Simulating multiple clients
    refresh_token(i * 5)  # Clients introduce jitter of 0, 5, 10, ..., 45 seconds
```

**Example input / output:**

```text
Input:
- Token IDs: [1, 2, 3]
- Expiry times: ['2023-10-01T18:37:00Z', '2023-10-01T18:40:00Z', '2023-10-01T18:42:00Z']

Output:
- Token 1 expires at 18:37
- Token 2 expires at 18:40
- Token 3 expires at 18:42

Impact:
- No thundering herd; tokens expire at staggered intervals, reducing load.
```

**Data:**

```json
{
  "tokens": [
    {"id": 1, "expiry_time": "2023-10-01T18:37:00Z"},
    {"id": 2, "expiry_time": "2023-10-01T18:40:00Z"},
    {"id": 3, "expiry_time": "2023-10-01T18:42:00Z"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Token Refresh
4. Is Client ID Deterministic?
5. Deterministic Jitter
6. Staggered TTLs
7. Spread out refresh timesevenly
8. Ensure tokens expire atstaggered intervals
9. Smooth Token Refreshes
10. Implement rate limiting orqueuing

