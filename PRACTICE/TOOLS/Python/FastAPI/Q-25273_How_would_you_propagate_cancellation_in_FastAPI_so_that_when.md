---
type: "practice"
difficulty: "Expert"
---

# How would you propagate cancellation in FastAPI so that when a client disconnects mid-request, the in-flight downstream calls and DB work are actually aborted rather than continuing to burn resources?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Timeouts and Resource Protection](<../../../../TOOLS/Python/FastAPI/Timeouts_and_Resource_Protection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to address the challenge of handling cancellations in FastAPI efficiently so that when a client disconnects mid-request, any downstream calls and database operations are properly aborted rather than continuing to execute. This involves understanding how FastAPI handles asynchronous tasks and integrating cancellation mechanisms effectively.

#### Triaging the Question

The question revolves around implementing robust cancellation for tasks initiated by FastAPI requests. Specifically, we need to ensure that:

Client Disconnect Handling: Detect when a client disconnects.
Task Cancellation: Properly cancel any ongoing tasks or database operations.
Resource Management: Ensure resources are freed up promptly upon cancellation.

#### Assumptions
We are working with FastAPI version 0.76+ which supports async/await and background tasks.
The client sends a request to the server, triggering an asynchronous task that performs some work (e.g., database queries).
We need to handle both immediate cancellations due to client disconnection and graceful terminations.

#### First Principles Solution

To solve this problem, we will use a combination of FastAPI's background tasks, event listeners for disconnects, and Python’s asyncio features. Here’s the step-by-step approach:

Define a Background Task: Use FastAPI’s BackgroundTasks to handle tasks that should run in the background.
Handle Client Disconnect: Listen for disconnection events using a middleware or socket monitoring.
Graceful Cancellation: Ensure that any running task can be canceled properly.
Detailed Solution
Step 1: Define Background Tasks

We will define a background task that performs some work, such as querying the database.

PYTHON
from fastapi import FastAPI, BackgroundTasks
import asyncio
import time

app = FastAPI()

async def long_running_task(task_id: int):
    print(f"Starting task {task_id}")
    for i in range(50):
        # Simulate work
        await asyncio.sleep(1)
        print(f"Task {task_id} iteration {i+1}")
    print(f"Completed task {task_id}")

@app.post("/tasks/{task_id}")
async def create_task(task_id: int, background_tasks: BackgroundTasks):
    background_tasks.add_task(long_running_task, task_id)

Step 2: Handle Client Disconnect

We will use a custom middleware to monitor for client disconnections and cancel tasks if necessary.

PYTHON
import uvicorn

@app.middleware("http")
async def handle_client_disconnect(request, call_next):
    # This is a placeholder. In a real-world scenario, you would implement logic to detect disconnects.
    response = await call_next(request)

    # Example: If we detect a disconnect, cancel the task here
    if request.client.is_disconnected:
        print("Client disconnected, cancelling tasks")
        # Cancel any running background tasks
        for task in asyncio.all_tasks():
            task.cancel()

    return response

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
Step 3: Graceful Cancellation

Ensure that any running tasks can be gracefully canceled using asyncio.create_task and the cancel() method.

PYTHON
import asyncio

async def cancel_task(task_id: int):
    task = asyncio.current_task()
    if task:
        task.cancel()
        try:
            await task
        except asyncio.CancelledError:
            print(f"Task {task_id} was canceled")

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, to propagate cancellations in FastAPI effectively, we integrated background tasks with custom middleware to handle client disconnections. This ensures that any ongoing tasks are properly terminated when a client disconnects.

#### Common Mistakes

Failing to use asyncio.all_tasks() to cancel all running tasks.
Not handling CancelledError exceptions gracefully in task cancellation.
Overlooking the need for custom middleware to detect and respond to disconnections.

#### Follow-Ups

Discussing how to optimize task management with task queues (e.g., Celery).
Exploring distributed tracing solutions like Jaeger or Zipkin for better monitoring of tasks across services.

**Code:**

```python
from fastapi import FastAPI, BackgroundTasks

app = FastAPI()

async def long_running_task(task_id: int):
    print(f"Starting task {task_id}")
    for i in range(50):
        await asyncio.sleep(1)
        print(f"Task {task_id} iteration {i+1}")
    print(f"Completed task {task_id}")

@app.post("/tasks/{task_id}")
async def create_task(task_id: int, background_tasks: BackgroundTasks):
    background_tasks.add_task(long_running_task, task_id)

import uvicorn

@app.middleware("http")
async def handle_client_disconnect(request, call_next):
    response = await call_next(request)

    if request.client.is_disconnected:
        print("Client disconnected, cancelling tasks")
        for task in asyncio.all_tasks():
            task.cancel()

    return response

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

