---
type: "practice"
difficulty: "Deep"
---

# How would you size and shard a cache for a workload where 5% of keys account for 95% of reads, so the hot keys never contend and the cold tail does not waste memory?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Caching Expensive Queries](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Caching_Expensive_Queries/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a caching system to handle a workload with extremely uneven key access patterns (where only 5% of keys account for 95% of reads), we need to ensure that the hot keys are served without contention and that the cold tail does not waste too much memory. This involves careful sizing, sharding, and possibly using different cache layers or eviction policies.

The primary challenges in this scenario are:

Hot Key Contention: Ensuring that frequently accessed keys do not cause excessive contention.
Memory Efficiency: Avoiding unnecessary storage of less frequently accessed data, which can waste memory resources.

To address these issues effectively:

Triaging the Problem
Question Breakdown: The question is asking about sizing and sharding a cache for a workload with an 80/20 distribution of access patterns. Specifically, it requires ensuring that high-frequency (hot) keys are served efficiently without contention, while low-frequency (cold) keys do not consume unnecessary space.

#### Assumptions
We have a distributed cache system where multiple nodes can store parts of the cache.
The cache uses an eviction policy to manage memory usage.

#### First Principles Solution

Sharding Strategy:

Use consistent hashing or range-based sharding to distribute hot and cold keys across cache nodes.

#### Example
If we have N cache nodes, each node will handle a subset of the total key space. For instance, Node 1 could handle keys from A-M, Node 2 could handle N-Z, etc.

Hot Key Handling:

Implement a local or distributed mutex (e.g., distributed locking mechanism) for hot keys to avoid contention.
Alternatively, use read-through/write-through caching for hot keys and in-memory replication across nodes.

Cold Key Management:

Use an LRU (Least Recently Used) eviction policy for cold keys on each node.

#### Cons
ider using tiered caching where frequently accessed cold keys are promoted to a higher-tier cache with more resources, while less frequently accessed ones are dropped to lower tiers or eventually evicted.

Memory Sizing:

Estimate the total memory required based on the expected hit rate of hot keys and the distribution of data.
Allocate a larger portion of the available memory to nodes serving hot keys, ensuring they can handle their high load without thrashing.

#### Closing Summary

By implementing sharding to separate hot from cold keys, using appropriate eviction policies, and strategically sizing cache resources, we can effectively manage an uneven access pattern. Sharding ensures that hot keys are served efficiently on dedicated nodes, while cold keys do not waste memory. The key is to balance the distribution of load and memory usage across nodes.

#### Common Mistakes/Follow-Ups
Overlooking Contention Handling: Forgetting to implement a mechanism to handle contention among high-frequency keys.
Insufficient Memory Management: Not allocating enough resources to handle hot keys, leading to degraded performance.
Ignoring Eviction Policies for Cold Keys: Failing to optimize memory usage by aggressively evicting less frequently accessed data.

#### Example

#### Cons
ider a scenario with 10 cache nodes and an estimated total of 5 million unique keys:

Each node could handle approximately 500,000 keys.
Hot keys (e.g., top 5%) could be assigned to specific nodes to ensure they are served efficiently.
Cold keys (bottom 95%) would follow a more aggressive eviction policy.

**Comparison:**

| Node | Hot Keys       | Cold Keys |
|------|---------------|----------|
| 0    | [123456, 789012] | []        |
| 1    | [234567, 890123] | []        |
| ...  | ...           | ...      |
| 9    | [987654, 432109] | []        |

**Code:**

```python
def shard_key(key):
    # Example consistent hashing function
    return hash(key) % NUM_NODES

class CacheNode:
    def __init__(self, node_id):
        self.node_id = node_id
        self.cache = {}
        self.locks = {}

    def get(self, key):
        node_id = shard_key(key)
        if key in self.cache[node_id]:
            return self.cache[node_id][key]
        else:
            value = fetch_from_storage(node_id)  # Simulate fetching from storage
            self.cache[node_id][key] = value
            return value

    def set(self, key, value):
        node_id = shard_key(key)
        with self.locks[node_id]:
            self.cache[node_id][key] = value
```

**Example input / output:**

```text
Input: Get key 123456
Output: Value for key 123456

Input: Set key 123457 value "new_value"
Output: Value for key 123457 updated to "new_value"
```

**Data:**

```json
{
  "nodes": [
    {"id": 0, "hot_keys": [123456, 789012], "cold_keys": []},
    {"id": 1, "hot_keys": [234567, 890123], "cold_keys": []},
    ...
    {"id": 9, "hot_keys": [987654, 432109], "cold_keys": []}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Key Request
4. Is Hot Key?
5. Retrieve from Cache
6. Fetch from Storage, UpdateCache
7. Return Value
8. Store in Cache, Return Value

