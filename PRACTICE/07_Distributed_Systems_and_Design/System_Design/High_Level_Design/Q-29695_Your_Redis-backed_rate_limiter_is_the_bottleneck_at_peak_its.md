---
type: "practice"
difficulty: "Deep"
---

# Your Redis-backed rate limiter is the bottleneck at peak — its INCR calls are adding tail latency — how do you shard the limiter keys and batch increments while keeping the limit approximately correct?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Rate Limiter Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Distributed_Rate_Limiter_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the bottleneck in our Redis-backed rate limiter, we need to implement a sharding strategy that distributes the workload across multiple Redis instances and batches increment operations to reduce latency. The core idea is to use consistent hashing or a custom key distribution algorithm to shard the rate-limiter keys. Additionally, we'll batch incremental updates to limit calls to Redis as much as possible.

#### Triaging the Question

The question is probing how to scale our rate limiter by sharding and batching operations while maintaining accuracy. The assumptions are:

We have multiple Redis instances available for sharding.
Each Redis instance can handle a certain number of concurrent requests.
We need to ensure that the limit remains approximately correct even with sharded keys.

#### First-Principles Solution
Sharding Strategy: Use consistent hashing to map rate-limiter keys to specific Redis instances. This ensures an even distribution of load and avoids hotspots.
Batching Increment Operations: Instead of performing increment operations one by one, batch them to minimize the number of calls to Redis.

#### Implementation
Steps

Sharding with Consistent Hashing:

Define a consistent hashing function that maps rate-limiter keys to specific Redis instances based on their hash values.
For example, if we have N Redis instances and an increment request for key k, compute the hash value of k % N to determine which instance it should be sent to.

Batch Increment Operations:

Implement a local cache (e.g., in-memory store) on each client-side where requests are aggregated before being sent to Redis.
Periodically flush this cache to Redis, reducing the number of individual increment operations and spreading them over time.
Code Example
PYTHON
import hashlib

class RateLimiter:
    def __init__(self, redis_instances):
        self.redis_instances = redis_instances  # List of Redis instance connections
        self.local_cache = {}  # Local in-memory cache for batching increments

    def increment(self, key):
        if key not in self.local_cache:
            self.local_cache[key] = 0

        self.local_cache[key] += 1

        # Periodically flush the local cache to Redis
        if len(self.local_cache) > batch_size:
            self.flush_cache()

    def flush_cache(self):
        for key, count in self.local_cache.items():
            instance_index = hash(key) % len(self.redis_instances)
            redis_instance = self.redis_instances[instance_index]
            redis_instance.incr(key, count)

# Example usage
redis_instances = [r1, r2, r3]  # List of Redis instances
limiter = RateLimiter(redis_instances)
for _ in range(1000):
    limiter.increment('user:1')

#### Closing Summary
and Common Mistakes/Follow-Ups

By implementing consistent hashing for sharding and local batching to minimize individual increment operations, we can significantly reduce tail latency. Key points include ensuring an even distribution of load across Redis instances and maintaining a balance between cache size and flush frequency.

#### Common mistakes

Failing to properly shard keys, leading to imbalanced loads.
Not batching increments sufficiently, resulting in excessive Redis calls.
Ignoring the need for periodic flushing, which can lead to data loss or inconsistent limits.

For follow-ups, we might discuss more advanced techniques like using time-series buckets for rate limiting and handling distributed cache invalidation scenarios.

**Code:**

```python
import hashlib

class RateLimiter:
    def __init__(self, redis_instances):
        self.redis_instances = redis_instances  # List of Redis instance connections
        self.local_cache = {}  # Local in-memory cache for batching increments

    def increment(self, key):
        if key not in self.local_cache:
            self.local_cache[key] = 0

        self.local_cache[key] += 1

        # Periodically flush the local cache to Redis
        if len(self.local_cache) > batch_size:
            self.flush_cache()

    def flush_cache(self):
        for key, count in self.local_cache.items():
            instance_index = hash(key) % len(self.redis_instances)
            redis_instance = self.redis_instances[instance_index]
            redis_instance.incr(key, count)

# Example usage
redis_instances = [r1, r2, r3]  # List of Redis instances
limiter = RateLimiter(redis_instances)
for _ in range(1000):
    limiter.increment('user:1')
```

