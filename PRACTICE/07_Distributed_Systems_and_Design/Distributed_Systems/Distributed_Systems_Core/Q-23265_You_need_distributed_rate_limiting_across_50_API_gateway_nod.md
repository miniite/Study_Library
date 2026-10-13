---
type: "practice"
difficulty: "Deep"
---

# You need distributed rate limiting across 50 API gateway nodes with a global limit. Compare token bucket in a shared Redis vs local buckets with periodic reconciliation. What consistency do you sacrifice?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Rate Limiting](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Rate_Limiting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

In this scenario, we are comparing two approaches to implement distributed rate limiting across 50 API gateway nodes—using a shared token bucket in Redis versus having local buckets with periodic reconciliation. The trade-offs involve consistency and the complexity of maintaining state across multiple nodes.

#### Triage

The core of the question revolves around understanding the implications of consistency, availability, and the cost of communication between nodes when implementing rate limiting in distributed systems. We need to consider the specific use case of having a global limit while ensuring that each node can independently enforce its local limits.

Detailed Solution
Shared Token Bucket with Redis

#### Pros

Global Consistency: All nodes have access to the same token bucket, which enforces a single, unified rate limit across all API gateways.
Simplicity in Configuration: Only one configuration is needed for the rate limit, simplifying management.

#### Cons

Latency and Performance: Since tokens are fetched from Redis, there can be significant latency if nodes need to frequently access this resource.
High Load on Redis: High traffic volumes could overwhelm Redis, leading to performance issues or even service disruptions.
Failure Modes: If the Redis node fails, all API gateways will be unable to enforce rate limits until the Redis instance is back online.
Local Buckets with Periodic Reconciliation

#### Pros

Reduced Latency: Each node maintains its own token bucket locally, reducing the latency of fetching tokens.
Decoupling from Centralized Systems: Nodes are less dependent on external services like Redis, making them more resilient to outages.
Scalability: Easier to scale as the number of nodes increases since each node only needs to communicate with a local Redis instance.

#### Cons

Partial Consistency: The global rate limit might not always be perfectly enforced due to potential misalignment between the distributed buckets during reconciliation periods.
Complexity in Reconciliation Logic: Ensuring that all nodes have consistent token counts after updates can be complex and error-prone.
Higher Operational Overhead: Periodic reconciliation requires additional monitoring and management, increasing operational complexity.

#### Cons
istency Considerations
Shared Redis (Linearizability): By using shared state in Redis, we achieve a linearizable consistency model, where each read or write operation appears to be completed instantaneously.
Local Buckets with Reconciliation (Eventual Consistency): This approach uses a more relaxed consistency model, where updates are eventually consistent but may temporarily diverge.

#### Conclusion

Choosing between these approaches depends on the specific requirements of your system. For highly available and low-latency applications, shared Redis might be preferable due to its simplicity and global consistency guarantees. However, for scenarios with high node counts or a need for reduced latency and resilience, local buckets with periodic reconciliation can offer better performance and fault tolerance.

#### Common Mistakes
and Follow-Ups
Overlooking Latency Issues: For critical applications requiring low-latency rate limiting, failure to account for Redis fetch latency might lead to suboptimal performance.
Neglecting Operational Complexity: Failing to recognize the added complexity of periodic reconciliation can result in higher operational overhead without clear benefits.
Ignoring Failure Scenarios: Not adequately planning for Redis outages or misconfigurations could leave your system vulnerable.

By carefully considering these factors, you can make an informed decision that best meets the needs of your distributed system architecture.

**Comparison:**

```
Approach	Global Consistency	Latency	Complexity
Shared Redis	Yes	High	Low
Local Buckets	No	Low	High
```

**Code:**

```python
# Example code for a simple local token bucket with periodic reconciliation
import time

class TokenBucket:
    def __init__(self, capacity, refill_rate):
        self.capacity = capacity
        self.tokens = 0
        self.refill_rate = refill_rate
        self.last_refill_time = time.time()

    def consume(self, tokens_needed):
        current_time = time.time()
        elapsed_time = current_time - self.last_refill_time
        self.tokens += min(elapsed_time * self.refill_rate, self.capacity)
        self.tokens -= min(tokens_needed, self.tokens)
        if self.tokens < 0:
            self.tokens = 0
        self.last_refill_time = current_time

# Example usage
bucket = TokenBucket(capacity=100, refill_rate=50)
print(bucket.consume(20))  # Tokens remaining: 80
time.sleep(1)  # Simulate time passing
print(bucket.consume(30))  # Tokens remaining: 40
```

**Example input / output:**

```text
Input:
- Node 1 consumes 30 tokens.
- Node 2 consumes 40 tokens.

Output:
Node 1: Tokens remaining: 70
Node 2: Tokens remaining: 60

After reconciliation (1 second later):
Node 1: Tokens remaining: 50
Node 2: Tokens remaining: 40
```

**Data:**

```json
{
  "nodes": [
    {"id": 1, "token_bucket_capacity": 100, "refill_rate": 50},
    {"id": 2, "token_bucket_capacity": 100, "refill_rate": 50}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Time Passage
4. Yes
5. Redis
6. Fetch Tokens from Redis
7. Enforce Rate Limits Locally
8. Node Consumes Tokens
9. Is Reconciliation Needed?
10. Reconcile Token Counts
11. Local Rate Limit Enforcement
12. Update Local Buckets
13. Is Refill Time?
14. Refill Token Bucket in Redis

