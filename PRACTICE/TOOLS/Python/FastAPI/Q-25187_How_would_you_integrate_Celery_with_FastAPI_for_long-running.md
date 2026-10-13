---
type: "practice"
difficulty: "Deep"
---

# How would you integrate Celery with FastAPI for long-running background jobs and get task results?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Long-running Jobs and Async Patterns](<../../../../TOOLS/Python/FastAPI/Long-running_Jobs_and_Async_Patterns.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: Integrating Celery with FastAPI for handling long-running background tasks involves setting up a worker to manage the tasks asynchronously, while the main application remains responsive. We'll cover the setup process, including task creation, worker configuration, and result retrieval.

#### Triaging the Question
The interviewer is probing your understanding of asynchronous processing in web applications using FastAPI and Celery, as well as how to properly integrate these tools to ensure performance and reliability. Key assumptions include a basic familiarity with FastAPI for handling HTTP requests and Celery's role in managing background tasks.

We'll start by creating a simple task function that performs some long-running computation (e.g., processing data), then move on to setting up the FastAPI application to accept these tasks, configure the Celery worker to handle them, and finally retrieve the results.

Step-by-Step Solution:

Setup Task Function in Celery: First, we define a task function using Celery that performs our long-running operation.

Configure FastAPI Application to Accept Tasks: We create an endpoint in FastAPI to submit tasks to the Celery worker.

Configure and Run the Celery Worker: We set up the Celery worker to listen for task requests and process them asynchronously.

Retrieve Task Results: Finally, we ensure that our FastAPI application can track and retrieve results from completed tasks.

Let's break this down in detail:

Define a Celery Task Function:

PYTHON
from celery import Celery

app = Celery('tasks', broker='redis://localhost:6379/0')

@app.task
def long_running_task(input_data):
    # Simulate a long-running process
    result = sum([i * i for i in range(input_data)])
    return result

FastAPI Application to Accept Tasks:

PYTHON
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from celery.result import AsyncResult

app = FastAPI()

class TaskRequest(BaseModel):
    input_data: int

@app.post("/tasks/")
async def create_task(task_request: TaskRequest):
    task_id = long_running_task.apply_async((task_request.input_data,))
    return {"task_id": task_id}

Configure and Run the Celery Worker:

SH
celery -A tasks worker --loglevel=info

This command starts a Celery worker that listens for task requests.

Retrieve Task Results: We can create another endpoint to retrieve results:

PYTHON
@app.get("/tasks/{task_id}/")
async def get_task_result(task_id: str):
    result = AsyncResult(task_id).get()
    if result is None:
        raise HTTPException(status_code=404, detail="Task not found or still running")
    return {"result": result}

#### Closing Summary
and Common Mistakes/Follow-Ups: By following this approach, we ensure that FastAPI remains responsive while Celery handles the long-running tasks. The key is proper task creation, worker configuration, and result retrieval. Common mistakes include not handling task states correctly or neglecting to configure Celery properly.

For follow-ups, you might be asked about scaling the Celery setup with multiple workers, using different brokers, or integrating more complex data models into tasks.

**Code:**

```

from celery import Celery

app = Celery('tasks', broker='redis://localhost:6379/0')

@app.task def long_running_task(input_data): result = sum([i * i for i in range(input_data)]) return result

CODE
Copy

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from celery.result import AsyncResult

app = FastAPI()

class TaskRequest(BaseModel):
    input_data: int

@app.post("/tasks/")
async def create_task(task_request: TaskRequest):
    task_id = long_running_task.apply_async((task_request.input_data,))
    return {"task_id": task_id}

@app.get("/tasks/{task_id}/")
async def get_task_result(task_id: str):
    result = AsyncResult(task_id).get()
    if result is None:
        raise HTTPException(status_code=404, detail="Task not found or still running")
    return {"result": result}
```

**Flow:**

1. Yes
2. No
3. Submit Task
4. Is task ID valid?
5. Return Result
6. Throw 404 error
7. Retrieve and return result

