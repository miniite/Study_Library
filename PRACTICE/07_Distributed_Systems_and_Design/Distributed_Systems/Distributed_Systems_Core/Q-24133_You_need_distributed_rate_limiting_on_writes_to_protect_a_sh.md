---
type: "practice"
difficulty: "Deep"
---

# You need distributed rate limiting on writes to protect a shared database across 50 API nodes with a hard global quota. Compare a shared-Redis token bucket versus local buckets with periodic reconciliation, and what consistency you sacrifice.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Distributed Rate Limiting](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Rate_Limiting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the distributed rate limiting requirement in a multi-node setup while adhering to a strict global quota, we need to compare two strategies: shared-Redis token bucket versus local buckets with periodic reconciliation. The choice between these strategies involves evaluating trade-offs related to consistency and system complexity.

#### Triaging the Question

What the Question is Asking: The question requires comparing two approaches for implementing distributed rate limiting in a scenario where 50 API nodes need to protect a shared database with a hard global quota. The goal is to determine which approach ensures adherence to the quota while considering the consistency trade-offs.

Key Assumptions:

Each node needs to perform writes to the database.
There is a strict, non-negotiable global rate limit that all nodes must respect.
The solution should be robust and scalable across multiple API nodes.

#### First-Principles Solution
Shared-Redis Token Bucket Approach

In this approach, a shared Redis instance acts as a central authority to manage the token bucket. Each write operation by an API node involves checking with the Redis server for available tokens before proceeding with the write operation. This ensures that no more than the global quota is exceeded.

Steps:

Token Acquisition: Before performing a write, each API node checks its local state (token count) and requests additional tokens from Redis.
Token Replenishment: Tokens are replenished in Redis at regular intervals to ensure they can be consumed by nodes.
Rate Limit Enforcement: If the token count is insufficient for a write operation, the request fails with an appropriate error.

#### Pros

Global Consistency: Ensures strict adherence to the global quota due to centralized control.
Simplicity of Implementation: The logic is straightforward and can be easily managed by a dedicated Redis instance.

#### Cons

Latency Issues: High latency can occur if Redis operations are slow or network connectivity issues arise.
Single Point of Failure (SPOF): If the Redis server goes down, no nodes can perform writes until it is back online.
Local Buckets with Periodic Reconciliation Approach

In this approach, each API node maintains its own local token bucket and periodically syncs its state with a central Redis instance. This allows for more localized decision-making while still ensuring eventual global consistency.

Steps:

Local Token Management: Each node manages tokens locally based on its current quota.
Periodic Reconciliation: Nodes periodically send their token counts to a central Redis server, which then adjusts the local state of each node to reflect the current global quota.
Write Operations: Local decisions are made for write operations based on the current local state.

#### Pros

Reduced Latency: Lower latency compared to shared Redis as nodes make decisions locally.
Improved Resilience: The system is less prone to failure if the central Redis server fails, as nodes can continue operating with their local state.

#### Cons

Complexity of Implementation: Requires more complex logic and synchronization mechanisms to maintain eventual consistency.
Potential for Overlap: There may be a brief period where writes are allowed based on stale data before the reconciliation step updates the local state.

#### Closing Summary

The shared-Redis token bucket approach ensures strict adherence to the global quota but comes with potential latency issues and single points of failure. On the other hand, the local buckets with periodic reconciliation approach provides lower latency and improved resilience at the cost of more complex implementation and potential temporary inconsistencies.

#### Common Mistakes/Follow-Ups

Assuming Redundancy: Failing to consider redundancy in shared-Redis systems can lead to unexpected downtime.
Ignoring Network Latency: Overlooking network latency when evaluating local vs. central decision-making.
Underestimating Consistency Requirements: Not accounting for the level of consistency required, which can impact the choice of approach.

**Comparison:**

```
Approach	Pros	Cons
Shared-Redis Token Bucket	Global consistency, simple implementation	High latency, SPOF if Redis fails
Local Buckets with Periodic Reconciliation	Reduced latency, improved resilience	More complex, potential temporary inconsistencies
```

**Code:**

```python
# Example Python code snippet for shared Redis token bucket (simplified)
import redis

def get_token_bucket(redis_client):
    # Logic to manage token acquisition and replenishment
    pass

def write_to_database(node_id, data):
    redis_client = redis.Redis(host='localhost', port=6379, db=0)
    if not get_token_bucket(redis_client):
        return "Rate limit exceeded"
    else:
        # Perform database write operation
        pass
```

**Example input / output:**

```text
Input:
- Node 1 requests token bucket state.
- Node 2 writes to database.

Output:
Node 1 receives updated token count from Redis.
Node 2's write is accepted or rejected based on current local token count.
```

**Data:**

```json
{
  "nodes": [
    {"id": 1, "current_tokens": 50},
    {"id": 2, "current_tokens": 75}
  ],
  "global_quota": 300,
  "reconciliation_interval": 60
}
```

**Flow:**

1. Yes
2. No
3. Node Request
4. Is Token Available?
5. Write to Database
6. Rate Limit Error
7. Token Replenished
8. Retry or Fail
9. Periodic Reconciliation
10. Update Local State

