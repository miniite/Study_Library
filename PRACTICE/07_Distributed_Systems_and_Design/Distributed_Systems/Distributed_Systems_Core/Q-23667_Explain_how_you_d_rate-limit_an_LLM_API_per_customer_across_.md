---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd rate-limit an LLM API per customer across a fleet of gateway nodes with a hard global token-per-minute quota. Compare a shared Redis token bucket versus local buckets with reconciliation, and what consistency you sacrifice.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Rate Limiting](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Rate_Limiting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To rate-limit an LLM (Large Language Model) API per customer across a fleet of gateway nodes while adhering to a hard global token-per-minute quota, we need to ensure that the total number of requests from all customers does not exceed this limit. The approach involves choosing between using a shared Redis token bucket or local buckets with periodic reconciliation, and understanding the trade-offs in terms of consistency.

#### Triaging the Question

The key aspects being probed are:

Rate Limiting Strategy: How to manage API calls without exceeding a global quota.

#### Cons
istency Models: What level of data consistency is acceptable between shared and local token buckets.

#### Implementation
Details: The technical nuances involved in setting up and maintaining these strategies.

#### Assumptions
Each gateway node receives requests from multiple customers.
The global quota is enforced at the fleet level, not on individual nodes.
Redis is used as a distributed cache to store tokens.

#### First-Principles Solution
Shared Redis Token Bucket Approach

In this approach, all nodes share a single token bucket stored in a Redis instance. Here’s how it works:

Token Refills: The Redis instance refills the bucket at regular intervals based on the global quota.
Token Checks: Each node checks if there are available tokens before processing a request.
Decrement Tokens: If a token is available, it is decremented from the bucket.
Reconciliation: Periodically, all nodes reconcile their state with the Redis instance to ensure consistency.

#### Pros

Single point of truth for token management.
Simplifies the implementation as there’s no need for complex reconciliation logic among multiple nodes.
Easier to debug and monitor as everything is centralized.

#### Cons

High latency due to network requests to the Redis server, especially under load.
Potential bottleneck if the Redis server becomes a single point of failure.
Reduced scalability as the number of nodes increases (each node needs to communicate with the Redis instance).
Local Buckets with Reconciliation

In this approach, each gateway node maintains its own token bucket. Periodically, these buckets are reconciled to ensure they match the global quota.

Token Refills: Each node refills its local token bucket independently.
Token Checks and Decrement: Nodes check and decrement tokens locally before processing a request.
Reconciliation: A periodic process ensures that all nodes’ token counts match the global quota, typically by sending updates to a central coordination service.

#### Pros

Better performance as requests don’t need to go through Redis.
More scalable since there are fewer network dependencies.

#### Cons

More complex implementation due to the need for consistent reconciliation logic across multiple nodes.
Risk of inconsistent states if reconciliation fails or is delayed.

#### Closing Summary

Both approaches have their strengths and weaknesses. The shared Redis token bucket simplifies management but can introduce bottlenecks, while local buckets with reconciliation offer better performance but require more sophisticated consistency handling.

#### Common Mistakes/Follow-Ups
Underestimating Latency: Failing to account for the network latency between nodes and the Redis server can lead to inconsistent rate limiting.
Ignoring Scalability: Not considering how the solution scales with increasing numbers of gateway nodes or higher global quotas.
Lack of Error Handling: Failing to handle reconciliation failures, which could lead to violations of the global quota.

**Comparison:**

| Node | Initial Local Bucket Size | Updated Local Bucket Size |
|------|--------------------------|----------------------------|
| 1    | 5                        | 40                         |
| 2    | 3                        | 30                         |

**Code:**

```python
# Example code snippet for shared Redis token bucket (simplified)
import redis

def get_token_bucket(redis_client):
    return redis_client.zrange('token-bucket', 0, -1)

def refill_tokens(token_bucket, tokens_to_add):
    # Refill the token bucket with new tokens
    pass

def check_and_decrement(tokens_required, token_bucket):
    if len(token_bucket) >= tokens_required:
        token_bucket.pop()
        return True
    return False

# Example code snippet for local buckets with reconciliation (simplified)
import redis

class GatewayNode:
    def __init__(self, node_id):
        self.node_id = node_id
        self.local_bucket = []  # Local token bucket

    def refill_local_token_bucket(self, tokens_to_add):
        # Refill the local token bucket
        pass

    def check_and_decrement_tokens(self, tokens_required):
        if len(self.local_bucket) >= tokens_required:
            self.local_bucket.pop()
            return True
        return False

def reconcile_token_buckets(nodes, redis_client):
    for node in nodes:
        node.local_bucket = get_token_bucket(redis_client)
```

**Example input / output:**

```text
Input:
- Global quota: 100 tokens per minute
- Number of nodes: 5

Output (Shared Redis):
- Token bucket in Redis is refilled to 100 tokens at the start.
- Nodes check and decrement tokens from their shared token bucket.

Reconciliation process:
- Periodic reconciliation ensures all nodes have updated token counts.

Local Buckets with Reconciliation:
- Each node maintains its own local bucket but periodically updates it based on the global quota.
- Reconciliation ensures consistency across nodes.

Dry-run example for Local Buckets with Reconciliation:
Node 1: Initial local bucket [5, 4]
Node 2: Initial local bucket [3, 2]

Reconciliation step:
Node 1: Updates its local bucket to match the new global state.
Node 2: Updates its local bucket to match the new global state.

Final state after reconciliation:
All nodes have updated their local buckets to reflect the global quota.
```

**Data:**

```json
{
  "global_quota": 100,
  "nodes": [
    {"node_id": 1, "local_bucket_size": 5},
    {"node_id": 2, "local_bucket_size": 3}
  ]
}
```

**Flow:**

1. No
2. Yes
3. Refill Token Bucket
4. Is Global Quota Exceeded?
5. Check and Decrement Tokens
6. Return Error or Wait
7. Local Buckets withReconciliation
8. Node Updates Local Bucket
9. Periodic Reconciliation Step
10. All Nodes Update to GlobalQuota

