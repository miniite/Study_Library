---
type: "practice"
difficulty: "Intermediate"
---

# How would you run a long LLM job in the background with BackgroundTasks and let the client poll for the result in FastAPI?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Long-running Jobs and Async Patterns](<../../../../TOOLS/Python/FastAPI/Long-running_Jobs_and_Async_Patterns.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To run a long LLM (Large Language Model) job in the background using BackgroundTasks in FastAPI, we need to ensure that our application can handle asynchronous tasks efficiently and provide a mechanism for clients to check on the job status without blocking. This involves setting up an async task queue and implementing a polling endpoint.

Firstly, let's break down what the question is asking:

Asynchronous Task Execution: We need to run the LLM job in the background without blocking the FastAPI server.
Client Polling: The client should be able to periodically check on the status of their job and receive updates.

Now, we will reason through this step-by-step:

Step 1: Setting Up Background Tasks

In FastAPI, BackgroundTasks are a built-in feature that allows us to run tasks in the background. We can use these to handle long-running processes like LLM jobs.

PYTHON
from fastapi import FastAPI, BackgroundTasks

app = FastAPI()

@app.post("/start_llm_job/")
async def start_llm_job(background_tasks: BackgroundTasks):
    # Start the long running job here
    background_tasks.add_task(run_llm_job)

    return {"status": "Job started"}

def run_llm_job():
    # Simulate a long-running LLM job
    import time
    time.sleep(10)  # Simulating delay
    print("LLM Job completed")
Step 2: Implementing Client Polling

The client needs to periodically check on the status of their job. We can achieve this by setting up a polling endpoint that returns the current status.

PYTHON
@app.get("/check_llm_job_status/")
async def check_llm_job_status():
    # Here we assume there's some backend system tracking jobs
    return {"status": "completed"}  # Replace with actual status checking logic

Step 3: Integration and Testing

To ensure everything works as expected, we need to test both the job initiation and polling functionality.

Sample Data:
PYTHON
# Sample data for testing purposes
from fastapi.testclient import TestClient

client = TestClient(app)

# Start the LLM Job
response_start = client.post("/start_llm_job/")
print(response_start.json())  # Should return {"status": "Job started"}

# Poll for the status of the job after a delay to simulate real-time checks
import time
time.sleep(15)  # Wait for more than 10 seconds

response_status = client.get("/check_llm_job_status/")
print(response_status.json())  # Should return {"status": "completed"}

#### Closing Summary
and Common Mistakes/Follow-Ups:
Summary: By using BackgroundTasks in FastAPI, we can run long LLM jobs asynchronously. Clients can periodically poll a specific endpoint to check the status of their job without blocking.

#### Common Mistakes
Not using asyncio properly: Ensure all tasks are marked as async.
Blocking within FastAPI handlers: Avoid blocking operations in BackgroundTasks or request handlers.
Ignoring concurrency issues: Properly handle race conditions and ensure thread safety if using multiple workers.

This approach ensures that the server remains responsive while long-running jobs complete in the background, providing a seamless user experience through client polling.

**Code:**

```python
from fastapi import FastAPI, BackgroundTasks

app = FastAPI()

@app.post("/start_llm_job/")
async def start_llm_job(background_tasks: BackgroundTasks):
    background_tasks.add_task(run_llm_job)
    return {"status": "Job started"}

def run_llm_job():
    import time
    time.sleep(10)  # Simulating delay
    print("LLM Job completed")

@app.get("/check_llm_job_status/")
async def check_llm_job_status():
    # Assume this returns the job status from a backend system
    return {"status": "completed"}
```

**Example input / output:**

```text
# Response from starting the LLM job
{"status": "Job started"}

# Response after polling for status
{"status": "completed"}
```

**Data:**

```python
from fastapi.testclient import TestClient

client = TestClient(app)

response_start = client.post("/start_llm_job/")
print(response_start.json())

import time
time.sleep(15)
response_status = client.get("/check_llm_job_status/")
print(response_status.json())
```

**Flow:**

1. Start LLM Job
2. BackgroundTasks.run(run_llm_job)
3. Poll Status
4. Check Job Status
5. Job Completes

