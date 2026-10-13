---
type: "practice"
difficulty: "Intermediate"
---

# How do you decide whether an LLM-calling FastAPI endpoint benefits from being async, given that the bottleneck is a multi-second network call to the provider rather than local CPU?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Async vs Sync Endpoints](<../../../../TOOLS/Python/FastAPI/Async_vs_Sync_Endpoints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding whether an LLM-calling FastAPI endpoint should be asynchronous (async), especially when the primary bottleneck is a multi-second network call, it's crucial to consider both performance and correctness. The TL;DR is that async is beneficial if you can achieve significant concurrent processing without introducing race conditions or complexity.

The core question here is: How does asynchrony impact the overall efficiency of your FastAPI endpoint? In this context, we are probing how well you understand asynchronous programming in Python, particularly with FastAPI. The key assumptions include understanding the nature of I/O-bound tasks like network calls and recognizing that async can reduce wait times by allowing other tasks to run concurrently.

Let's start from first principles:

Understanding Asynchronous Programming: In Python, async functions (coroutines) are defined using async def. These functions return a coroutine object which, when awaited, runs until it hits the next awaitable or completes. This allows non-blocking execution of I/O-bound tasks.

Bottleneck Analysis: Given that the bottleneck is primarily the multi-second network call to an LLM provider, this is an ideal scenario for asynchrony because the task is predominantly waiting on external I/O (network response).

Concurrency Benefits: Async allows you to run multiple coroutines concurrently without blocking the event loop. For instance, if your endpoint receives a batch of requests and each request involves a network call, async can process these requests in parallel, significantly improving throughput.

#### Implementation
Details:

Define your FastAPI route with @app.post("/endpoint") as an async def function.
Use the aiohttp library for making asynchronous HTTP calls within your coroutine.

#### Example
code:
PYTHON
from fastapi import FastAPI, Depends
from pydantic import BaseModel
import aiohttp

app = FastAPI()

class RequestModel(BaseModel):
    prompt: str

async def fetch_data(prompt: str) -> dict:
    url = "https://api.provider.com/llm"
    headers = {"Authorization": "Bearer YOUR_API_KEY"}
    async with aiohttp.ClientSession() as session:
        async with session.post(url, json={"prompt": prompt}, headers=headers) as resp:
            return await resp.json()

@app.post("/endpoint")
async def endpoint(request: RequestModel):
    response_data = await fetch_data(prompt=request.prompt)
    # Process and format the response data
    return {"result": response_data}

Testing and Validation: Ensure your implementation handles edge cases, such as network errors or timeouts, gracefully.

Performance Metrics: Monitor performance using tools like asyncio’s event loop statistics to understand how well your async code is utilizing the available resources.

Synchronization Considerations: Be cautious with shared state and synchronization primitives (like locks) to avoid race conditions.

In conclusion, given a primary bottleneck of multi-second network calls, an asynchronous FastAPI endpoint can significantly improve performance by enabling concurrent processing. However, it’s important to ensure that the added complexity doesn’t outweigh the benefits, especially in terms of debugging and maintenance.

**Code:**

```python
from fastapi import FastAPI, Depends
from pydantic import BaseModel
import aiohttp

app = FastAPI()

class RequestModel(BaseModel):
    prompt: str

async def fetch_data(prompt: str) -> dict:
    url = "https://api.provider.com/llm"
    headers = {"Authorization": "Bearer YOUR_API_KEY"}
    async with aiohttp.ClientSession() as session:
        async with session.post(url, json={"prompt": prompt}, headers=headers) as resp:
            return await resp.json()

@app.post("/endpoint")
async def endpoint(request: RequestModel):
    response_data = await fetch_data(prompt=request.prompt)
    # Process and format the response data
    return {"result": response_data}
```

