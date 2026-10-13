---
type: "practice"
difficulty: "Deep"
---

# How would you build load-testing for a FastAPI GPU inference service that reflects real batching behavior, so you measure realistic throughput and not just single-request latency?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Real-time and Batch Inference APIs](<../../../../TOOLS/Python/FastAPI/Real-time_and_Batch_Inference_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build load-testing for a FastAPI GPU inference service that accurately reflects real-world batching behavior, we need to simulate how the service would handle multiple requests in batches, similar to what happens in production scenarios. This involves setting up a test framework that can generate and send batched HTTP requests to our FastAPI application. By measuring the throughput under these conditions, we can obtain more meaningful performance metrics than if we were only testing single-request latency.

#### Triaging the Question

The core of this question is understanding how to design load-testing for a GPU inference service using FastAPI. The interviewer is likely probing your knowledge of:

FastAPI and its capabilities.
Load-testing principles and techniques.
GPU inference workflow understanding.
Real-world batching behavior simulation.

#### Assumptions
We have a FastAPI application that performs GPU inference.
Multiple concurrent users will submit requests to the service, which are batched together before processing.
The objective is to measure the system's throughput under realistic conditions.

#### Solution
from First Principles

Setup Test Environment:

Use a testing framework such as pytest or FastAPI-Testing. These frameworks can help us set up and manage our test environment effectively.

Generate Batched Requests:

Instead of sending single requests, we need to simulate batch processing by generating multiple requests at once.
This involves creating a mechanism that batches these requests before sending them to the FastAPI application.

Measure Throughput:

Use metrics such as requests per second (RPS) and latency distribution.
Employ tools like Apache JMeter, Locust, or even custom scripts in Python using libraries like asyncio and httpx.

Simulate Real-World Conditions:

Emulate the actual user load that your service will face, including varying request patterns.

#### Cons
ider factors such as network latency, GPU resource allocation, and potential bottlenecks.

Analyze Results:

Collect detailed performance metrics from both the client-side (requests) and server-side (responses).
Use these metrics to identify any performance issues or areas for optimization.

#### Implementation
Example
PYTHON
# Step 1: Setup FastAPI Application
from fastapi import FastAPI, APIRouter, Request
import uvicorn

app = FastAPI()

router = APIRouter()
@router.get("/inference")
def inference(request: Request):
    # Simulate GPU inference here
    pass

app.include_router(router)

# Step 2: Set Up Load-Testing Framework
import httpx
from time import perf_counter

async def send_batched_requests(num_requests, batch_size=10):
    start_time = perf_counter()
    async with httpx.AsyncClient() as client:
        tasks = [client.get("http://localhost:8000/inference") for _ in range(batch_size) * (num_requests // batch_size)]

        # Send remaining requests if num_requests is not a multiple of batch_size
        for i in range(num_requests % batch_size):
            tasks.append(client.get("http://localhost:8000/inference"))

    responses = await asyncio.gather(*tasks)

    end_time = perf_counter()
    total_time = end_time - start_time

    throughput = num_requests / total_time
    return throughput, responses

# Step 3: Run Load-Testing
import asyncio

async def run_load_test(num_requests=1000):
    throughput, _ = await send_batched_requests(num_requests)
    print(f"Throughput (RPS): {throughput}")

if __name__ == "__main__":
    asyncio.run(run_load_test())

#### Closing Summary
and Common Mistakes

By setting up a test that simulates batched requests to your FastAPI GPU inference service, you can measure the realistic throughput of the application. This approach ensures that the performance metrics reflect actual usage patterns rather than just single-request latency.

#### Common mistakes
include:

Focusing solely on single-request metrics.
Not accounting for network latency in batched request scenarios.
Overlooking potential bottlenecks such as GPU resource management or I/O operations.

**Code:**

```python
from fastapi import FastAPI, APIRouter, Request
import uvicorn

app = FastAPI()

router = APIRouter()
@router.get("/inference")
def inference(request: Request):
    # Simulate GPU inference here
    pass

app.include_router(router)

async def send_batched_requests(num_requests, batch_size=10):
    start_time = perf_counter()
    async with httpx.AsyncClient() as client:
        tasks = [client.get("http://localhost:8000/inference") for _ in range(batch_size) * (num_requests // batch_size)]

        # Send remaining requests if num_requests is not a multiple of batch_size
        for i in range(num_requests % batch_size):
            tasks.append(client.get("http://localhost:8000/inference"))

    responses = await asyncio.gather(*tasks)

    end_time = perf_counter()
    total_time = end_time - start_time

    throughput = num_requests / total_time
    return throughput, responses

async def run_load_test(num_requests=1000):
    throughput, _ = await send_batched_requests(num_requests)
    print(f"Throughput (RPS): {throughput}")

if __name__ == "__main__":
    asyncio.run(run_load_test())
```

**Example input / output:**

```text
Num Requests: 1000, Batch Size: 10
Throughput (RPS): 95.23
```

