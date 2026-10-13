---
type: "practice"
difficulty: "Deep"
---

# Design a distributed cache layer in front of a database — how do you choose write-through vs write-back vs cache-aside, and how do you handle stampede on a hot key expiry?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Cache Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Distributed_Cache_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a high-level distributed cache layer in front of a database, we need to consider several factors including the trade-offs between write-through, write-back, and cache-aside strategies, as well as how to handle stampede events when keys expire or become hot. This problem involves understanding the application's data access patterns, consistency requirements, and performance needs.

#### Triaging the Question

The question is asking for a detailed high-level design of a distributed caching strategy, which involves choosing between different cache update policies (write-through, write-back, cache-aside) and addressing potential issues such as stampede events. The interviewer is probing your understanding of caching mechanisms, consistency models, system scalability, and fault tolerance.

#### Assumptions
We are designing this for a web application with high read-to-write ratios.
The database backend supports ACID properties (Atomicity, Consistency, Isolation, Durability).
The cache layer will be distributed across multiple nodes to handle load balancing.
The system experiences significant data traffic.

#### First-Principles Solution
Write-Through Strategy

In a write-through strategy, writes are immediately propagated to both the database and the cache. This ensures strong consistency but can lead to performance bottlenecks if the cache is frequently updated. It’s suitable for scenarios where the most recent version of data is critical.

Write-Back (Write-Behind) Strategy

Write-back caching defers updates to the cache until a certain policy triggers a flush to the database, such as expiration or after a period of inactivity. This reduces write pressure on the database but may introduce latency and eventual consistency issues. It's ideal for scenarios with high read-to-write ratios where stale data can be tolerated.

Cache-Aside Strategy

Cache-aside involves using the cache only for reads; writes go directly to the database. Reads from the cache are invalidated when the underlying data changes, which is often managed by application logic. This strategy provides a good balance between performance and consistency but requires careful management of read-misses.

Handling Stampede Events

A stampede event occurs when many requests hit an expired key simultaneously, causing them to access the database concurrently, leading to potential hotspots and performance degradation. To handle this:

Read-Through with Expiration: When a cache miss happens for a key that has expired, fetch data from the database but update the cache only if certain conditions are met (e.g., low load).
Rate Limiting: Implement rate limiting to control the number of concurrent requests hitting the database.
Load Balancing: Use a distributed cache with multiple nodes to distribute read and write loads.
Database Connection Pooling: Optimize database connection usage by reusing connections.

#### Implementation
Steps
Determine Data Access Patterns: Analyze which data are frequently accessed and updated.

#### Cons
istency Model Selection: Choose between strong consistency (write-through), eventual consistency (write-back, cache-aside) based on the application's requirements.
Cache Configuration: Set appropriate cache expiration times and eviction policies to minimize stale data issues.
Load Testing: Simulate heavy load conditions to test cache performance and handle stampede scenarios.

#### Closing Summary

In summary, designing a distributed cache layer involves carefully choosing between write-through, write-back, and cache-aside strategies based on the application's specific needs for consistency and performance. Proper handling of stampede events is crucial to ensure system robustness under heavy load conditions.

#### Common Mistakes/Follow-Ups
Ignoring Application Specifics: Always consider the unique characteristics of your application (e.g., real-time vs non-real-time data) when deciding on caching strategies.
Over-reliance on Cache: Ensure that cache misses do not lead to a degradation in overall system performance or availability.
Underestimating Stampede Risk: Failing to implement robust mechanisms to handle concurrent access can lead to significant performance issues.

**Comparison:**

```
Strategy	Description	Pros	Cons
Write-Through	Writes are immediately propagated to both cache and database.	Ensures strong consistency.	High write load on the database.
Write-Back (Write-Behind)	Writes are deferring updates until a certain policy triggers flushes.	Reduces write pressure on the database.	Stale data issues, eventual consistency.
Cache-Aside	Cache is used for reads; writes go directly to the database.	Balanced between performance and consistency.	Requires careful management of read-misses.
```

**Code:**

```python
# Example of implementing read-through with expiration logic
def get_data(key):
    if cache.exists(key):  # Check if data is in cache
        return cache.get(key)

    # If not in cache, fetch from database and update cache
    data = db.get_data(key)  # Simulate fetching from database
    cache.set(key, data, timeout=3600)  # Set a timeout for cache expiration
    return data
```

**Example input / output:**

```text
Input:
get_data("key1")

Output:
"data1"

Dry Run:
- Cache exists for key1, return cached value.
```

**Data:**

```json
{
  "cache": {
    "key1": "data1",
    "key2": "data2"
  },
  "db": {
    "key1": "original_data1",
    "key2": "original_data2"
  }
}
```

**Flow:**

1. Yes
2. No
3. Get Data
4. Cache Hit?
5. Return Cache Value
6. Fetch from Database
7. Update Cache if Necessary
8. Return Data

