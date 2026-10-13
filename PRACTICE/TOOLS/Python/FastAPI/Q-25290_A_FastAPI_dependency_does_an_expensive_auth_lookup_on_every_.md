---
type: "practice"
difficulty: "Deep"
---

# A FastAPI dependency does an expensive auth lookup on every request; how would you cache it correctly per-request versus across-requests, and what invalidation bug could that introduce?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Caching Predictions](<../../../../TOOLS/Python/FastAPI/Caching_Predictions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this FastAPI dependency issue, we need to understand when caching is appropriate and how to avoid common pitfalls. The core challenge is balancing performance by caching expensive auth lookups while ensuring data consistency.

TL;DR
Per-Request Caching: Use a per-request cache mechanism like FastAPI's Depends to store the result of the auth lookup for each request.
Across-Requests Caching: Implement an across-requests cache using a key-value store (like Redis) with appropriate expiration times and invalidation strategies.

#### Triage

The question is about optimizing performance in a scenario where an expensive authorization lookup needs to be performed on every request, but we want to avoid unnecessary computations. The goal is to balance between caching the auth results per-request or across multiple requests while ensuring data consistency and avoiding potential bugs.

#### Assumptions
The auth lookup is computationally expensive.
We use FastAPI for web development.
We are using a Redis cache as our key-value store.

#### First Principles Solution

Per-Request Caching

Use FastAPI's Depends to define a dependency that performs the auth lookup and stores the result in a local context.

#### Example
PYTHON
from fastapi import Depends, HTTPException

cache = {}

async def get_auth_info(token: str) -> dict:
    if token not in cache:
        # Perform expensive auth lookup here
        auth_info = await perform_expensive_auth_lookup(token)
        cache[token] = auth_info
    return cache[token]

@app.get("/user")
async def read_user(auth_info=Depends(get_auth_info)):
    # Use the auth_info to process user requests
    ...

Across-Requests Caching

Store the auth results in a Redis cache with appropriate keys and expiration times.

#### Example
PYTHON
import os
from fastapi import Depends, HTTPException
from redis import asyncio as aioredis

REDIS_URL = "redis://localhost:6379"
conn = aioredis.from_url(REDIS_URL)

async def get_auth_info(token: str) -> dict:
    key = f"auth:{token}"
    auth_info = await conn.get(key)
    if auth_info is None:
        # Perform expensive auth lookup here
        auth_info = await perform_expensive_auth_lookup(token)
        await conn.set(key, auth_info, ex=60 * 5)  # Cache for 5 minutes
    return json.loads(auth_info)

@app.get("/user")
async def read_user(auth_info=Depends(get_auth_info)):
    # Use the auth_info to process user requests
    ...
Invalidation Bug

One common pitfall is not properly invalidating the cache when the underlying data changes. For instance, if a user's role or permissions are updated in the database, we need to ensure that any cached auth info reflects these changes.

#### Example
Invalidation Bug:
Scenario: A user's role is changed through an admin interface.
Bug: The FastAPI dependency cache (either per-request or across-requests) might not be invalidated. If a user makes another request before the cache expires, they will still see outdated auth info.

#### Closing Summary

By carefully considering both per-request and across-requests caching strategies, we can optimize performance while ensuring data consistency. Using Redis for cross-request caching provides a good balance between performance and reliability but requires careful invalidation mechanisms to handle state changes.

#### Common Mistakes/Follow-Ups
Failing to Invalidate Caches: Not updating the cache when underlying data changes.
Incorrect Cache Expiration Times: Setting overly short or long expiration times can lead to either too many redundant lookups or stale data issues.
Not Handling Edge Cases: For example, handling concurrent updates and ensuring thread safety.

**Comparison:**

```
Cache Strategy	Description
Per-request	Caches auth info per request in local context. Useful for small, short-lived caches.
Across-requests	Caches auth info in Redis with expiration times. Suitable for larger data sets and longer-term storage.
```

**Code:**

```

from fastapi import Depends, HTTPException

Per-request caching example

cache = {}

async def get_auth_info(token: str) -> dict: if token not in cache: # Perform expensive auth lookup here auth_info = await perform_expensive_auth_lookup(token) cache[token] = auth_info return cache[token]

@app.get("/user") async def read_user(auth_info=Depends(get_auth_info)): # Use the auth_info to process user requests ...

CODE
Copy

```python
import os
from fastapi import Depends, HTTPException
from redis import asyncio as aioredis

# Across-requests caching example
REDIS_URL = "redis://localhost:6379"
conn = aioredis.from_url(REDIS_URL)

async def get_auth_info(token: str) -> dict:
    key = f"auth:{token}"
    auth_info = await conn.get(key)
    if auth_info is None:
        # Perform expensive auth lookup here
        auth_info = await perform_expensive_auth_lookup(token)
        await conn.set(key, auth_info, ex=60 * 5)  # Cache for 5 minutes
    return json.loads(auth_info)

@app.get("/user")
async def read_user(auth_info=Depends(get_auth_info)):
    # Use the auth_info to process user requests
    ...
```

**Example input / output:**

```text
# Input: GET /user?token=abc123
# Output: {"role": "admin", "permissions": ["read", "write"]}
```

**Data:**

```yaml
# Example Redis key and value for caching
auth:abc123: {"role": "admin", "permissions": ["read", "write"]}
```

**Flow:**

1. Yes
2. No
3. Success
4. Failure
5. User makes request
6. Is cache hit?
7. Return cached auth info
8. Perform expensive auth lookup
9. Cache auth info in Redis
10. Throw error or default tonon-cached auth
11. Use cached auth info toprocess request

