---
type: "practice"
difficulty: "Deep"
---

# How would you integrate Redis as a cache in FastAPI and invalidate cached entries when data changes?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Caching Predictions](<../../../../TOOLS/Python/FastAPI/Caching_Predictions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To integrate Redis as a cache in FastAPI, we need to focus on two key aspects: setting up the cache and invalidating it when underlying data changes. This involves creating appropriate decorators or middleware that handle caching and using the redis-py client for interactions with Redis.

#### Triage
The question is asking how to ensure that cached responses are invalidated whenever there's a change in the database. We need to cover setting up FastAPI, configuring Redis, and implementing cache invalidation mechanisms. Assumptions include having a working setup of FastAPI and basic knowledge of Redis operations.

#### First-Principles Solution

Setup FastAPI: Ensure you have a running FastAPI application.
Install Redis: Install the redis-py library via pip.
Cache Decorator: Create a custom cache decorator to handle caching logic.
Database Changes Notification: Implement a mechanism to notify the cache invalidation when database changes occur.

Let's walk through each step in detail:

Setup FastAPI:

PYTHON
from fastapi import FastAPI

app = FastAPI()

Install Redis:

SH
pip install redis

Cache Decorator:

PYTHON
from functools import wraps
import time
import redis

cache = redis.Redis(host='localhost', port=6379, db=0)

def cache_it(key_prefix):
    """Custom cache decorator."""
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            key = f"{key_prefix}:{time.time()}"  # Use time to prevent key collisions
            if not cache.exists(key):
                result = await func(*args, **kwargs)
                cache.setex(key, 3600, result)  # Cache for an hour
                return result
            else:
                return cache.get(key)
        return wrapper
    return decorator

@app.get("/data/{id}")
@cache_it("data")
async def get_data(id: int):
    # Simulate database fetch
    await time.sleep(2)  # Emulating slow DB lookup
    return {"id": id, "value": f"Data for {id}"}

Database Changes Notification: This step requires integrating with your existing data storage mechanism. For example, if you use a database like PostgreSQL, you can set up triggers or listen to DDL events.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

Ensure proper error handling in Redis interactions.
Use TTL (Time To Live) wisely to balance cache freshness and resource usage.

#### Cons
ider edge cases such as network issues with Redis.
In production, use a more robust caching solution like Kubernetes Service Mesh or a dedicated caching service.

**Code:**

```python
from fastapi import FastAPI
import time
import redis

app = FastAPI()
cache = redis.Redis(host='localhost', port=6379, db=0)

def cache_it(key_prefix):
    """Custom cache decorator."""
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            key = f"{key_prefix}:{time.time()}"
            if not cache.exists(key):
                result = await func(*args, **kwargs)
                cache.setex(key, 3600, result)  # Cache for an hour
                return result
            else:
                return cache.get(key)
        return wrapper
    return decorator

@app.get("/data/{id}")
@cache_it("data")
async def get_data(id: int):
    await time.sleep(2)
    return {"id": id, "value": f"Data for {id}"}
```

**Example input / output:**

```text
Input:
GET /data/1

Output:
{"id": 1, "value": "Data for 1"}
```

**Flow:**

1. Yes
2. No
3. Database Query
4. Cache Hit?
5. Return Cache
6. Fetch Data from DB
7. Store in Cache
8. Return Data

