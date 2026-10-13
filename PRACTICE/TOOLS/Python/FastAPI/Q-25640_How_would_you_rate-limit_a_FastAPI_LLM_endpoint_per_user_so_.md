---
type: "practice"
difficulty: "Intermediate"
---

# How would you rate-limit a FastAPI LLM endpoint per user so one client cannot burn through your API budget?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Rate Limiting and Canary Releases](<../../../../TOOLS/Python/FastAPI/Rate_Limiting_and_Canary_Releases.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To rate-limit an AI Engineer 2/3 LLM endpoint in FastAPI, we need to ensure that each user does not overwhelm our service and exhaust our API budget. The key idea is to track API calls per user within a certain time window and enforce limits accordingly.

#### Triaging the Question

The question asks for implementing rate limiting on an AI model's endpoint served by FastAPI. We're looking at ensuring that individual clients do not make too many requests in a given timeframe, which could lead to overutilization or abuse of our API budget. The main challenges include:

Identifying users (e.g., via tokens).
Tracking the number of requests per user.
Enforcing rate limits based on predefined thresholds.

#### First Principles Solution
Identify Users: We need a way to uniquely identify each client making requests, such as by API token or session ID.
Track Requests: Maintain a count of requests made within a sliding window for each identified user.
Enforce Rate Limits: Apply rules based on the tracked request counts.
Step-by-Step Solution
1. Setting Up Dependencies

First, ensure FastAPI is set up with necessary dependencies:

PYTHON
pip install fastapi uvicorn
2. Create a Rate-Limiting Middleware

Create a middleware to manage rate limiting for each user.

PYTHON
from fastapi import APIRouter, Depends, HTTPException
from typing import Dict, Any
from starlette.requests import Request

class RateLimiter:
    def __init__(self):
        self.clients = {}

    async def __call__(self, request: Request) -> None:
        if "Authorization" in request.headers and "Bearer" in request.headers["Authorization"]:
            token = request.headers["Authorization"].split(" ")[1]
            client_info = await self.get_client_info(token)
            if not client_info:
                raise HTTPException(status_code=401, detail="Unauthorized")
            # Check if the client has exceeded the rate limit
            if "request_count" in client_info and "last_request_time" in client_info:
                current_time = request.timestamp  # Assuming FastAPI provides a way to get timestamp
                time_since_last_request = current_time - client_info["last_request_time"]
                if time_since_last_request < 60:  # Assuming 1 minute window
                    requests_in_window = client_info["request_count"]
                    if requests_in_window > 5:  # Example limit of 5 requests per user in a minute
                        raise HTTPException(status_code=429, detail="Too Many Requests")
        else:
            raise HTTPException(status_code=401, detail="Unauthorized")

    async def get_client_info(self, token):
        if token not in self.clients:
            # Simulate fetching client info from database or cache
            await asyncio.sleep(0.1)  # Simulating delay for fetching data
            return {"request_count": 0, "last_request_time": request.timestamp}
        return self.clients[token]

router = APIRouter()
rate_limiter = RateLimiter()

@router.get("/api/llm")
async def get_llm_data(request: Request):
    await rate_limiter(request)
    # Your LLM logic here
    return {"message": "LLM response"}
3. Testing the Middleware

Ensure that requests from a single client are being tracked and rate-limited.

PYTHON
import uvicorn

if __name__ == "__main__":
    uvicorn.run(router, host="0.0.0.0", port=8000)

You can test this by sending multiple requests with the same token within 1 minute.

#### Closing Summary
and Common Mistakes/Follow-Ups

This approach ensures that each user does not exceed a certain number of API calls per unit time, preventing abuse. Common mistakes include:

Not properly identifying users.
Failing to enforce rate limits strictly.
Using incorrect timestamps or logic.

#### Follow-up
questions might involve optimizing for performance, handling distributed systems, and ensuring compatibility with different authentication mechanisms.

**Code:**

```python
from fastapi import APIRouter, Depends, HTTPException
from typing import Dict, Any
from starlette.requests import Request

class RateLimiter:
    def __init__(self):
        self.clients = {}

    async def __call__(self, request: Request) -> None:
        if "Authorization" in request.headers and "Bearer" in request.headers["Authorization"]:
            token = request.headers["Authorization"].split(" ")[1]
            client_info = await self.get_client_info(token)
            if not client_info:
                raise HTTPException(status_code=401, detail="Unauthorized")
            current_time = request.timestamp  # Assuming FastAPI provides a way to get timestamp
            time_since_last_request = current_time - client_info["last_request_time"]
            if time_since_last_request < 60:  # Assuming 1 minute window
                requests_in_window = client_info["request_count"]
                if requests_in_window > 5:  # Example limit of 5 requests per user in a minute
                    raise HTTPException(status_code=429, detail="Too Many Requests")
        else:
            raise HTTPException(status_code=401, detail="Unauthorized")

    async def get_client_info(self, token):
        if token not in self.clients:
            # Simulate fetching client info from database or cache
            await asyncio.sleep(0.1)  # Simulating delay for fetching data
            return {"request_count": 0, "last_request_time": request.timestamp}
        return self.clients[token]

router = APIRouter()
rate_limiter = RateLimiter()

@router.get("/api/llm")
async def get_llm_data(request: Request):
    await rate_limiter(request)
    # Your LLM logic here
    return {"message": "LLM response"}
```

