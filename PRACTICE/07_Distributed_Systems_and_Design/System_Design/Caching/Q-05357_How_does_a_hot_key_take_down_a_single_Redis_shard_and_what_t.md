---
type: "practice"
difficulty: "Expert"
---

# How does a hot key take down a single Redis shard, and what techniques (key splitting, local cache) mitigate it?

**Difficulty:** Expert · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Redis Fundamentals](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Redis_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with high traffic, a "hot key" can overwhelm a single Redis shard, causing performance issues or even crashes. As an expert in caching, you'll need to understand why this happens and how to mitigate it using techniques like key splitting and local caching.

To break down the problem:

What is a Hot Key?: A hot key refers to a database query that receives significantly more requests than other keys.
Why Does It Impact Redis Shards?: Redis shards can be distributed among multiple nodes, but in practice, each node may handle some hot keys. When one shard gets disproportionately high traffic, it can lead to performance degradation or crashes due to resource exhaustion (CPU, memory, I/O).

The core issue here is balancing the load across shards while ensuring that no single node becomes a bottleneck.

Mitigation Techniques
1. Key Splitting

Key splitting involves distributing keys across multiple Redis nodes in such a way that hot keys are spread out evenly. For example, you can use consistent hashing or range-based partitioning:

#### Cons
istent Hashing: Assign each key to a specific node based on its hash value. This ensures a more even distribution of data and requests.
Hash Key
Node 1
Node 2
Node 3
Range-Based Partitioning: Split keys into ranges (e.g., by key prefix or suffix) and assign each range to a different node. This method is simpler but might not be as efficient for non-uniform key distributions.
2. Local Cache

Implementing a local cache (such as Memcached or another in-memory store) can significantly reduce the load on Redis by storing frequently accessed data locally. When a request comes in, first check the local cache; if it's not there, fetch from Redis and then cache it for future use.

Local Cache
Check Cache
Return from Cache
Fetch from Redis
Cache in Local Cache
Return from Redis
Step-by-Step Solution
Assess the Key Distribution: Analyze which keys are being accessed most frequently.
Implement Consistent Hashing or Range-Based Partitioning:
For consistent hashing, determine the number of shards and hash each key to assign it a specific node.
For range-based partitioning, define key ranges and map them to nodes.
Set Up Local Caches: Configure local caches at the application layer to store frequently accessed data.
Monitor Performance: Continuously monitor Redis shard performance and adjust cache settings as needed.

#### Closing Summary

Mitigating hot keys in a Redis environment requires both strategic key distribution techniques like consistent hashing or range-based partitioning, and local caching strategies to reduce load on individual nodes. By implementing these solutions, you can ensure more balanced and efficient data access.

#### Common Mistakes/Follow-Ups

Not Considering Data Freshness: Ensuring that the cache is invalidated correctly when data changes.
Over-reliance on a Single Cache Node: Distributing the local cache load effectively to avoid creating new bottlenecks.
Ignoring Write Operations: While focus is often on reads, managing write operations in distributed caches can be complex.

**Comparison:**

```
Node	Keys
Node 1	key1, key2
Node 2	key3, key4
Node 3	key5, key6
```

**Code:**

```python
# Example of key splitting with consistent hashing using Python
import hashlib

def get_node_for_key(key):
    hash_val = int(hashlib.md5(key.encode()).hexdigest(), 16)
    num_nodes = 10  # Number of shards
    return hash_val % num_nodes

# Example of local caching
from functools import lru_cache

@lru_cache(maxsize=1024)  # Local cache with a max size limit
def get_data_from_redis(key):
    # Simulate Redis fetch
    data = fetch_from_redis(key)
    return data
```

**Example input / output:**

```
Input: GET /api/data?key=key1
Output:
JSON
Copy
{
  "status": "success",
  "data": "Sample data for key1"
}
Local Cache Hit Example:
PYTHON
Copy
@lru_cache(maxsize=1024)
def get_data_from_redis(key):
    if cache_hit := local_cache.get(key):
        return cache_hit
    else:
        # Fetch from Redis and update cache
        data = fetch_from_redis(key)
        local_cache[key] = data
        return data
```

**Data:**

```json
{
  "nodes": ["Node 1", "Node 2", "Node 3"],
  "keys": ["key1", "key2", "key3", "key4"]
}
```

**Flow:**

1. Yes
2. No
3. Key
4. Is Key Cached?
5. Return from Cache
6. Fetch from Redis
7. Cache in Local Cache
8. Return from Redis

