---
type: "practice"
difficulty: "Deep"
---

# A managed cache node failover caused a 30-second stampede on your database. How do you design the cache client and DB access so a failover degrades gracefully?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Read Your Writes with Replicas](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Read_Your_Writes_with_Replicas.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a system to handle managed cache node failures gracefully, we need to ensure that a failover does not result in a "stampede" on our database. A stampede occurs when too many clients simultaneously request access due to the cache failure, overwhelming the database and causing performance issues or even data corruption. To avoid this, we must implement a robust failover strategy for both the cache client and the database access layer.

#### Triage

The question is essentially asking us to design a resilient caching solution that can handle node failures without leading to catastrophic database load spikes. We need to consider how the cache client should behave during a failure (e.g., fallback mechanisms) and how the application interacts with the database when the cache is unavailable. We'll also need to think about scaling and performance implications.

#### First Principles Solution

Graceful Failover for Cache Client:

Implement a read-through cache, where if the cache node fails, the client should attempt to fetch data from the database directly.
Use a caching strategy that allows for stale reads during the failover period (e.g., use weak consistency or allow some staleness).

Degraded Read Handling:

When the cache is unavailable, design your application to degrade gracefully by performing read operations through the database with reduced concurrency and retries.
Implement exponential backoff and jitter for retry logic to prevent a stampede.

Write Consistency:

Ensure that write operations are still handled efficiently, even when the cache node fails. Use distributed transactions or optimistic locking mechanisms if needed.

Monitoring and Alerting:

Set up robust monitoring and alerting systems to detect and respond quickly to cache failures.
Monitor the database load during failovers to ensure it doesn't get overwhelmed.

Load Balancing and Throttling:

Implement load balancing and throttling mechanisms at the application level to manage incoming requests effectively.
Use a caching strategy that minimizes the number of direct DB hits, such as caching read-heavy operations more aggressively.
Detailed Solution
Read-Through Caching with Failover:
When accessing data via the cache client, implement a get method that first checks if the value is present in the local cache.
If not found locally, use an asynchronous call to retrieve from the database and then update the cache before returning the result.
PYTHON
def get(key):
    # Local cache lookup
    value = cache.get(key)
    if value:
        return value

    # Database fallback with retries
    for attempt in range(3):
        try:
            value = db_read(key)  # Asynchronous database read
            cache.set(key, value)  # Update the local cache
            return value
        except Exception as e:
            time.sleep(attempt * 2)  # Exponential backoff with jitter

Degraded Read Handling:

During a cache failure, the application should perform read operations directly from the database.
Use connection pooling and throttling to manage the number of concurrent DB connections.

Write Consistency:

For write operations, use distributed transactions or optimistic locking to ensure consistency across nodes.
Implement idempotent write operations where possible to handle retries without data corruption.

Monitoring and Alerting:

Set up monitoring for cache node health and database load.
Use alerting tools like Prometheus and Grafana to trigger notifications during failures.

Load Balancing and Throttling:

Implement a load balancer that can distribute requests across multiple DB instances.
Add rate limiting at the application level to prevent overwhelming the database with too many concurrent requests.

#### Closing Summary

To degrade gracefully during cache node failovers, we need to implement robust caching strategies, fallback mechanisms for read operations, and reliable monitoring systems. By doing so, we can minimize the impact on our database and ensure that our application remains performant and resilient in the face of failures.

#### Common Mistakes/Follow-Ups

Mistake 1: Failing to implement exponential backoff with jitter during retries.
Mistake 2: Ignoring write consistency during cache failures, leading to data corruption.
Mistake 3: Not monitoring and alerting on critical system metrics like cache health and database load.

**Comparison:**

```
Key	Value
key1	data
```

**Code:**

```python
def get(key):
    # Local cache lookup
    value = cache.get(key)
    if value:
        return value

    # Database fallback with retries
    for attempt in range(3):
        try:
            value = db_read(key)  # Asynchronous database read
            cache.set(key, value)  # Update the local cache
            return value
        except Exception as e:
            time.sleep(attempt * 2 + random.uniform(0, 1))  # Jittered backoff

def db_read(key):
    # Simulate database read operation with potential exceptions
    if not key_exists_in_db(key):
        raise Exception("Key does not exist in DB")
    return get_data_from_db(key)
```

**Example input / output:**

```text
Input: get('key1')
Output:
Step 1: Check local cache for 'key1'
Step 2: Cache miss, perform database read
Step 3: Read success, update cache with key value
Step 4: Return cached value

Input: db_read('key1')
Output: {'value': 'data'}
```

**Data:**

```json
{
  "cache": {
    "type": "redis",
    "nodes": ["10.0.0.1:6379", "10.0.0.2:6379"]
  },
  "database": {
    "type": "mysql",
    "host": "db.example.com",
    "port": 3306
  }
}
```

**Flow:**

1. Yes
2. No
3. Cache Client Request
4. Is Cache Hit?
5. Return Cached Value
6. Database Read
7. Update Cache with Result fromDB
8. Return DB Result

