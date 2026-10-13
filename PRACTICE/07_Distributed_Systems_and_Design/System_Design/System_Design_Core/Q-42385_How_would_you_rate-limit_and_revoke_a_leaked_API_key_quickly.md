---
type: "practice"
difficulty: "Deep"
---

# How would you rate-limit and revoke a leaked API key quickly?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Rate Limiter Storage](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Rate_Limiter_Storage/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the problem of rating-limiting and quickly revoking a leaked API key, we need to design a robust system that can handle both high throughput and real-time decision-making under pressure. At a backend architect/principal level, the focus will be on ensuring the solution is scalable, fault-tolerant, and minimizes latency while maintaining security.

The question is essentially asking how to manage two critical aspects: rate limiting (controlling API usage) and key revocation (revoking access when an API key might have been leaked). We need to consider a few assumptions:

The system must handle a large number of concurrent requests.
Revocation should be immediate, with minimal delay.
Rate limits should be enforced efficiently without significantly impacting performance.
Step-by-Step Solution
1. Rate Limiting

To implement rate limiting, we can use a token bucket algorithm or leaky bucket algorithm. Both algorithms are well-suited for handling varying request rates and ensuring that too many requests are not processed simultaneously.

Token Bucket Algorithm:

Each API key is associated with a bucket.
Tokens are added to the bucket at a fixed rate (R).
When a request comes in, one token is consumed from the bucket.
If no tokens are available, the request is dropped or throttled.
API Request
Check Bucket

#### Cons
ume Token
Process Request
Reject Request

Leaky Bucket Algorithm:

Tokens are added to the bucket at a fixed rate (R).
Excess tokens "leak" out of the bucket at a slower rate (L).
When a request comes in, one token is consumed from the bucket.
If no tokens are available, the request is dropped or throttled.
2. Revocation

Revoking an API key quickly involves removing its access and ensuring that any ongoing requests are terminated.

Immediate Revocation:

Use a central service to manage active API keys.
Implement a mechanism (e.g., a Redis set) to store the active API keys and their associated tokens or buckets.
When revoking, remove the key from the set immediately.
API Key Revoked
Remove Key
Update Token Bucket
Notify Clients
3. Scalability and Fault Tolerance

To ensure scalability:

Use distributed systems (e.g., Kubernetes) to manage the rate-limited services.
Implement a load balancer to distribute requests evenly across multiple instances.

For fault tolerance:

Use redundant storage (e.g., Redis clusters or distributed databases).
Implement retries and fallback mechanisms for client-side errors.

#### Closing Summary

In summary, rating-limiting and revoking a leaked API key involves implementing rate limiting algorithms like token bucket or leaky bucket to control request rates and using centralized services to manage active keys and ensure immediate revocation. The solution should be scalable, fault-tolerant, and capable of handling high throughput while maintaining minimal latency.

#### Common Mistakes
Ignoring Edge Cases: Failing to handle situations where the rate limit is reached too quickly or when a key needs to be revoked in an emergency.
Synchronization Issues: Ensuring that token buckets are updated consistently across all instances can be challenging, leading to race conditions or inconsistencies.
Performance Impact: Implementing the solution without considering the performance overhead of maintaining state and enforcing rates.

**Comparison:**

| Request | Token Bucket State |
|---------|--------------------|
| 1       | Tokens: 20, Last Time: 1634078590.012345 |
| 2       | Tokens: 19, Last Time: 1634078590.012345 |
| ...     | ...                |
| 30      | Tokens: 5, Last Time: 1634078595.012345 |

**Example input / output:**

```text
# Example Usage of RateLimiter

limiter = RateLimiter(rate=5, capacity=20)
for _ in range(30):
    if limiter.consume_token():
        print("Token consumed")
    else:
        print("Rate limit exceeded")
```

**Flow:**

1. API Request
2. Check Bucket
3. Bucket Full?|> DropRequest|Consume Token
4. Process Request|> ConsumeToken
5. Reject Request

