---
type: "practice"
difficulty: "Intermediate"
---

# Your FastAPI service calls three downstream APIs sequentially and p99 is the sum of all three; how would you restructure it and what failure-isolation concerns come with parallelizing them?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Rate Limits and Backpressure](<../../../../TOOLS/Python/FastAPI/Rate_Limits_and_Backpressure.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview
In this scenario, we have a FastAPI service that currently calls three downstream APIs sequentially, where the 99th percentile (p99) latency of the entire operation is the sum of their individual p99 latencies. To improve performance, we want to parallelize these calls. However, this introduces new failure-isolation concerns.

#### Triage
The question asks us to restructure our FastAPI service for parallel execution while considering potential issues that might arise from doing so. We need to ensure that the new design maintains or even improves overall latency and reliability, especially in terms of error handling and isolation between services.

#### First-Principles Solution

Current Scenario Analysis:

The current setup calls three APIs sequentially.
P99 is the sum of their individual p99 latencies.
This indicates that the overall performance is limited by the slowest API response.

Goal:

Parallelize the calls to reduce the total latency as much as possible.
Ensure that failures in one API do not propagate to others, maintaining service stability.

Parallel Execution Strategy:

Use asynchronous HTTP requests (requests or httpx) within FastAPI.
Employ a concurrency control mechanism like concurrent.futures.ThreadPoolExecutor for managing parallelism.

#### Implementation
Steps:

Define an endpoint in FastAPI to trigger the three calls.
Use asynchronous context managers to handle the API calls concurrently.
Collect and aggregate results from all APIs once they complete.

Error Handling and Isolation:

Implement try-except blocks for individual API requests to catch errors without crashing the entire request chain.
Use timeouts on each concurrent task to prevent any single task from causing a long delay or hang.
Use exception handling to gracefully handle failures, possibly retrying failed tasks.

Code Example:

PYTHON
from fastapi import FastAPI, Request
import asyncio
import httpx

app = FastAPI()

async def fetch_data(url):
    async with httpx.AsyncClient() as client:
        response = await client.get(url)
    return response.json()

@app.post("/parallel-calls")
async def parallel_calls(request: Request):
    urls = ["https://api1.com", "https://api2.com", "https://api3.com"]

    tasks = [fetch_data(url) for url in urls]
    results = await asyncio.gather(*tasks, return_exceptions=True)

    # Handle and log any exceptions
    for i, result in enumerate(results):
        if isinstance(result, Exception):
            print(f"Error fetching data from {urls[i]}: {result}")

    # Process the aggregated results as needed
    # Example: sum up some metrics or return combined data

    return {"status": "success", "results": results}

Testing and Validation:

Test with varying loads to ensure parallelization does not introduce unexpected latency.
Validate that all error handling works as expected without disrupting the service.

Potential Issues:

Thread safety: Ensure no shared state is accessed by multiple threads simultaneously.
Resource Contention: Be mindful of system resources (CPU, memory) when running many concurrent tasks.
Latency Variability: Some APIs might still take longer than others, causing p99 latency to be higher.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

The primary goal is to improve overall response time by parallelizing the API calls. However, it’s crucial to handle errors and failures gracefully to maintain service stability.
Potential pitfalls include not handling exceptions properly, running out of thread pool capacity, or failing to manage concurrency control effectively.

#### Common mistakes

Not setting timeouts on concurrent tasks can lead to indefinite hangs.
Ignoring error propagation in asynchronous functions can cause unexpected crashes.
Failing to log and analyze errors from downstream APIs can make troubleshooting harder.

#### Follow-ups
could include discussing load testing strategies, retry mechanisms for failed calls, or implementing a more sophisticated concurrency control mechanism if the service experiences high traffic.

**Code:**

```python
from fastapi import FastAPI, Request
import asyncio
import httpx

app = FastAPI()

async def fetch_data(url):
    async with httpx.AsyncClient() as client:
        response = await client.get(url)
    return response.json()

@app.post("/parallel-calls")
async def parallel_calls(request: Request):
    urls = ["https://api1.com", "https://api2.com", "https://api3.com"]

    tasks = [fetch_data(url) for url in urls]
    results = await asyncio.gather(*tasks, return_exceptions=True)

    # Handle and log any exceptions
    for i, result in enumerate(results):
        if isinstance(result, Exception):
            print(f"Error fetching data from {urls[i]}: {result}")

    # Process the aggregated results as needed
    # Example: sum up some metrics or return combined data

    return {"status": "success", "results": results}
```

**Flow:**

1. Yes
2. No
3. No
4. Yes
5. Start
6. Parallel API Calls
7. Is Result Available?
8. Aggregate Results and Return
9. Wait for Next Task to Complete
10. Check Exceptions
11. Exception Occurred?
12. Log Exception, ContinueProcessing Other Tasks
13. Aggregated Results
14. Return Results

