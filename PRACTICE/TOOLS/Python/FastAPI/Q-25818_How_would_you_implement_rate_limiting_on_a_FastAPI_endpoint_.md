---
type: "practice"
difficulty: "Deep"
---

# How would you implement rate limiting on a FastAPI endpoint to prevent abuse?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Rate Limits and Backpressure](<../../../../TOOLS/Python/FastAPI/Rate_Limits_and_Backpressure.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement rate limiting on a FastAPI endpoint, we need to ensure that only a certain number of requests are allowed within a specific time window. This is crucial to prevent abuse and ensure that the service remains responsive. Let's break down the steps involved in implementing this feature from first principles.

#### Triage

The question asks us to implement rate limiting on a FastAPI endpoint. The key aspects being probed include understanding how to track request frequency, managing state across requests, and handling different levels of abuse prevention (e.g., bursts vs. sustained traffic).

#### Assumptions
We are using FastAPI as the web framework.
Rate limits will be applied based on IP addresses or user sessions.
The rate limit parameters include the number of allowed requests (limit) and the time window over which these are counted (window).
We need to handle both GET and POST requests.

#### Solution
from First Principles
State Management: We need a way to keep track of the request count per IP or session.
Rate Calculation: Calculate whether the current request has exceeded the allowed limit within the specified time window.
Concurrency Handling: Ensure that multiple requests do not interfere with each other, especially in high-concurrency scenarios.
Step-by-Step Solution
Choose a Storage Mechanism: For simple rate limiting, we can use an in-memory dictionary to track request counts per IP or session. However, for production environments, using a database (like Redis) would be more robust.
Rate Limiting Logic:
On each incoming request, check if the IP address/session has exceeded the allowed limit within the time window.
If the limit is not yet reached, update the count and allow the request to proceed.
If the limit is reached, return a 429 Too Many Requests status code or any other appropriate response.
Detailed Implementation

Let's implement rate limiting using Redis for state management. We'll use FastAPI middleware to handle incoming requests.

PYTHON
from fastapi import FastAPI, Request, HTTPException
from starlette.middleware.base import BaseHTTPMiddleware
import asyncio

app = FastAPI()

# In-memory dictionary for testing (replace with Redis in production)
request_counts = {}

class RateLimitingMiddleware(BaseHTTPMiddleware):
    def __init__(self, app, limit: int = 10, window: float = 60.0):
        super().__init__(app)
        self.limit = limit
        self.window = window

    async def dispatch(self, request: Request, call_next):
        ip_address = str(request.client.host)

        # Check if the IP address has exceeded the rate limit
        if ip_address in request_counts:
            count, last_request_time = request_counts[ip_address]
            current_time = time.time()
            if (current_time - last_request_time) < self.window:
                if count >= self.limit:
                    raise HTTPException(status_code=429, detail="Too many requests")
                else:
                    # Increment the count and update the timestamp
                    request_counts[ip_address] = (count + 1, current_time)
            else:
                # Reset the count after the window period has passed
                del request_counts[ip_address]
        else:
            # Initialize the count for a new IP address
            request_counts[ip_address] = (1, time.time())

        response = await call_next(request)

        return response

# Apply the middleware to all requests
app.middleware("http")(RateLimitingMiddleware(limit=50, window=30.0))

#### Closing Summary
and Common Mistakes/Follow-Ups

The key takeaway is that we implemented rate limiting using a custom middleware in FastAPI. This approach ensures that each IP or session can only make a certain number of requests within a specified time window.

#### Common Mistakes
Not Using an Expiring Cache: If you use an in-memory dictionary, the limits will persist indefinitely even after the client disconnects.
Ignoring Concurrent Requests: Without proper synchronization, multiple concurrent requests could interfere with each other's counts.
Overlooking Edge Cases: Ensure that bursty traffic is handled correctly (e.g., 10 requests every second for a minute vs. 600 requests over ten minutes).

#### Follow-Ups
How would you handle different types of abuse (e.g., bot detection)?
What if the rate limit needs to be dynamic based on user roles or subscription tiers?
How would you scale this solution horizontally?

**Code:**

```python
from fastapi import FastAPI, Request, HTTPException
import time

app = FastAPI()

# In-memory dictionary for testing (replace with Redis in production)
request_counts = {}

class RateLimitingMiddleware(BaseHTTPMiddleware):
    def __init__(self, app, limit: int = 10, window: float = 60.0):
        super().__init__(app)
        self.limit = limit
        self.window = window

    async def dispatch(self, request: Request, call_next):
        ip_address = str(request.client.host)

        # Check if the IP address has exceeded the rate limit
        if ip_address in request_counts:
            count, last_request_time = request_counts[ip_address]
            current_time = time.time()
            if (current_time - last_request_time) < self.window:
                if count >= self.limit:
                    raise HTTPException(status_code=429, detail="Too many requests")
                else:
                    # Increment the count and update the timestamp
                    request_counts[ip_address] = (count + 1, current_time)
            else:
                # Reset the count after the window period has passed
                del request_counts[ip_address]
        else:
            # Initialize the count for a new IP address
            request_counts[ip_address] = (1, time.time())

        response = await call_next(request)

        return response

# Apply the middleware to all requests
app.middleware("http")(RateLimitingMiddleware(limit=50, window=30.0))
```

**Flow:**

1. Yes
2. No
3. Request comes in
4. Is rate limit exceeded?
5. Rate limit reached, return 429
6. Handle request

