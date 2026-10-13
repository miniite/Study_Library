---
type: "practice"
difficulty: "Deep"
---

# How would you implement per-request locking or a mutex so only one caller repopulates a hot key on expiry?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Cache Stampede Prevention](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Cache_Stampede_Prevention.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When implementing a caching solution, it's essential to ensure that critical operations like cache repopulation are atomic and thread-safe. In this scenario, we need to implement per-request locking or a mutex mechanism to guarantee that only one caller repopulates a hot key on expiry. This ensures data consistency and prevents race conditions.

#### Triage

The question is essentially asking how to implement a mutual exclusion (mutex) mechanism at the request level for cache management. The core concern here is ensuring that during cache expiry, only one process can recompute and update the cache while others wait. We need to consider both the technical implementation and the broader implications on system architecture.

#### Assumptions
Caching System: Assume we are working with an in-memory caching solution like Redis.
Language & Framework: We will use Python for this example, but the principles can be applied to other languages as well.
Concurrent Requests: The cache key in question is frequently accessed and potentially expires often.

#### First-Principles Solution
1. Cache Expiry Handling

When a cache entry expires, we need to check if another request has already started repopulating it. If not, start the repopulation process; otherwise, wait until the current process completes.

2. Mutex Implementation

We can use Redis's SETNX (Set if Not Exists) command to implement this functionality. SETNX allows us to atomically set a key-value pair only if the key does not already exist. Additionally, we can use WATCH and EXEC commands from within a transaction block to ensure atomic operations.

3. Workflow
Check Cache Expiry: Check if the cache is expired.
Acquire Mutex: Use Redis's SETNX command to try setting a key (e.g., lock_key) that will act as a mutex.
PYTHON
from redis import StrictRedis

def acquire_lock(redis_client: StrictRedis, lock_key: str) -> bool:
    return redis_client.setnx(lock_key, 1)
If Lock Acquired:
Repopulate the cache.
Release the lock using DEL.
If Lock Not Acquired:
Wait for the current lock to release by periodically checking again.
4. Implementation Example

Here's a more detailed implementation in Python:

PYTHON
import time
from redis import StrictRedis

def repopulate_cache(redis_client: StrictRedis, cache_key: str):
    # Check if the key exists and is expired (assuming an expiry policy)
    if not redis_client.exists(cache_key) or int(redis_client.ttl(cache_key)) <= 0:
        # Try to acquire a lock
        lock_key = f'cache_replication:{cache_key}'
        if acquire_lock(redis_client, lock_key):
            try:
                # Repopulate the cache
                value = get_data_from_source()  # Simulating data retrieval from another source
                redis_client.set(cache_key, value)
                # Set a new TTL for the cache to avoid future locks
                redis_client.expire(cache_key, 3600)  # Cache valid for 1 hour
            finally:
                # Always release the lock
                redis_client.delete(lock_key)
        else:
            print(f"Waiting for lock on {cache_key}...")
            while not acquire_lock(redis_client, lock_key):
                time.sleep(0.5)

def get_data_from_source() -> str:
    return "Sample Data"

#### Closing Summary
and Common Mistakes

In summary, we have implemented a per-request locking mechanism using Redis to ensure that only one request at a time can repopulate the cache when it expires. This approach leverages SETNX for lock acquisition and DEL for releasing the lock.

#### Common Mistakes

Not Handling Lock Release Properly: Always ensure that the lock is released even if an error occurs during cache repopulation.
Race Conditions: Not using transactions (e.g., WATCH, EXEC) can lead to race conditions, especially in high-concurrency scenarios.
Incorrect Expiry Logic: Ensure that the cache expiry logic correctly identifies when the cache should be repopulated.

#### Follow-Ups
What if Redis is not Available?
Use a fallback mechanism or a local semaphore implementation.
Handling High Concurrency:

#### Cons
ider using a more advanced locking strategy like distributed semaphores.
Performance Implications:
Evaluate the performance impact of acquiring and releasing locks frequently.

**Code:**

```python
import time
from redis import StrictRedis

def acquire_lock(redis_client: StrictRedis, lock_key: str) -> bool:
    return redis_client.setnx(lock_key, 1)

def repopulate_cache(redis_client: StrictRedis, cache_key: str):
    lock_key = f'cache_replication:{cache_key}'
    if not redis_client.exists(cache_key) or int(redis_client.ttl(cache_key)) <= 0:
        if acquire_lock(redis_client, lock_key):
            try:
                value = get_data_from_source()  # Simulating data retrieval from another source
                redis_client.set(cache_key, value)
                redis_client.expire(cache_key, 3600)  # Cache valid for 1 hour
            finally:
                redis_client.delete(lock_key)
        else:
            print(f"Waiting for lock on {cache_key}...")
            while not acquire_lock(redis_client, lock_key):
                time.sleep(0.5)

def get_data_from_source() -> str:
    return "Sample Data"
```

**Flow:**

1. Yes
2. No
3. Check Cache Expiry
4. Acquire Lock
5. Lock Acquired?
6. Repopulate Cache
7. Wait for Lock
8. Retry Check

