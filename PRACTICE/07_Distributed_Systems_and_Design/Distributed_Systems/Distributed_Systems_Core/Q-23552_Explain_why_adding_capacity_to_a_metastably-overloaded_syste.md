---
type: "practice"
difficulty: "Deep"
---

# Explain why adding capacity to a metastably-overloaded system can make it worse, not better, and enumerate the counter-intuitive first interventions (shed load, cap concurrency, admission control) you'd reach for before scaling out during an active incident.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Slow Dependencies and Load Shedding](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Deadline_Propagation/Slow_Dependencies_and_Load_Shedding.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

Adding capacity to a metastably-overloaded system can degrade performance due to issues like increased contention and degraded cache efficiency. To mitigate this, initial interventions should focus on shedding load, capping concurrency, and implementing admission control.

#### Triage

The core of the question is understanding why adding more resources (e.g., servers or CPU cores) might not help an already-overloaded distributed system and how to address the immediate symptoms effectively. The interviewer wants you to demonstrate your deep understanding of distributed systems, load balancing, resource management, and performance optimization.

#### First-Principles Solution

Understanding Metastability: A metastable state in a distributed system occurs when the system is operating under heavy load, but not crashing or showing immediate failures. This can lead to subtle degradation in performance as resources are strained to their limits.

Resource Contention: When adding more capacity (e.g., servers), you increase contention for shared resources such as databases, caches, and network bandwidth. Each additional node adds to the overhead of coordination and synchronization, which can reduce overall throughput.

Cache Efficiency: As the load increases, cache hits may decrease because the increased demand can cause more frequent invalidation or miss scenarios. This reduces the effectiveness of caching mechanisms, leading to a performance bottleneck even with more resources.

Load Distribution: Simply adding capacity doesn't necessarily distribute the load evenly. Without proper load balancers and monitoring, newly added nodes might not be utilized optimally, leading to underutilization of some resources while others remain overloaded.

First Interventions:

Shed Load: Prioritize reducing unnecessary requests or tasks to lower overall system load. This can be done by temporarily stopping non-critical operations or optimizing existing ones.
Cap Concurrency: Limit the number of concurrent operations that a client or service can perform, especially in rate-limited services. This helps prevent overwhelming back-end resources and ensures smoother operation under high demand.
Admission Control: Implement mechanisms to control which requests are allowed into the system during peak times. This might involve queuing or prioritizing critical tasks.

#### Closing Summary

When a distributed system is metastably overloaded, adding capacity alone can exacerbate issues due to increased resource contention and reduced cache efficiency. Instead of scaling out immediately, initial interventions should focus on shedding load, capping concurrency, and implementing admission control to stabilize the system before considering further scaling.

#### Common Mistakes/Follow-Ups
Mistake: Focusing solely on adding capacity without addressing immediate load issues.

#### Follow-Up
How do you measure and monitor system performance in real-time? What tools or metrics would you use to identify which interventions are most effective?
Mistake: Assuming that more resources will always help, even when the root cause is unbalanced load distribution.

#### Follow-Up
Can you explain how you would design a distributed cache with adaptive eviction policies to handle increased loads effectively?

**Comparison:**

```
Time Period	Requests
09:00-10:00	50
10:00-11:00	75
11:00-12:00	100
12:00-13:00	120
Cache Hits	Total Requests	Hit Rate
1200	1500	80%
```

**Code:**

```python
# Example of basic rate limiting in Python using a simple queue
from collections import deque

class RateLimiter:
    def __init__(self, max_requests):
        self.requests = deque()
        self.max_requests = max_requests

    def request_allowed(self):
        while self.requests and self.requests[0] < time.time() - 3600:  # Remove old requests
            self.requests.popleft()

        if len(self.requests) >= self.max_requests:
            return False
        else:
            self.requests.append(time.time())
            return True

# Example usage
limiter = RateLimiter(max_requests=100)
if limiter.request_allowed():
    print("Request is allowed")
else:
    print("Rate limit exceeded")
```

**Example input / output:**

```text
Input:
- A system experiencing high load with fluctuating request rates.
- Cache miss rate increasing as the number of requests grows.

Output:
- Implementation of a simple rate limiter to control concurrent requests.
- Introduction of caching strategies with dynamic eviction policies based on access patterns.
```

**Data:**

```json
{
  "requests_per_minute": {
    "09:00-10:00": 50,
    "10:00-11:00": 75,
    "11:00-12:00": 100,
    "12:00-13:00": 120
  },
  "cache_hits": {
    "total_requests": 1500,
    "hit_rate": 80
  }
}
```

**Flow:**

1. Yes
2. No
3. Initial Overload
4. Is Load Critical?
5. Implement Admission Control
6. Monitor and Optimize Cache
7. Stabilize System Before Scaling
8. Optimize Caching Policies

