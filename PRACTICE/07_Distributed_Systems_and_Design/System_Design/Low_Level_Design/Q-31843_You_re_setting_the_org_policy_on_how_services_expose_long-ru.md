---
type: "practice"
difficulty: "Deep"
---

# You're setting the org policy on how services expose long-running operations (async job + status endpoint vs blocking); why does the sync-blocking anti-pattern keep reappearing and how do you make the async pattern the easy default in your framework?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Two-level Cache Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/Two-level_Cache_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a system that involves long-running operations, one might be tempted to use synchronous (blocking) calls, especially when the operation is expected to complete quickly or to ensure immediate feedback. However, this approach has significant drawbacks and can lead to scalability issues, thread pool exhaustion, and poor user experience. In contrast, an asynchronous pattern allows for non-blocking calls, enabling better resource utilization and more responsive systems. The goal of your framework should be to make the asynchronous (async job + status endpoint) model the default choice by reducing friction for developers.

The core challenge lies in understanding why the synchronous (sync-blocking) approach keeps reappearing despite its known disadvantages. This often happens because:

Developers are accustomed to synchronous patterns.
There might be a lack of robust and straightforward asynchronous implementation libraries or APIs.
Performance metrics can sometimes favor blocking code under certain conditions.

To address this, you need to establish guidelines and design principles within your framework that promote the use of asynchronous operations by default, making it as easy for developers to write non-blocking code as it is to write blocking code. This requires a deep understanding of both the technical and organizational factors at play.

**Comparison:**

```
Synchronous	Asynchronous
Blocks I/O	Non-blocking
Single-threaded	Multi-threaded or async-friendly
Can lead to deadlock and resource exhaustion	More efficient use of resources
```

**Code:**

```python
# Example of an async job in Python using asyncio

import asyncio

async def long_running_task():
    await asyncio.sleep(5)  # Simulate a long-running task with sleep
    return "Task completed"

# Example usage within a framework

class AsyncService:
    async def perform_async_operation(self):
        result = await self.long_running_task()  # Call the async function
        print(result)
```

**Example input / output:**

```

Input: Call perform_async_operation on AsyncService instance.

Output:

CODE
Copy
Task completed
```

**Data:**

```json
{
  "service": {
    "name": "AsyncService",
    "operation": "perform_async_operation",
    "description": "Performs an asynchronous operation and prints the result"
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. User Request
6. Is the operation long-running?
7. Submit a job and returnimmediately
8. Is the current thread busy withother tasks?
9. Handle request synchronously
10. Handle requestasynchronously, submit toevent loop

