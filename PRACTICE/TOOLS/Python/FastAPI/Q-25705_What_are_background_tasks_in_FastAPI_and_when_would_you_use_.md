---
type: "practice"
difficulty: "Intermediate"
---

# What are background tasks in FastAPI and when would you use BackgroundTasks?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Long-running Jobs and Async Patterns](<../../../../TOOLS/Python/FastAPI/Long-running_Jobs_and_Async_Patterns.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Background tasks in FastAPI allow you to run functions asynchronously, ensuring that your application can handle multiple requests without blocking the event loop. This is particularly useful in scenarios where a task might take some time to complete but isn't essential for immediate response (like sending an email or scheduling a job). In this answer, I'll explain what BackgroundTasks are, when and why you'd use them, and walk through an example.

FastAPI uses BackgroundTasks from the fastapi background_tasks module. These tasks are executed asynchronously after a request has completed, meaning they don't block the response to the client. You might use this feature in situations like:

Email Sending: You want to send emails but don’t need immediate confirmation.
Scheduled Jobs: Running periodic tasks without waiting for them to finish.
Data Processing: Performing heavy computations or fetching data that can run separately from your request handling.

Let's dive into a practical example of using BackgroundTasks in FastAPI:

#### Triaging the Question

The core issue is understanding how asynchronous background tasks are implemented and used effectively within FastAPI. The interviewer might be probing your knowledge on:

Asynchronous programming concepts.
How to manage non-blocking operations in web applications.
Best practices for handling requests that don't require an immediate response.

#### First-Principles Solution

In a FastAPI application, you can use the BackgroundTasks feature to run functions asynchronously. This is achieved by defining tasks and adding them to the background task queue using BackgroundTasks.

#### Example
Code
PYTHON
from fastapi import FastAPI, BackgroundTasks
import time

app = FastAPI()

@app.post("/long-operation/")
async def long_operation(background_tasks: BackgroundTasks):
    print("Request received")

    # Simulate a long-running operation
    for i in range(50_000_000):
        pass

    background_tasks.add_task(slow_task)

    return {"message": "Long operation scheduled"}

def slow_task():
    time.sleep(10)  # Simulating a heavy computation
    print("Slow task completed")

In this example:

The long_operation function simulates a long-running process.
It then schedules the slow_task to be run asynchronously using BackgroundTasks.
This ensures that the response is sent back to the client immediately, and the slow_task runs in the background.

#### Closing Summary

In summary, BackgroundTasks are a powerful feature in FastAPI for executing tasks asynchronously without blocking the main request handling loop. This is particularly useful when dealing with operations that don’t need immediate responses from your web application. By leveraging these tasks, you can ensure smooth and efficient performance, enhancing the user experience.

#### Common Mistakes/Follow-Ups
Forgetting to add the task: Always remember to use background_tasks.add_task(task_name) to schedule a background task.
Blocking the main loop: Ensure that long-running operations are handled outside of the main request loop to avoid blocking.
Understanding the lifecycle: Be aware of how tasks are managed and when they might be executed, as there can be edge cases where tasks aren't run due to shutdown or other conditions.

If asked further, you could discuss more advanced topics like task prioritization, error handling for background tasks, and integrating external services into these tasks.

**Code:**

```python
from fastapi import FastAPI, BackgroundTasks
import time

app = FastAPI()

@app.post("/long-operation/")
async def long_operation(background_tasks: BackgroundTasks):
    print("Request received")

    # Simulate a long-running operation
    for i in range(50_000_000):
        pass

    background_tasks.add_task(slow_task)

    return {"message": "Long operation scheduled"}

def slow_task():
    time.sleep(10)  # Simulating a heavy computation
    print("Slow task completed")
```

