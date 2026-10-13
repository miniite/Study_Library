---
type: "practice"
difficulty: "Deep"
---

# You need to give the gateway a way to reach a database-backed auth service without that service being a hard dependency; how do you architect a local cache with a TTL and a fail-open ceiling, and how do you tune staleness under real traffic?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Gateway Caching and Token Validation](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Response_Transformation_and_Upstream_Errors/Gateway_Caching_and_Token_Validation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of providing an API Gateway with a way to reach a database-backed auth service without it being a hard dependency, we need to architect a local cache system that has both a Time-To-Live (TTL) and a fail-open ceiling. This ensures that even if the backend service is down, the gateway can still provide access but with some level of staleness in authentication data. Here’s a step-by-step approach to achieve this:

TL;DR: We will implement a caching layer within the API Gateway using an in-memory cache like Redis or Memcached, which will store auth tokens and their associated metadata (such as expiration times). The gateway will check the cache first before hitting the database-backed service. If the cache misses, it will fall back to the backend service but with some staleness management.

#### Triage
This question is probing your ability to decouple dependencies, manage state locally within a distributed system, and handle failure gracefully. We need to assume that the auth service can be slow or unavailable at times, so we want to cache its responses with an expiration time (TTL) to avoid constant database queries.

#### First-Principles Solution

Local Cache: Use Redis or Memcached for caching tokens and their metadata.
TTL Management: Set a TTL on cached items to ensure they are refreshed periodically from the backend service.
Fail-Open Ceiling: Design a mechanism where if the cache is exhausted, the gateway falls back to the database but only serves data up to the last known good state.

Step-by-Step Solution:

Choose Cache Technology: Select Redis or Memcached for its performance and ease of use.
Cache Key Generation: Generate a key based on the user ID or token, e.g., auth:<token>.
TTL Setting: Set a TTL (e.g., 5 minutes) to refresh the cache from the backend service before it expires.
Local Cache Lookup: In the API Gateway, first attempt to look up the auth token in Redis/Memcached. If found, use the cached data; if not, proceed to the next step.
Backend Service Call and Staleness Management: If the cache is missed, call the backend service. Once the response is received, update the cache with a new TTL. However, if the backend service fails, serve the most recent valid token from the cache (the "fail-open ceiling").

#### Closing Summary

By implementing this architecture, we decouple the API Gateway from the database-backed auth service, making it more resilient and performant.
The fail-open strategy ensures that users can still access services even if the backend is temporarily unavailable or slow.

#### Common Mistakes/Follow-Ups

Failing to set a TTL on cached items, leading to unnecessary database queries.
Not handling stale data gracefully, which could lead to security vulnerabilities or incorrect service invocation.
Over-relying on the cache without proper fallback mechanisms, causing performance degradation.

**Comparison:**

```
Cache Key	Token	TTL (Seconds)
auth:12345	token_12345	300
```

**Code:**

```python
import redis

# Initialize Redis client
redis_client = redis.Redis(host='localhost', port=6379, db=0)

def get_auth_token(user_id):
    token_key = f'auth:{user_id}'

    # Check cache first
    cached_token = redis_client.get(token_key)
    if cached_token:
        return cached_token.decode('utf-8')

    # Fallback to backend service (assuming this function handles the call and returns a token)
    db_token = get_auth_token_from_db(user_id)

    # Update cache with new TTL
    redis_client.set(token_key, db_token, ex=300)  # Set TTL to 5 minutes

    return db_token

def get_auth_token_from_db(user_id):
    # Simulate database call (for illustrative purposes)
    import time
    time.sleep(2)  # Simulate slow DB response
    return f'token_{user_id}'
```

**Example input / output:**

```
Input: get_auth_token('12345')
Output:
JSON
Copy
{
  "cached_token": "token_12345"
}
or in case of a cache miss,
JSON
Copy
{
  "db_token": "token_12345",
  "update_cache": true
}
```

**Data:**

```json
{
  "cache_key": "auth:12345",
  "token": "token_12345",
  "ttl": 300
}
```

**Flow:**

1. API Gateway
2. Cache Lookup
3. Cache Hit
4. Use Cached Token
5. Cache Miss
6. Call Backend Service
7. Get Auth Token from DB
8. Update Cache with New TTL
9. Use Retrieved Token

