---
type: "practice"
difficulty: "Deep"
---

# Explain the thundering-herd problem when a cache entry expires and describe cache stampede mitigation: request coalescing, probabilistic early expiration, and locks. Which do you pick under 100k QPS?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cache Invalidation Across Regions](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Cache_Invalidation_Across_Regions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In distributed systems, one common issue arises when a large number of requests hit an expired cache entry simultaneously—a scenario known as the thundering herd problem. When this happens, all those cached entries expire at once, causing a surge in database queries and increasing the load on the database or backend services. To mitigate this, several strategies can be employed: request coalescing, probabilistic early expiration, and locks. Each of these approaches has its trade-offs and is suitable for different scenarios.

To understand which strategy to choose under 100k QPS (queries per second), we need to consider the underlying architecture, latency requirements, data access patterns, and the criticality of the service. Let's dive into each mitigation technique step by step.

#### Triaging the Question

The core of this question revolves around understanding the thundering herd problem and then exploring three different strategies to mitigate it: request coalescing, probabilistic early expiration, and locks. The interviewer is likely probing your knowledge of distributed systems architecture, understanding of concurrency issues, and ability to make informed decisions based on performance metrics.

Request Coalescing

Request coalescing involves combining multiple requests that arrive in quick succession into a single request. This reduces the number of database queries by ensuring only one request triggers an update or expiration event. Here’s how it works:

Caching Layer: When a cache entry is accessed, check if there are any pending requests to update the cache.
Locking Mechanism: Use a distributed lock (e.g., Redis) to ensure that only one request can trigger the cache expiration and database update process.
Coalesced Request: If multiple requests arrive within a short window, they are combined into a single operation.
Advantages:
Reduces the number of database queries.
Ensures consistency by preventing multiple simultaneous updates.
Disadvantages:
Introduces latency due to the distributed lock mechanism.
Can lead to contention if locks are not properly managed.
Probabilistic Early Expiration

Probabilistic early expiration is a strategy that involves setting cache entries to expire slightly earlier than necessary. This reduces the likelihood of all requests hitting an expired entry simultaneously. Here’s how it works:

Randomized Expiry Time: Instead of expiring cache entries at exact time, introduce some randomness by setting them to expire between T and T + Δt.
Load Balancing: Spread out the expiration times across different nodes to ensure a more even distribution.
Advantages:
Reduces the likelihood of simultaneous cache expiry.
Simplifies implementation compared to request coalescing.
Disadvantages:
May lead to slightly stale data if not managed carefully.
Can cause increased memory usage due to potentially more entries in cache.
Locks

Locks are used to ensure mutual exclusion during critical operations, preventing multiple requests from accessing the same resource simultaneously. Here’s how it works:

Cache Expiration Handling: When a cache entry is accessed and found expired, obtain a lock.
Critical Section Execution: Execute the logic to update the cache or fetch data from the database while holding the lock.
Release Lock: Release the lock after the critical section completes.
Advantages:
Ensures consistency during critical operations.
Provides fine-grained control over which resources are accessed concurrently.
Disadvantages:
Introduces latency due to potential lock contention.
Requires careful management of distributed locks to avoid deadlocks and livelocks.
Which Strategy to Pick?

Under 100k QPS, the choice between these strategies depends on several factors:

Latency Requirements: If minimizing delay is critical, request coalescing or locks might be preferred as they reduce unnecessary database hits.
Concurrency Management: For systems with high contention, probabilistic early expiration can help distribute load more evenly.
Resource Availability: Distributed locks require a robust lock management system to ensure reliability and performance.

Given the specific QPS threshold, request coalescing is likely the best choice for minimizing unnecessary database queries while ensuring consistency. Probabilistic early expiration could be an alternative if simpler implementation or better load distribution is more critical than absolute consistency.

#### Closing Summary

In summary, the thundering herd problem can be mitigated using techniques like request coalescing, probabilistic early expiration, and locks. Each strategy has its trade-offs and is suitable for different scenarios. Under 100k QPS, request coalescing is a strong candidate due to its balance of reducing unnecessary database queries while maintaining consistency.

#### Common Mistakes/Follow-Ups
Forgetting Contention Management: Ensure that any locking mechanism is robust and avoids deadlocks.
Ignoring Data Consistency: Probabilistic early expiration can introduce staleness; ensure it’s acceptable for the application.
Overlooking Latency Trade-offs: Understand how each strategy affects overall system latency.

Would you like to explore a specific scenario or another mitigation technique in more detail?

**Comparison:**

```
Strategy	Advantages	Disadvantages
Request Coalescing	Reduces database queries, ensures consistency	Introduces latency due to distributed locks, potential contention
Probabilistic Early Expiration	Simplifies implementation, reduces simultaneous cache expirations	May introduce staleness, increased memory usage
Locks	Ensures mutual exclusion during critical operations	Can lead to deadlock and livelock, increases latency
```

**Code:**

```python
# Example of request coalescing using Redis locks

import redis

def get_client():
    return redis.Redis(host='localhost', port=6379, db=0)

def handle_request(key):
    client = get_client()
    lock_key = f"lock:{key}"

    # Try to acquire the lock
    if client.lock(lock_key, timeout=1).acquire(blocking=False):
        try:
            # Perform cache expiration and database update logic here
            print(f"Processing request for {key}")
        finally:
            client解锁(lock_key)
```

**Example input / output:**

```text
Input: Request for cache entry 'user_123'
Output: Processing request for user_123 (after acquiring lock)
```

**Data:**

```json
{
  "cache_entry": {
    "key": "user_123",
    "value": "John Doe",
    "timestamp": "2023-10-05T14:00:00Z"
  }
}
```

**Flow:**

1. Yes
2. No
3. No
4. Cache Expiration
5. Is there a pending request?
6. Acquire Lock
7. Process CacheExpiration/Database Update
8. Is lock expired?
9. Release Lock
10. Return Cached Data

