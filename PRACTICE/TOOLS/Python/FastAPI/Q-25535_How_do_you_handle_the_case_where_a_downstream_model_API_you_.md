---
type: "practice"
difficulty: "Deep"
---

# How do you handle the case where a downstream model API you proxy through FastAPI has strict rate limits, so you queue, retry, and backpressure without dropping user requests?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Rate Limits and Backpressure](<../../../../TOOLS/Python/FastAPI/Rate_Limits_and_Backpressure.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the scenario where a downstream model API proxied through FastAPI is subject to strict rate limits, we need to implement robust queuing, retries, and backpressure mechanisms to ensure that user requests are not dropped while still adhering to the rate limit constraints. The core idea is to decouple incoming requests from the upstream API calls using an asynchronous queue system, ensuring smooth operation and reliability.

#### Triage

The question essentially asks how to design a FastAPI application to handle rate-limited downstream APIs by queuing user requests, retrying failed ones, and managing backpressure to prevent overwhelming the downstream service. The interviewer is likely probing your understanding of concurrency management, asynchronous programming, and system architecture principles in a practical context.

#### First-Principles Solution
Asynchronous Queuing: Use an asynchronous queue (like Celery) to decouple incoming requests from API calls.
Rate Limiting: Implement rate limiting at the FastAPI level using tools like starlette-ratelimiter or custom middleware.
Retry Mechanism: Set up a retry mechanism with exponential backoff for failed API calls.
Backpressure Management: Use an overflow queue to manage too many incoming requests during peak times.
Detailed Solution
Step 1: Asynchronous Queuing

We'll use Celery, a distributed task queue system, to handle the queuing of tasks asynchronously. This allows us to process user requests in parallel while ensuring that each request is handled once and only once.

PYTHON
# Setup Celery
from celery import Celery

app = Celery('tasks', broker='redis://localhost:6379/0')

@app.task
def call_downstream_api(input_data):
    # Call the downstream API here
    pass
Step 2: Rate Limiting

FastAPI provides built-in support for rate limiting through its dependencies mechanism. We can use this to limit the number of requests per second or minute.

PYTHON
from fastapi import FastAPI, Depends
from fastapi_limiter.depends import RateLimiter

app = FastAPI()

@app.get("/")
async def read_root(dependency=Depends(RateLimiter(times=50, minutes=1))):
    return {"message": "Hello World"}
Step 3: Retry Mechanism

For the retry mechanism, we'll use an exponential backoff strategy with a maximum number of retries. This helps in managing transient failures without overwhelming the downstream service.

PYTHON
from fastapi import HTTPException

async def call_downstream_api_with_retries(input_data):
    for attempt in range(5):  # Max 5 attempts
        try:
            result = await call_downstream_api.delay(input_data)
            return result.get()  # Get the result from Celery task
        except Exception as e:
            print(f"Attempt {attempt + 1} failed: {e}")
            await asyncio.sleep(2 ** (attempt + 1))  # Exponential backoff

@app.post("/predict")
async def predict(input_data):
    try:
        result = await call_downstream_api_with_retries(input_data)
        return {"prediction": result}
    except Exception as e:
        raise HTTPException(status_code=503, detail=str(e))
Step 4: Backpressure Management

To manage backpressure, we can use an overflow queue. When the rate of incoming requests exceeds a certain threshold, excess requests are queued and processed later.

PYTHON
from collections import deque

class OverflowQueue:
    def __init__(self, max_size):
        self.queue = deque(maxlen=max_size)

@app.post("/predict")
async def predict(input_data):
    if len(overflow_queue) > 100:  # Example threshold
        await asyncio.sleep(5)  # Sleep for a bit to cool down

    try:
        result = await call_downstream_api_with_retries(input_data)
        return {"prediction": result}
    except Exception as e:
        raise HTTPException(status_code=503, detail=str(e))

#### Closing Summary

By implementing an asynchronous queue system with rate limiting, retry mechanisms, and backpressure management, we can ensure that our FastAPI service handles downstream API rate limits gracefully without dropping user requests. This approach not only adheres to the rate limit constraints but also improves overall system reliability and performance.

#### Common Mistakes/Follow-Ups
Mistake 1: Not implementing proper retry mechanisms can lead to data loss or inconsistent states.
Mistake 2: Overlooking overflow queue management can result in overwhelming the downstream service during peak times.

#### Follow-Up
Discussing how you would monitor and alert on the health of your system, as well as how you would adjust rate limits dynamically based on observed load.

**Comparison:**

```
Input Data	Predicted Values
[1, 2, 3]	[4, 5, 6]
```

**Code:**

```python
from celery import Celery
from fastapi import FastAPI, Depends
from starlette_limiter.depends import RateLimiter
import asyncio

app = FastAPI()

@app.get("/")
async def read_root(dependency=Depends(RateLimiter(times=50, minutes=1))):
    return {"message": "Hello World"}

@app.post("/predict")
async def predict(input_data):
    overflow_queue = OverflowQueue(max_size=100)

    if len(overflow_queue.queue) > 100:
        await asyncio.sleep(5)

    try:
        result = await call_downstream_api_with_retries(input_data)
        return {"prediction": result}
    except Exception as e:
        raise HTTPException(status_code=503, detail=str(e))

@app.task
def call_downstream_api(input_data):
    # Call the downstream API here
    pass

async def call_downstream_api_with_retries(input_data):
    for attempt in range(5):
        try:
            result = await call_downstream_api.delay(input_data)
            return result.get()
        except Exception as e:
            print(f"Attempt {attempt + 1} failed: {e}")
            await asyncio.sleep(2 ** (attempt + 1))
```

**Example input / output:**

```text
# Example input -> output
input_data = {"data": [1, 2, 3]}
result = await call_downstream_api_with_retries(input_data)
print(result)  # Output prediction from the downstream API
```

**Data:**

```json
{
    "input": {
        "data": [1, 2, 3]
    },
    "output": {
        "prediction": [4, 5, 6]
    }
}
```

**Flow:**

1. User Request
2. Rate Limiter
3. Async Queue (Celery)
4. Call Downstream API withRetries
5. Return Result to User or Retry

