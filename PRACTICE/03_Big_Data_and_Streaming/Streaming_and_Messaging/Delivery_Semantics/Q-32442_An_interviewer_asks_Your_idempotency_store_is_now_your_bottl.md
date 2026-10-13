---
type: "practice"
difficulty: "Expert"
---

# An interviewer asks: 'Your idempotency store is now your bottleneck at peak.' Redesign for horizontal scale: sharding the dedup keyspace, avoiding hot keys, and keeping cross-shard correctness under rebalancing.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Idempotency and Schema Evolution](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Idempotency_and_Schema_Evolution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of idempotency storage becoming a bottleneck at peak times, we need to redesign our system to handle horizontal scaling more effectively while maintaining the idempotency guarantees. Specifically, we will focus on sharding the deduplication keyspace, ensuring that hot keys are evenly distributed across shards, and implementing mechanisms to maintain cross-shard correctness during rebalancing.

The core problem is that certain high-frequency operations can lead to a single shard becoming overloaded due to excessive read/write requests for idempotency checks. To resolve this, we'll introduce sharding into the deduplication process while ensuring that the system remains resilient and maintains its guarantees under dynamic scaling conditions.

#### Triaging the Question

The interviewer is probing our understanding of distributed systems, specifically around key-value store semantics, rebalancing strategies, and fault tolerance mechanisms in a highly available setup. We need to assume a high-traffic service with frequent operations that require idempotency checks (e.g., payment processing or transaction confirmation). The goal is to ensure that no matter how many shards we add or remove, the system remains performant and guarantees that each operation is processed exactly once.

#### First Principles Solution
Sharding Strategy: We'll shard our idempotency keyspace based on a hash of the key. This ensures an even distribution of load across all shards.
Hot Key Avoidance: Implement mechanisms to detect and mitigate hot keys, ensuring that no single shard becomes overloaded.
Cross-Shard Correctness: Ensure that rebalancing operations do not compromise the idempotency guarantees by implementing consistent cross-shard reconciliation and update strategies.
Detailed Implementation

Sharding Key Generation:

PYTHON
def get_shard_key(key):
    # Use a hash function to distribute keys across shards
    return hash(key) % number_of_shards

Hot Key Detection & Mitigation:

Monitor the request patterns and detect any shard that is handling more than its fair share of requests.
Implement automatic rebalancing or load-shedding mechanisms to redistribute hot keys.

Cross-Shard Consistency:

When a shard needs to be added or removed, implement a cross-shard reconciliation mechanism.
Use consensus protocols (e.g., Raft) for distributed transactions to ensure that all shards are in agreement on the state of idempotency checks.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By sharding the deduplication keyspace and ensuring hot key avoidance, we can significantly reduce the risk of a single shard becoming a bottleneck. Additionally, by maintaining cross-shard consistency through distributed transactions or other consensus mechanisms, we ensure that idempotency guarantees are upheld even as the system scales.

#### Common Mistakes/Follow-Ups

Failing to properly hash keys can lead to uneven distribution and hotspots.
Not having effective hot key detection and mitigation strategies can result in performance degradation under peak loads.
Inadequate cross-shard consistency mechanisms may introduce race conditions or data inconsistencies, compromising idempotency guarantees.
Summary

Sharding the deduplication keyspace, detecting and mitigating hot keys, and ensuring cross-shard correctness are crucial steps to redesigning a scalable idempotency store. This solution ensures that our system remains performant and reliable even as it scales horizontally.

**Comparison:**

```
Shard ID	Request Count
0	5000
1	4000
2	6000
3	3000
4	7000
```

**Code:**

```python
def get_shard_key(key):
    return hash(key) % number_of_shards

# Example of hot key detection
class HotKeyDetector:
    def __init__(self, threshold=0.75):
        self.threshold = threshold
        self.request_counts = {}

    def update_request_count(self, shard_id):
        if shard_id not in self.request_counts:
            self.request_counts[shard_id] = 1
        else:
            self.request_counts[shard_id] += 1

    def is_hot_key(self, shard_id):
        return self.request_counts[shard_id] / sum(self.request_counts.values()) > self.threshold
```

**Example input / output:**

```text
Input:
- Shard key distribution before rebalancing
- High request counts for shard 4

Output:
- Shard 4 requests are redirected to other shards, ensuring even distribution.
- New shards may be added and existing ones adjusted based on current hot key patterns.
```

**Data:**

```json
{
  "keys": [
    "user-001",
    "transaction-123456789",
    "payment-xyz"
  ],
  "shards": 5,
  "request_counts": {
    0: 5000,
    1: 4000,
    2: 6000,
    3: 3000,
    4: 7000
  }
}
```

**Flow:**

1. Yes
2. No
3. Initial Key Distribution
4. Detect Hot Keys
5. Rebalance
6. Distribute Requests Evenly
7. Monitor and Adjust
8. Maintain Cross-ShardConsistency

