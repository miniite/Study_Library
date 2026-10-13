---
type: "practice"
difficulty: "Deep"
---

# How would you pre-warm a cache before a big sale or product launch to avoid a traffic spike hitting the DB?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Cache-aside and Read Replicas](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Cache-aside_and_Read_Replicas.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To pre-warm a cache before a big sale or product launch, we need to ensure that relevant data is readily available in memory to handle the anticipated high traffic. This involves loading and keeping frequently accessed data in the cache before the event starts. Let's break down how we can achieve this from first principles.

Firstly, understanding the concept of caching is crucial: caching stores copies of data so that future requests for that data can be served faster by accessing the cached copy instead of retrieving it from a slower or more expensive storage mechanism like a database.

For our scenario, the primary goal is to minimize latency and reduce the load on the database. To achieve this, we'll focus on:

Identifying the critical data.
Loading that data into the cache.
Validating the cache's effectiveness.
Managing cache invalidation and expiration.

#### Triaging the Question

The interviewer is probing how to efficiently manage a cache in a high-traffic scenario. Key aspects include understanding the data access patterns, choosing appropriate cache mechanisms, managing cache size, and ensuring that the cache remains up-to-date without overwhelming the database.

#### First Principles Solution
Step 1: Identify Critical Data

Identify which products or categories are most likely to be accessed during the sale. For instance, best-selling items, new launches, or frequently purchased combinations might need to be pre-warmed.

SQL
-- Example SQL query to fetch top-selling items before the event
SELECT * FROM products WHERE is_best_selling = true;
Step 2: Load Data into Cache

Use a cache system that supports in-memory storage like Redis. We can use an HTTP request or a script to populate the cache with this data.

PYTHON
import requests

def pre_warm_cache():
    url = "http://cache.example.com/warmup"
    headers = {'Content-Type': 'application/json'}
    response = requests.post(url, json={"key": "best_selling_items", "value": fetch_top_products()})
    if response.status_code != 200:
        raise Exception("Failed to pre-warm cache")
Step 3: Validate Cache Effectiveness

Ensure that the cached data is being used by monitoring requests. Tools like New Relic or Prometheus can help track cache hits and misses.

BASH
# Example command to check cache hit ratio using New Relic Insights
newrelic insights query "SELECT count(*) FROM 'cache_hits' WHERE application='example'"
Step 4: Manage Cache Invalidations and Expirations

Set appropriate expiration times for the cached data. For highly volatile data, consider a shorter TTL (Time To Live). For static or rarely changing data, longer TTLs can be used.

PYTHON
# Example configuration in Redis
redis.setex("best_selling_items", 3600, fetch_top_products())

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, pre-warming a cache involves identifying critical data, loading it into the cache, validating its effectiveness, and managing cache invalidations to ensure freshness. Common mistakes include overloading the database with unnecessary pre-warm operations or setting excessively long TTLs that could lead to stale data.

#### Follow-up
questions might involve discussing different cache strategies (e.g., distributed caching, local caches), dealing with cache consistency issues, or integrating this into a larger system architecture.

**Comparison:**

```
Key	Value
key	best_selling_items
value	[{"id": 1, "name": "Product A"}, {"id": 2, "name": "Product B"}]
```

**Code:**

```python
import requests

def pre_warm_cache():
    url = "http://cache.example.com/warmup"
    headers = {'Content-Type': 'application/json'}
    response = requests.post(url, json={"key": "best_selling_items", "value": fetch_top_products()})
    if response.status_code != 200:
        raise Exception("Failed to pre-warm cache")

def fetch_top_products():
    # Example SQL query
    return [{"id": 1, "name": "Product A"}, {"id": 2, "name": "Product B"}]
```

**Example input / output:**

```

Input:

BASH
Copy
curl -X POST http://cache.example.com/warmup -H 'Content-Type: application/json' -d '{"key": "best_selling_items", "value": [{"id": 1, "name": "Product A"}, {"id": 2, "name": "Product B"}]}'

Output:

JSON
Copy
{"status": "Success"}
```

**Data:**

```json
{
  "key": "best_selling_items",
  "value": [
    { "id": 1, "name": "Product A" },
    { "id": 2, "name": "Product B" }
  ]
}
```

**Flow:**

1. Identify Critical Data
2. Pre-Warm Cache
3. Load Data into Cache
4. Validate Cache Effectiveness
5. Manage Cache Invalidationsand Expirations

