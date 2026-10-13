---
type: "practice"
difficulty: "Deep"
---

# Your managed cache evicts hot keys under memory pressure causing a database stampede. How would you pick an eviction policy and add protection so eviction degrades gracefully?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Read Your Writes with Replicas](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Read_Your_Writes_with_Replicas.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of managed cache evictions leading to a database stampede, we need to carefully select an eviction policy that minimizes the impact on our application's performance while ensuring data consistency and availability. We'll focus on Least Recently Used (LRU) policies with additional protections to prevent cascading failures.

Firstly, let’s summarize the problem: When a managed cache is under memory pressure and starts evicting hot keys, it can trigger a database stampede where multiple requests race to access the same key in the database. This can lead to increased load on the database and potential performance bottlenecks or even service degradation.

#### Triage

The core of this question lies in understanding the eviction strategy and ensuring that the cache evictions do not cascade into harmful database interactions. The interviewer is probing our knowledge of caching principles, the trade-offs between different policies, and how to design robust systems that handle unexpected load spikes gracefully.

#### Assumptions
We are using a managed cache service.
The cache uses LRU eviction by default but we need to fine-tune it.
There are mechanisms in place for detecting high load or memory pressure.
We can monitor the performance of both the cache and database.

#### First Principles Solution

Choose an Appropriate Eviction Policy:

LRU (Least Recently Used): This is a common choice as it tends to remove items that haven’t been used recently, which might be good candidates for eviction.
However, with LRU, we must ensure that frequently accessed keys are not evicted quickly. We can fine-tune the policy by adjusting parameters like max-age and max-idle-time.

Implement Key Protection Mechanisms:

Warm-up Keys: Identify hot keys early on and keep them pinned in memory to avoid eviction.
Graceful Degradation: Design fallback mechanisms that allow the system to continue functioning even if a key is evicted.

Load Detection and Thresholds:

Monitor cache hit rates, eviction rates, and database load. Define thresholds for when to switch from cache reads to database reads.
Implement circuit breakers or rate limiting on the cache side to prevent overwhelming the database.

Database Protection:

Use connection pooling in your application to manage database connections efficiently.
Ensure that read operations are non-blocking and can be retried with exponential backoff if necessary.

#### Example
Implementation

To illustrate, let's assume we have a managed Redis cache service with the following configuration:

PYTHON
from redis import StrictRedis, ConnectionPool

# Initialize connection pool with max idle time and timeout settings to handle memory pressure gracefully.
pool = ConnectionPool(
    host='localhost',
    port=6379,
    max_connections=100,
    max_idle_time=3600  # 1 hour
)

redis_client = StrictRedis(connection_pool=pool)

# Eviction policy configuration
redis_client.config_set('maxmemory-policy', 'allkeys-lru')  # Use LRU eviction

def get_key(key):
    try:
        value = redis_client.get(key)
        if not value:
            # If key is evicted, fall back to database read.
            with redis_client.lock(f'lock:{key}', timeout=10) as lock:
                if lock.acquire(blocking=False):
                    value = fetch_from_database(key)  # Simulate DB read
                    redis_client.setex(key, 3600, value)  # Cache the value for an hour
    except Exception as e:
        print(f"Error fetching key {key}: {e}")
    return value

def fetch_from_database(key):
    # Simulate database fetch operation
    import time
    time.sleep(1)
    return f"Value from DB: {key}"

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, we need to carefully choose an eviction policy that balances memory usage with application performance. Implementing key protections like warm-up mechanisms and graceful degradation strategies ensures the system handles high load scenarios more robustly. Additionally, monitoring and tuning thresholds can help prevent database stampedes.

#### Common mistakes
often include:

Overlooking the need for key protections.
Ignoring the importance of fallback mechanisms in case of cache evictions.
Not implementing proper load detection to preemptively handle memory pressure.

During follow-ups, the interviewer might ask about specific configuration parameters, alternative eviction policies (like LFU or TTL-based), or real-world examples of similar issues and how they were addressed.

**Comparison:**

```
Key	Value
hot-key	Cached value
cold-key	Retrieved from DB, cached
```

**Code:**

```python
from redis import StrictRedis, ConnectionPool

pool = ConnectionPool(
    host='localhost',
    port=6379,
    max_connections=100,
    max_idle_time=3600  # 1 hour
)

redis_client = StrictRedis(connection_pool=pool)

redis_client.config_set('maxmemory-policy', 'allkeys-lru')  # Use LRU eviction

def get_key(key):
    try:
        value = redis_client.get(key)
        if not value:
            with redis_client.lock(f'lock:{key}', timeout=10) as lock:
                if lock.acquire(blocking=False):
                    value = fetch_from_database(key)
                    redis_client.setex(key, 3600, value)
    except Exception as e:
        print(f"Error fetching key {key}: {e}")
    return value

def fetch_from_database(key):
    import time
    time.sleep(1)
    return f"Value from DB: {key}"
```

**Example input / output:**

```text
# Example usage of get_key function
print(get_key('hot-key'))  # Fetches from cache if available or database
print(get_key('cold-key'))  # Fallback to database read and caches the result for an hour
```

**Data:**

```yaml
# Example configuration for Redis connection pool
host: localhost
port: 6379
max_connections: 100
max_idle_time: 3600  # in seconds, 1 hour
```

**Flow:**

1. Yes
2. No
3. Cache Hit
4. Is key evicted?
5. Fall back to database
6. Fetch and cache the value foran hour
7. Return cached value

