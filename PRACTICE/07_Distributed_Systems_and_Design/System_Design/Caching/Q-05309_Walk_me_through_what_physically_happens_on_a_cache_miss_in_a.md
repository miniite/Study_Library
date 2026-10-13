---
type: "practice"
difficulty: "Intermediate"
---

# Walk me through what physically happens on a cache miss in a cache-aside setup — every hop involved.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Cache Failover and Resilience](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Cache_Invalidation/Cache_Failover_and_Resilience.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a cache-aside setup, when there's a cache miss, several key operations happen to fulfill the request. This process involves multiple layers and components, from the application layer to the database layer, and includes interactions with caches at each step.

To start, let's assume we have an application that uses a local in-memory cache (e.g., Redis) alongside a relational database system (e.g., PostgreSQL). When a client requests data, the first hop is always the application. If the requested data isn't found in the local cache, it triggers a cache miss.

Application Layer: The application checks its local cache (Redis).
Cache Miss Detected: Since the data isn’t present in the Redis cache, the application determines that it needs to fetch the data from the database.
Network Hop 1: The request is sent over the network to the database server.

Next, we'll delve into the details of what happens at each of these hops.

**Comparison:**

```
Hop	Description
Application Layer	Checks local cache, detects a miss
Network Hop 1	Request sent to the database server
Database Server	Processes query, returns data
Cache Update	Data stored in Redis with an expiry
```

**Code:**

```python
# Example Python code snippet for checking cache and fetching data
def fetch_data(user_id):
    # Check local in-memory cache (Redis)
    if redis_client.exists(f"user:{user_id}"):
        return json.loads(redis_client.get(f"user:{user_id}"))

    # Cache miss, proceed to database query
    db_response = db.execute_query("SELECT * FROM users WHERE id = %s", user_id)
    # Store data in cache before returning
    redis_client.setex(f"user:{user_id}", 3600, json.dumps(db_response))
    return db_response
```

**Example input / output:**

```text
Input: fetch_data(123)
Output:
- Redis check for key 'user:123'
- Network request to database server (db.execute_query("SELECT * FROM users WHERE id = %s", user_id))
- Data returned from database and stored in Redis with an expiry of 3600 seconds
```

**Data:**

```json
{
  "redis": {
    "existsKey": "user:123",
    "key": "user:123"
  },
  "db": {
    "query": "SELECT * FROM users WHERE id = %s",
    "parameters": [123]
  }
}
```

**Flow:**

1. No
2. Application
3. Local Cache Check
4. Cache Hit?
5. Send Request to DB Server
6. DB Response Received
7. Store Data in Cache
8. Return Data to Client

