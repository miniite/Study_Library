---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between a synchronous def and an async def path operation in FastAPI?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Async vs Sync Endpoints](<../../../../TOOLS/Python/FastAPI/Async_vs_Sync_Endpoints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In FastAPI, the primary distinction between def (synchronous) and async def (asynchronous) path operations lies in how they handle I/O-bound tasks. Synchronous functions execute sequentially, one after another, whereas asynchronous functions can run concurrently by suspending their execution when waiting for I/O operations to complete.

#### Triaging the Question

The question is asking about the differences between synchronous and asynchronous functions specifically within the context of FastAPI's routing system. The key points to address are:

Definition: What does def and async def represent in Python?
Behavior: How do they behave differently when called, especially with I/O-bound tasks like database queries or network requests?
Performance Implications: When should you use one over the other?

#### First-Principles Solution
1. Definition:
A def function is a synchronous function that waits for I/O operations to complete before continuing.
An async def function, on the other hand, can yield control back to the event loop when it encounters an awaitable expression.
2. Behavior with I/O Bound Tasks
Synchronous (def): If your path operation involves a database query or network request that takes time (I/O-bound), the entire application will block until this task is completed. This can lead to significant delays and poor user experience, especially for web applications.
Asynchronous (async def): When an I/O-bound task is awaited within an async def function, control returns to the event loop immediately. The function does not block, allowing other tasks to run in parallel.
3. Performance Implications
Synchronous: Best for CPU-bound tasks or simple operations that do not involve waiting on external resources.
Asynchronous: Ideal for I/O-bound tasks as it maximizes resource utilization and responsiveness of your application.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, choosing between def and async def in FastAPI path operations depends on the nature of the task. For CPU-bound tasks or simple operations, def is sufficient. However, for I/O-bound tasks such as database queries, network requests, or file I/O, async def offers better performance and user experience.

#### Common Mistakes

Overusing Asynchronous: Using asynchronous functions for non-I/O bound tasks can introduce unnecessary complexity without providing benefits.
Misunderstanding Synchronization: Not recognizing the difference between CPU-bound and I/O-bound operations can lead to misapplication of async def.

#### Example Scenario

#### Cons
ider a scenario where you need to fetch user data from a database before rendering a response:

Synchronous Approach (def): The function will wait for the database query to complete, blocking other requests until it returns.
Asynchronous Approach (async def): The function can return control back to the event loop while waiting for the database query, allowing other requests to be processed concurrently.
PYTHON
# Synchronous example (def)
from fastapi import FastAPI

app = FastAPI()

@app.get("/user/{user_id}")
def get_user_sync(user_id: int):
    # Simulate a time-consuming operation (e.g., database query)
    user_data = get_user_from_db(user_id)
    return user_data

# Asynchronous example (async def)
from fastapi import FastAPI
import asyncio

app = FastAPI()

@app.get("/user/{user_id}")
async def get_user_async(user_id: int):
    # Simulate a time-consuming operation (e.g., database query)
    await asyncio.sleep(1)  # Placeholder for async task
    user_data = await get_user_from_db(user_id)
    return user_data

# Function to simulate a database call (real implementation should use actual DB connection)
def get_user_from_db(user_id: int):
    import time
    time.sleep(2)
    return {"id": user_id, "name": f"User {user_id}"}
Self-Check

The answer is comprehensive and covers the essential differences between def and async def in FastAPI. It provides clear examples and highlights common mistakes to avoid.

**Code:**

```python
# Synchronous example (def)
from fastapi import FastAPI

app = FastAPI()

@app.get("/user/{user_id}")
def get_user_sync(user_id: int):
    # Simulate a time-consuming operation (e.g., database query)
    user_data = get_user_from_db(user_id)
    return user_data

# Asynchronous example (async def)
from fastapi import FastAPI
import asyncio

app = FastAPI()

@app.get("/user/{user_id}")
async def get_user_async(user_id: int):
    # Simulate a time-consuming operation (e.g., database query)
    await asyncio.sleep(1)  # Placeholder for async task
    user_data = await get_user_from_db(user_id)
    return user_data

# Function to simulate a database call (real implementation should use actual DB connection)
def get_user_from_db(user_id: int):
    import time
    time.sleep(2)
    return {"id": user_id, "name": f"User {user_id}"}
```

