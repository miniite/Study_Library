---
type: "practice"
difficulty: "Deep"
---

# How would you implement graceful degradation in FastAPI so that when Redis is down the service serves stale or reduced functionality instead of erroring on every request?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Service Architecture and Versioning](<../../../../TOOLS/Python/FastAPI/Service_Architecture_and_Versioning.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary: When implementing fastAPI, gracefully degrading to a degraded mode during Redis downtime ensures continuous service availability by serving stale data or reduced functionality. This approach leverages middleware and environment checks at various levels of the FastAPI application.

#### Triaging the Question
The core of this question is about ensuring high availability in a distributed system where external dependencies like Redis might be unreliable. We need to implement robust error handling that can degrade gracefully, providing fallback strategies when services are unavailable. This involves checking the Redis connection status and handling errors appropriately within FastAPI's request/response cycle.

#### First-Principles Solution

Environment Check Middleware: Implement middleware to check if Redis is available before executing requests.
Request Handling with Fallbacks: Use context managers or custom decorators to manage fallback logic based on the availability of Redis.
Stale Data Handling: Ensure that stale data can be served when Redis is down, possibly from a local cache or even by falling back to database reads.

Let's break this down step-by-step:

Step 1: Environment Check Middleware

We'll use middleware to check the status of our Redis connection at the start of each request. If Redis is unavailable, we can mark the request as degraded and proceed with fallback logic.

PYTHON
import fastapi
from starlette.middleware.base import BaseHTTPMiddleware
from redis.client import StrictRedis

app = fastapi.FastAPI()

class RedisConnectionMiddleware(BaseHTTPMiddleware):
    def __init__(self, app, client: StrictRedis):
        super().__init__(app)
        self.redis_client = client

    async def dispatch(self, request: fastapi.Request, call_next):
        if not await self.check_redis_connection():
            # Mark the request as degraded
            request.state.degraded_mode = True
        response = await call_next(request)
        return response

    async def check_redis_connection(self) -> bool:
        try:
            status = await self.redis_client.ping()
            return status
        except Exception:
            return False
Step 2: Request Handling with Fallbacks

Next, we need to handle requests in a way that if Redis is down, the application can serve stale data or reduced functionality. We'll use context managers and custom decorators for this purpose.

PYTHON
from fastapi import Depends, HTTPException

def get_degraded_mode(request: fastapi.Request) -> bool:
    return request.state.degraded_mode

@app.get("/data")
async def read_data(degraded_mode: bool = Depends(get_degraded_mode)):
    if degraded_mode:
        # Serve stale data or reduced functionality
        data = get_stale_data_from_local_cache()
        return {"data": data}
    else:
        # Normal request handling with Redis
        data = await fetch_data_from_redis()
        return {"data": data}

def get_stale_data_from_local_cache() -> str:
    # Example implementation to serve stale data from local cache
    return "stale data"
Step 3: Stale Data Handling

In this example, we demonstrate how to serve stale data when Redis is down. In practice, you might have a more sophisticated strategy involving caching mechanisms or database fallbacks.

PYTHON
def get_stale_data_from_local_cache() -> str:
    # Example implementation to serve stale data from local cache
    return "stale data"

#### Closing Summary
and Common Mistakes

By implementing the above strategies, we ensure that our FastAPI application can handle Redis downtime gracefully. However, there are a few common pitfalls:

Inconsistent State Management: Ensure consistent state management across all request handlers.

#### Performance Considerations
Fallback logic should be optimized to avoid unnecessary database queries or excessive local cache hits.
Error Handling Overload: Avoid overloading error handling; it’s crucial to handle errors in a way that minimizes performance impact.

**Code:**

```python
import fastapi
from starlette.middleware.base import BaseHTTPMiddleware
from redis.client import StrictRedis

app = fastapi.FastAPI()

class RedisConnectionMiddleware(BaseHTTPMiddleware):
    def __init__(self, app, client: StrictRedis):
        super().__init__(app)
        self.redis_client = client

    async def dispatch(self, request: fastapi.Request, call_next):
        if not await self.check_redis_connection():
            request.state.degraded_mode = True
        response = await call_next(request)
        return response

    async def check_redis_connection(self) -> bool:
        try:
            status = await self.redis_client.ping()
            return status
        except Exception:
            return False

@app.get("/data")
async def read_data(degraded_mode: bool = Depends(get_degraded_mode)):
    if degraded_mode:
        data = get_stale_data_from_local_cache()
        return {"data": data}
    else:
        data = await fetch_data_from_redis()
        return {"data": data}

def get_degraded_mode(request: fastapi.Request) -> bool:
    return request.state.degraded_mode

def get_stale_data_from_local_cache() -> str:
    return "stale data"
```

**Example input / output:**

```text
# Example Input-Output
Request to /data when Redis is down:
Input: GET /data HTTP/1.1

Response:
HTTP/1.1 200 OK
Content-Type: application/json

{
    "data": "stale data"
}
```

**Flow:**

1. Yes
2. No
3. Request
4. Redis Available?
5. Fetch Data from Redis
6. Get Stale Data
7. Return Data
8. Response Sent

