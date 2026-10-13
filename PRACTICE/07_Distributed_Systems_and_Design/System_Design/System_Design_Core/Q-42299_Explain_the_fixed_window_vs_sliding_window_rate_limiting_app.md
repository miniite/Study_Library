---
type: "practice"
difficulty: "Intermediate"
---

# Explain the fixed window vs sliding window rate limiting approaches and their trade-offs.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Rate Limiter Storage](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Rate_Limiter_Storage/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In system design, rate limiting is a crucial mechanism to control the number of requests to an API or service, preventing overload and ensuring fair usage. This discussion will explore fixed window vs sliding window rate limiting approaches and their trade-offs from first principles.

Firstly, fixed window rate limiting divides time into discrete intervals (e.g., 1 minute), allowing a specified number of requests within each interval. For example, setting the limit to 50 requests per minute means that no more than 50 requests can be processed in any given minute. This approach is simple and easy to implement but has certain limitations.

Sliding window rate limiting, on the other hand, considers a moving time frame of a fixed duration (e.g., 1 minute), where request counts are tracked over this period. Requests that fall within the current window can be processed even if the total exceeds the limit in previous windows. This approach offers better responsiveness and fairness as it considers recent activity.

Now, let's dive into the trade-offs between these approaches:

Fixed Window Rate Limiting:

#### Pros
Simplicity: Easy to implement with straightforward logic.
Predictability: Clients can easily determine if their request will be allowed based on previous intervals.

#### Cons
Inefficiency: Can lead to frequent rate limit resets, causing bursts of requests just before the reset, followed by none after. This can result in underutilization and unfairness.

Sliding Window Rate Limiting:

#### Pros
Fairness: Better handles bursty traffic as it considers recent activity.
Responsiveness: Allows more fluid handling of traffic patterns without frequent resets.

#### Cons
Complexity: More intricate to implement due to the need for maintaining a sliding window state.
Memory Usage: Requires storage to keep track of request counts over time, which can increase memory usage.

To illustrate these concepts, consider a scenario where we have 100 requests per minute as our rate limit. In fixed window:

If at 59 seconds, we get the 60th request, it will be rejected until the next minute.
At 29 seconds, if we get the 70th request, it is immediately accepted.

In sliding window:

If at 31 seconds, we get the 80th request and 10 more requests come in between 59-60 seconds, all of these will be rejected.
If a burst comes in just before 29 seconds, they might all pass due to the sliding nature.

In summary, fixed window rate limiting is simpler but can be less efficient for handling bursts. Sliding window rate limiting provides better responsiveness and fairness but at the cost of complexity and increased memory usage.

#### Common mistakes
include:

Overlooking burst traffic in fixed windows.
Not accounting for recent activity in sliding windows, which could lead to over-utilization or under-utilization issues.

**Comparison:**

```
Rate Limiter Type	Fixed Window	Sliding Window
Logic	Count in fixed intervals	Track recent activity
Complexity	Simple	More complex
Fairness	Less	Better
Under/Over Utilization	Can lead to bursts and resets	Handles bursts better but may over-utilize if not managed properly
```

**Code:**

```
Example implementation of a simple fixed window rate limiter

class FixedWindowRateLimiter: def init(self, limit): self.limit = limit self.requests = 0

CODE
Copy
def allow_request(self):
    if self.requests < self.limit:
        self.requests += 1
        return True
    else:
        return False
CODE
Copy

```python
# Example implementation of a simple sliding window rate limiter

class SlidingWindowRateLimiter:
    def __init__(self, limit, window_size):
        self.limit = limit
        self.window_size = window_size
        self.requests = 0

    def allow_request(self, timestamp):
        # Simplified logic for demonstration purposes
        if self.requests < self.limit and (timestamp - self.last_timestamp) <= self.window_size:
            self.requests += 1
            return True
        else:
            return False
```

**Example input / output:**

```text
No specific input-output examples are needed for these concepts, but here’s a conceptual dry-run:

- **Fixed Window**: At 59 seconds, 60th request -> rejected. At 29 seconds, 70th request -> accepted.

- **Sliding Window**: At 31 seconds, 80th request + 10 more requests (from 59 to 60) -> all rejected.
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Fixed Window Rate Limiter
6. Check current request count
7. Current requests < limit?
8. Allow request
9. Reject request and wait fornext interval
10. Track recent activity overwindow size
11. Current requests in window <limit?
12. Allow request
13. Reject request based on slidingwindow count
14. Yes
15. No
16. Sliding Window Rate Limiter
17. Track recent activity overwindow size
18. Current requests in window <limit?
19. Allow request
20. Reject request based on slidingwindow count

