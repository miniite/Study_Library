---
type: "practice"
difficulty: "Deep"
---

# Your asyncio-based Python collector that polls hundreds of endpoints degrades when a few endpoints hang. How would you enforce per-request timeouts and concurrency limits so slow hosts don't starve the event loop?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Asyncio Concurrency](<../../../../TOOLS/Python/Python_Language/Asyncio_Concurrency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of hanging endpoints starving the asyncio event loop in a Python application, we need to implement two key mechanisms: per-request timeouts and concurrency limits. This ensures that slow endpoints do not monopolize resources, allowing other tasks to proceed.

#### Triage
and Assumptions
What is being asked: The interviewer wants you to design a robust asynchronous system that can handle thousands of concurrent requests without being blocked by slow or hanging endpoints.
Key concepts:
asyncio: Python's built-in library for writing concurrent code using coroutines, multiplexing I/O access over sockets and other resources, and parallelism with threads.
Per-request timeouts: Ensuring that each request does not take longer than a specified time to complete.
Concurrency limits: Controlling the maximum number of simultaneous requests to avoid overwhelming the system.

#### First-Principles Solution
Step 1: Implement Per-Request Timeouts

To enforce per-request timeouts, we can use asyncio.wait_for or asyncio.create_task with a timeout. The idea is to create tasks and wait for them not to exceed the timeout duration.

PYTHON
import asyncio

async def fetch_url(url):
    try:
        response = await asyncio.wait_for(make_request(url), timeout=5)  # Timeout set to 5 seconds
        return response
    except asyncio.TimeoutError:
        print(f"Timeout reached for {url}")
        return None

def make_request(url):
    # Simulate a network request
    import aiohttp
    async def inner():
        async with aiohttp.ClientSession() as session:
            async with session.get(url) as resp:
                return await resp.text()
    return inner
Step 2: Enforce Concurrency Limits

Concurrency limits can be enforced using a semaphore to control the number of active tasks. This ensures that only a certain number of requests are made at any given time.

PYTHON
import asyncio
from collections import deque

class AsyncCollector:
    def __init__(self, max_concurrent_tasks=10):
        self.semaphore = asyncio.Semaphore(max_concurrent_tasks)
        self.tasks_queue = deque()

    async def fetch_urls(self, urls):
        for url in urls:
            await self.fetch_url(url)

    async def fetch_url(self, url):
        async with self.semaphore:
            task = asyncio.create_task(fetch_url_with_timeout(url))
            self.tasks_queue.append(task)
            await task

async def fetch_url_with_timeout(url):
    # Same as the fetch_url function from above

#### Closing Summary
and Common Mistakes/Follow-Ups

To summarize, we implemented per-request timeouts using asyncio.wait_for to ensure that slow endpoints do not block the event loop. Additionally, concurrency limits are enforced using a semaphore to manage the number of simultaneous requests.

#### Common mistakes
include:

Not handling asyncio.TimeoutError properly.
Forgetting to use a semaphore for limiting concurrent tasks.
Incorrectly managing task queues and semaphores, leading to resource leaks or starvation issues.

For follow-ups, you might be asked about optimizing performance further, dealing with retries on timeout, or integrating more advanced error handling mechanisms. These are good opportunities to demonstrate your deeper understanding of the subject matter.

