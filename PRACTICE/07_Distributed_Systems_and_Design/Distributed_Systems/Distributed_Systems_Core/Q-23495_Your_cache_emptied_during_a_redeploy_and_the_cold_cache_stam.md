---
type: "practice"
difficulty: "Deep"
---

# Your cache emptied during a redeploy and the cold cache stampeded the origin, which wasn't sized for it. Explain the cache-warming and origin-protection you'd operationalize so a cold cache can't crater the source of truth.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cache Invalidation Across Regions](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Cache_Invalidation_Across_Regions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a cache empties during a redeploy and causes a sudden spike in requests to the origin, it can lead to significant load and potential downtime. To mitigate this issue, we need to operationalize two key components: cache warming and origin-protection mechanisms. Cache warming ensures that critical data is pre-fetched into the cache before the service goes live, while origin protection manages the load by limiting the number of simultaneous requests from the cache.

Cache warming can be achieved through a variety of methods, such as initializing the cache with data during deployment or using an automated tool to fetch and populate the cache. Origin protection mechanisms might include rate limiting, queuing systems, or even more sophisticated traffic management techniques like circuit breakers or retries. By combining these strategies, we can ensure that a cold cache does not overwhelm the origin.

**Comparison:**

```
Method	Effectiveness
Cache Warming	High (pre-fetched data reduces initial load on origin)
Rate Limiting	Moderate (controls the number of requests but may introduce latency for frequent clients)
```

**Code:**

```python
# Example of a simple rate limiter implementation in Python using Flask and Redis
from flask import request, abort
import redis

redis_client = redis.Redis(host='localhost', port=6379, db=0)

def rate_limit(view_func):
    def wrapped_view(*args, **kwargs):
        key = f"request:{request.remote_addr}:{request.url}"
        if redis_client.incr(key) == 1:
            redis_client.expire(key, 60)  # Set the TTL to 60 seconds
        else:
            if int(redis_client.get(key)) > 5:  # Allow up to 5 requests per minute
                abort(429, description="Too Many Requests")
        return view_func(*args, **kwargs)
    return wrapped_view

@app.route('/api/endpoint')
@rate_limit
def api_endpoint():
    # Endpoint logic here
    pass
```

**Example input / output:**

```text
**Input:**
- Deployment starts, cache empties.
- First requests to the origin occur.

**Output:**
- Cache warming kicks in, pre-fetching data from the origin into the new cache instances.
- Rate limiter triggers for each client IP, allowing up to 5 requests per minute before blocking further requests until a cooldown period.
```

**Data:**

```json
{
  "redis": {
    "host": "localhost",
    "port": 6379,
    "db": 0
  },
  "request_key_template": "request:{remote_addr}:{url}"
}
```

**Flow:**

1. No
2. Yes
3. Cache Empties
4. Is Cache Warming Configured?
5. Origin Overwhelmed
6. Data Pre-Fetched
7. Requests Managed by RateLimiter
8. Implement Origin Protection

