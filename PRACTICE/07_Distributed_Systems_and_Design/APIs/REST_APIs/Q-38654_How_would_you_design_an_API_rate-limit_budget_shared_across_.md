---
type: "practice"
difficulty: "Deep"
---

# How would you design an API rate-limit budget shared across a token bucket for normal calls and a separate stricter bucket for expensive endpoints, and how do you communicate both limits to a client cleanly?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ API Rate Limiting](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/API_Rate_Limiting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question, we'll first outline the core principles of designing an API rate-limiting system that distinguishes between normal and expensive endpoints using separate token buckets. We will then discuss how to effectively communicate these limits to clients. The key idea is to ensure a fair distribution of resources while maintaining client transparency.

#### Triaging the Question

The question asks for the design of a rate-limiting mechanism, specifically focusing on managing two different types of API calls: normal and expensive endpoints. The client needs to be informed about both sets of limits to manage their usage effectively. This involves understanding the requirements, making assumptions, and designing the system step-by-step.

#### Assumptions
and Requirements
Types of Endpoints: We have normal (less resource-intensive) and expensive (more resource-intensive) endpoints.
Rate Limits: Separate token buckets will be used for each type to manage different levels of throughput.
Client Notification: The client should be aware of both sets of limits, helping them optimize their API usage.
Step-by-Step Design
1. Token Buckets

Token buckets are a classic mechanism for rate limiting. Each bucket has an initial capacity and replenishment rate. Tokens can be added to the bucket at a constant or variable rate (replenishment), and clients draw tokens when making requests.

For our scenario, we will use two token buckets:

Normal Bucket: Lower refill rate but higher initial capacity.
Expensive Bucket: Higher refill rate but lower initial capacity due to stricter limits.
2. Token Allocation

Normal Endpoint:

Refill Rate: 50 tokens/sec (lower than the expensive bucket)
Initial Capacity: 1000 tokens (more than enough for typical usage patterns)

Expensive Endpoint:

Refill Rate: 20 tokens/sec (much lower to ensure stricter control)
Initial Capacity: 100 tokens (to enforce limited use while still allowing some requests)
3. Implementation

We will implement a rate-limiting middleware that enforces these rules. Here's a simplified version in pseudo-code:

PSEUDO
class RateLimiter:
    def __init__(self):
        self.normal_bucket = TokenBucket(refill_rate=50, capacity=1000)
        self.expensive_bucket = TokenBucket(refill_rate=20, capacity=100)

    def is_allowed(self, endpoint_type: str) -> bool:
        if endpoint_type == "normal":
            return self.normal_bucket.deduct_tokens()
        elif endpoint_type == "expensive":
            return self.expensive_bucket.deduct_tokens()

class TokenBucket:
    def __init__(self, refill_rate, capacity):
        self.refill_rate = refill_rate
        self.capacity = capacity
        self.tokens = capacity  # Initial tokens

    def deduct_tokens(self) -> bool:
        if self.tokens > 0:
            self.tokens -= 1
            return True
        else:
            return False

    def replenish(self):
        self.tokens += self.refill_rate * (time.time() - last_replenish_time)
4. Client Communication

To inform clients about the limits, we will return a structured response with both sets of rate limits:

JSON
{
    "normal_limit": {
        "refill_rate": 50,
        "capacity": 1000,
        "remaining": 973  # Assume some tokens have been used
    },
    "expensive_limit": {
        "refill_rate": 20,
        "capacity": 100,
        "remaining": 84  # Assume some tokens have been used
    }
}

#### Closing Summary

In summary, we designed a rate-limiting system with separate token buckets for normal and expensive endpoints. The normal bucket has a higher initial capacity but lower refill rate, while the expensive bucket has a stricter limit with lower initial tokens but a slower refill rate. Clients will receive detailed information about both sets of limits to manage their API usage effectively.

#### Common Mistakes
and Follow-Ups
Ignoring Edge Cases: Ensure handling scenarios like sudden bursts of requests or temporary overuse.
Insufficient Detail in Response: Include all relevant details such as current token count, remaining time until refill, etc.
Not Considering Caching: Implement caching mechanisms to avoid repeated checks for the same client.

Would you like me to elaborate on any specific part of this design?

**Comparison:**

```
2019-06-28
Refill Rate
Capacity
Remaining Tokens
Refill Rate
Capacity
Remaining Tokens
Normal Endpoint
Expensive Endpoint
Rate Limiting Parameters
```

**Code:**

```python
class TokenBucket:
    def __init__(self, refill_rate, capacity):
        self.refill_rate = refill_rate
        self.capacity = capacity
        self.tokens = capacity  # Initial tokens

    def deduct_tokens(self) -> bool:
        if self.tokens > 0:
            self.tokens -= 1
            return True
        else:
            return False

    def replenish(self, time_since_last_replenish):
        current_time = time.time()
        elapsed_time = current_time - time_since_last_replenish
        tokens_added = int(elapsed_time * self.refill_rate)
        self.tokens += min(tokens_added, self.capacity)  # Ensure we don't exceed capacity

class RateLimiter:
    def __init__(self):
        self.normal_bucket = TokenBucket(refill_rate=50, capacity=1000)
        self.expensive_bucket = TokenBucket(refill_rate=20, capacity=100)

    def is_allowed(self, endpoint_type: str) -> bool:
        if endpoint_type == "normal":
            return self.normal_bucket.deduct_tokens()
        elif endpoint_type == "expensive":
            return self.expensive_bucket.deduct_tokens()

# Example usage
limiter = RateLimiter()
print(limiter.is_allowed("normal"))  # Deduct tokens from normal bucket
print(limiter.is_allowed("expensive"))  # Deduct tokens from expensive bucket
```

**Example input / output:**

```text
Input:
- Endpoint type: normal

Output:
{
    "normal_limit": {
        "refill_rate": 50,
        "capacity": 1000,
        "remaining": 973  # Assume some tokens have been used
    }
}

Input:
- Endpoint type: expensive

Output:
{
    "expensive_limit": {
        "refill_rate": 20,
        "capacity": 100,
        "remaining": 84  # Assume some tokens have been used
    }
}
```

**Data:**

```json
{
    "normal_limit": {
        "refill_rate": 50,
        "capacity": 1000,
        "remaining": 973  # Assume some tokens have been used
    },
    "expensive_limit": {
        "refill_rate": 20,
        "capacity": 100,
        "remaining": 84  # Assume some tokens have been used
    }
}
```

**Flow:**

1. Yes
2. No
3. Client Request
4. Is Endpoint Normal?
5. Ratelimiter.is_allowed('normal')
6. Update Token Bucket andReturn Response
7. Ratelimiter.is_allowed('expensive')
8. Update Token Bucket andReturn Response
9. Response: Normal Limit Info
10. Response: Expensive Limit Info

