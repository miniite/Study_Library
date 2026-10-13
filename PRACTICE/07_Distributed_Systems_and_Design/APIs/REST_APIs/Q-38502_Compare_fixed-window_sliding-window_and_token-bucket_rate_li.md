---
type: "practice"
difficulty: "Expert"
---

# Compare fixed-window, sliding-window, and token-bucket rate limiting in terms of burst handling and fairness.

**Difficulty:** Expert · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ API Rate Limiting](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/API_Rate_Limiting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Rate limiting is a common technique to control request rates to an API or service, ensuring that no single user or client can flood the system with requests. In this context, we will compare fixed-window, sliding-window, and token-bucket rate limiting in terms of burst handling and fairness.

#### Triage

What the Question is Asking:

The interviewer wants to understand your knowledge of different rate-limiting techniques and their trade-offs.
Specifically, they are probing for your understanding of how each technique handles bursts (sudden increases in request rates) and ensures fairness among clients.

#### Assumptions

We assume a basic understanding of the concept of rate limiting and its importance in system design.
We will focus on comparing these techniques based on their burst handling capabilities and fairness properties.

#### First-Principles Solution
Fixed-Window Rate Limiting

Fixed-window rate limiting is one of the simplest forms. It divides time into fixed intervals (windows) and allows a certain number of requests per window. If a client exceeds this limit, they are temporarily blocked until the next window starts.

#### Pros

Easy to implement.
Fairness is straightforward; each request within the same window gets an equal opportunity to be processed.

#### Cons

Poor burst handling. A large number of requests can fill up the entire window quickly, leading to a sudden blockage for subsequent clients.
Does not account for varying client traffic patterns over time.
Sliding-Window Rate Limiting

Sliding-window rate limiting improves upon fixed-window by continuously tracking request rates without resetting at the end of each window. It maintains a sliding window that accumulates requests over time, providing better burst handling.

#### Pros

Better burst handling because it can dynamically adjust to bursts and then release them as they subside.
Fairness is generally improved compared to fixed-window since it more closely follows real-time request patterns.

#### Cons

More complex implementation due to the need for a sliding mechanism and maintaining state over time.
Potential for slightly higher latency, especially in high-load scenarios where window calculations are performed frequently.
Token-Bucket Rate Limiting

Token-bucket rate limiting is another advanced technique that uses a bucket filled with tokens. Tokens can be "refilled" at a steady rate or by bursts, and requests consume tokens from the bucket. If no tokens remain, the request is denied.

#### Pros

Excellent burst handling because it can absorb bursts of traffic without blocking other clients.
Fairness can be fine-tuned based on the token refill rate; slower refill rates favor more consistent throughput.

#### Cons

More complex implementation and state management compared to fixed and sliding windows.
Performance overhead, particularly in systems with very high request volumes or frequent token refills.

#### Closing Summary

In summary, while all three techniques aim to control request rates, they differ significantly in how they handle bursts and ensure fairness:

Fixed-Window is simple but suffers from poor burst handling and can block legitimate clients.
Sliding-Window offers better burst handling by dynamically adjusting over time but at the cost of increased complexity and potential latency.
Token-Bucket provides excellent burst handling and flexible fairness but requires more sophisticated implementation and state management.

Each technique has its use cases, and the choice depends on the specific requirements of your application. For example, if you need to handle unpredictable bursts without blocking other clients, token-bucket might be preferable, whereas for simpler, more controlled environments, fixed-window could suffice.

#### Common Mistakes/Follow-Ups
Mistake: Overlooking the importance of fairness in rate limiting.

#### Follow-Up
Discussing specific scenarios where one technique would outperform another, such as handling sudden spikes in traffic or ensuring consistent throughput for all clients.

**Comparison:**

```
Technique	Burst Handling	Fairness
Fixed-Window	Poor	Good
Sliding-Window	Better	Good
Token-Bucket	Excellent	Flexible
```

**Code:**

```python
# Example implementation of token-bucket rate limiting

import time
from collections import deque

class TokenBucket:
    def __init__(self, capacity, refill_rate):
        self.capacity = capacity  # Maximum tokens available
        self.refill_rate = refill_rate  # Rate at which tokens are refilled (tokens per second)
        self.tokens = min(capacity, capacity)  # Start with full bucket
        self.timestamp = time.time()  # Initial timestamp

    def consume(self, tokens):
        current_time = time.time()
        elapsed_time = current_time - self.timestamp
        self.tokens += elapsed_time * self.refill_rate
        if self.tokens > self.capacity:
            self.tokens = self.capacity
        if self.tokens < tokens:
            return False  # Not enough tokens to consume
        self.tokens -= tokens
        self.timestamp = current_time  # Update timestamp after successful consumption
        return True

# Example usage
bucket = TokenBucket(10, 5)
print(bucket.consume(3))  # Should print True
print(bucket.consume(7))  # Should print False (not enough tokens available)
```

**Example input / output:**

```text
Input:
- Create a TokenBucket with capacity=10 and refill_rate=5.
- Consume 3 tokens at time t=0.

Output:
- True

Explanation:
- At t=0, the bucket has 9.8 tokens (assuming it was filled just before this request).
- Consuming 3 tokens leaves 6.8 tokens in the bucket.
```

**Data:**

```json
{
  "capacity": 10,
  "refill_rate": 5,
  "tokens": 9.8,  // Example state after some time has passed and tokens have been refilled
  "timestamp": 1633072800.5  // Example timestamp (in seconds since epoch)
}
```

**Flow:**

1. Yes
2. No
3. TokenBucket Rate Limiting
4. Is there a sufficient number oftokens?
5. Grant request and refill tokens
6. Briefly block the request untilmore tokens are available

