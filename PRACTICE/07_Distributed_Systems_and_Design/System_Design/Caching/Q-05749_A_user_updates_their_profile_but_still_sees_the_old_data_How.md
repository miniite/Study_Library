---
type: "practice"
difficulty: "Intermediate"
---

# A user updates their profile but still sees the old data. How would you debug this caching issue?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Stale While Revalidate](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Stale_While_Revalidate.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To debug a caching issue where a user sees outdated profile data after updating, we need to methodically trace the flow of data from the update request to how it's stored and served. This involves understanding the cache layers in use, their configurations, and the interactions between different components.

Triaging the Issue

The question is asking us to diagnose why caching isn't working as expected, ensuring that updates are reflected correctly. Here are some key areas to investigate:

Cache Layers: Determine if there's a multi-layered cache (e.g., browser cache, reverse proxy cache, application-level cache) and how they interact.
Cache Expiration: Check the cache expiration policies—how long items stay in cache before being refreshed.
Dependency Tracking: Verify that changes are properly invalidated in all relevant caches.

#### First-Principles Solution

Identify Cache Layers:

Browser Cache: Typically, the browser cache holds static assets and might not be flushed automatically.
Reverse Proxy (e.g., NGINX): Acts as an intermediary and can cache API responses.
Application-Level Cache: Could be implemented in a microservices architecture or within the application itself.

Inspect Cache Expiration:

Ensure that cache entries are updated based on the ETag or Last-Modified headers.
Check if the cache is being refreshed at regular intervals or when data changes.

Dependency Tracking and Invalidation:

Implement a mechanism to invalidate caches upon certain events, such as updates to user profiles.
Use tools like Redis's keys * or EXPIRE commands to manage cache expiration more granularly.
Detailed Steps

Browser Cache: Clear the browser cache using developer tools (F12 -> Application tab) and see if it resolves the issue. If clearing the cache fixes the problem, then the issue is likely with the user's browser.

BASH
# In Chrome DevTools: Application > Storage > Cookies & Site Data > [Your App] > Clear site data

Reverse Proxy Cache: Check if NGINX or any other reverse proxy server has caching enabled and configured to expire cached responses at a specific interval or when the upstream service changes.

NGINX
# Example NGINX config snippet
location / {
    proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m inactive=60m;
    proxy_cache_key "$host$request_uri";
    proxy_pass http://backend/;
}

Application-Level Cache: If you're using a service like Redis or Memcached, ensure that the cache is invalidated when the user profile updates.

PYTHON
# Example Python code with Redis to invalidate cache on update
from redis import StrictRedis

def update_user_profile(user_id):
    # Update database first
    db.update_user_profile(user_id, new_data)

    # Invalidate cache
    r = StrictRedis()
    r.delete(f"user:{user_id}")

Cache Invalidation Strategy: Implement a strategy for invalidating caches in all layers when the user profile updates. This could be done via an event bus or distributed message broker like Kafka.

#### Closing Summary
and Common Mistakes

#### Common Mistakes
Forgetting to invalidate cache entries, misconfiguring cache expiration policies, not considering browser caching, and assuming a single-layered cache system.
Best Practices: Use consistent and reliable invalidation mechanisms across all layers, monitor cache performance, and use tools like distributed tracing to debug issues.

**Comparison:**

```
Cache Layer	Description
Browser	Stores static assets and can be cleared manually.
Reverse Proxy (e.g., NGINX)	Caches API responses and can expire based on headers or time.
Application-Level Cache (e.g., Redis, Memcached)	Manages cache entries for faster data retrieval.
```

**Code:**

```python
# Example Python code with Redis to invalidate cache on update
from redis import StrictRedis

def update_user_profile(user_id):
    # Update database first
    db.update_user_profile(user_id, new_data)

    # Invalidate cache
    r = StrictRedis()
    r.delete(f"user:{user_id}")
```

**Example input / output:**

```text
# Example input -> output for cache invalidation
Input: User updates profile with new data.
Output:
- Redis: DELETE user:123
- Database: Update user profile in the database
```

**Data:**

```json
{
  "cache_keys": [
    { "key": "user:123", "value": "user_data" },
    { "key": "profile:123", "value": "updated_profile" }
  ]
}
```

**Flow:**

1. User Updates Profile
2. Database Update
3. Cache Invalidation Triggered
4. Redis Invalidate Cache
5. Nginx Refresh Cache Headers

