---
type: "practice"
difficulty: "Deep"
---

# How would you cache rate-limit counters accurately across many app instances while keeping the counter available if the shared cache blips?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cache Failover and Resilience](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Cache_Invalidation/Cache_Failover_and_Resilience.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To cache rate-limit counters accurately across multiple app instances, we need to ensure consistency and availability in a distributed environment. The key solution involves using a distributed cache like Redis with appropriate data structures and synchronization mechanisms. This approach ensures that the counter is updated reliably even if there's a temporary issue with the cache.

#### Triaging the Question

The question asks how to maintain accurate rate-limit counters across multiple app instances, particularly when dealing with potential failures in the shared cache. The core challenge here involves ensuring consistency (i.e., every instance sees the same updated value) and availability (i.e., no data loss even if the cache is down temporarily).

#### Assumptions
We have multiple backend app instances serving requests.
Rate-limiting logic needs to be enforced uniformly across these instances.
The rate-limiter should persist counters across restarts and temporary cache outages.

#### First Principles Solution

To solve this problem, we'll use a distributed caching mechanism like Redis. Here’s the step-by-step approach:

Choose a Distributed Cache: Use Redis because it offers strong consistency guarantees, low latency, and built-in data structures suitable for counters (e.g., INCR for incrementing).

Key Design: Each app instance will use unique keys in Redis to store rate-limit counter values. For example, the key could be structured as rate_limit:<endpoint>:<user_id>.

Increment Counter on Request:

When a request comes in, each backend app instance should increment the corresponding rate limit counter in Redis.
If the counter exceeds the limit, then the request is rejected.

Handling Cache Blips: To ensure availability during temporary cache outages (e.g., due to network issues or Redis restarts), we can implement a local fallback mechanism:

Each backend instance maintains a local in-memory copy of the rate-limit counter.
If there's no recent update from Redis, use the cached value.

Conflict Resolution: In case of cache blips, ensure that after recovery, all instances are brought back to a consistent state by merging updates. This can be done through:

Periodic synchronization: Have instances periodically poll Redis for updated values.
Eventual Consistency: Use an event-based model where changes in one instance trigger updates in others.

Error Handling and Logging: Implement robust error handling and logging to detect and report any issues with the caching mechanism.

#### Closing Summary

In summary, using a distributed cache like Redis ensures that rate-limit counters are updated consistently across multiple app instances while maintaining availability during temporary outages. The solution involves leveraging Redis's data structures for atomic increments, implementing local fallbacks, and ensuring eventual consistency through periodic updates.

#### Common Mistakes/Follow-Ups
Mistake: Not considering the implications of cache blips can lead to inconsistent rate limiting.

#### Follow-Up
How would you handle a scenario where multiple instances increment the same counter simultaneously during a cache outage?
Further Exploration: Discussing advanced strategies like using Redis Sentinel for high availability, or implementing leader-election algorithms (like Raft) to ensure only one instance updates the counter in case of conflicts.

**Comparison:**

```
Instance	Local Cache	Redis Cache	Action
A	N/A	80	Update to 81
B	N/A	80	Update to 82
C	N/A	80	Update to 83

After Redis recovers, it will eventually converge with the in-memory values.
```

**Code:**

```python
import redis

# Initialize Redis client
redis_client = redis.Redis(host='localhost', port=6379, db=0)

def get_rate_limit_counter(endpoint, user_id):
    key = f"rate_limit:{endpoint}:{user_id}"
    # Check in Redis first
    count = redis_client.get(key)
    if not count:
        # Fallback to local cache
        return 0
    return int(count)

def update_rate_limit_counter(endpoint, user_id, increment_by=1):
    key = f"rate_limit:{endpoint}:{user_id}"
    try:
        # Increment counter in Redis atomically
        redis_client.incr(key)
    except Exception as e:
        # Log the error and handle temporary cache outages
        print(f"Error updating rate limit counter: {e}")
```

**Example input / output:**

```text
Input:
- Endpoint: `/api/v1/resource`
- User ID: `12345`

Output:
- Counter Value (from Redis): `90`

Process:
- Increment counter by 1.
- New Counter Value (after update): `91`.
```

**Data:**

```json
{
  "endpoint": "/api/v1/resource",
  "user_id": "12345"
}
```

**Flow:**

1. Yes
2. No
3. Initial Cache Blip
4. Is Redis Down?
5. Use Local Fallback
6. Periodic Sync with Redis
7. Increment Counter in Redis

