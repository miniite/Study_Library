---
type: "practice"
difficulty: "Deep"
---

# What is the difference between concurrency and parallelism in the context of a FastAPI async application?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Async vs Sync Endpoints](<../../../../TOOLS/Python/FastAPI/Async_vs_Sync_Endpoints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Concurrent and parallel execution are two fundamental concepts in asynchronous programming, particularly relevant when discussing FastAPI applications. In essence, concurrent execution allows multiple tasks to appear as if they were running simultaneously, while parallel execution involves executing multiple tasks simultaneously on different cores or threads.

In the context of a FastAPI async application, understanding these distinctions is crucial for optimizing performance and ensuring efficient use of resources. Let's explore each concept in detail from first principles and then discuss their implementation in FastAPI.

#### Triage
What Is Being Probed

The interviewer is likely probing your understanding of:

Concurrency vs Parallelism: The key difference between the two.
Asynchronous Programming: How concurrency and parallelism are implemented using async/await patterns.
Resource Management: How to effectively manage resources in concurrent or parallel environments.

#### Assumptions
We assume you have a basic understanding of asynchronous programming concepts like coroutines, events, and tasks.
FastAPI is used as an example, but the principles apply broadly to any modern async framework.
Step-by-Step Solution from First Principles
Concurrency in Async Applications

Concurrency refers to the ability to handle multiple requests or tasks simultaneously. In a FastAPI application, concurrency is achieved using asynchronous coroutines and event loops. A coroutine can perform I/O-bound operations (like waiting for a database response) without blocking other coroutines.

For example, consider a scenario where you need to fetch data from two different APIs:

PYTHON
import asyncio

async def get_data_from_api1():
    # Simulate API call
    await asyncio.sleep(2)
    return 'Data 1'

async def get_data_from_api2():
    # Simulate API call
    await asyncio.sleep(2)
    return 'Data 2'

async def main():
    task1 = asyncio.create_task(get_data_from_api1())
    task2 = asyncio.create_task(get_data_from_api2())

    result1, result2 = await asyncio.gather(task1, task2)
    print(result1, result2)

# Run the event loop
asyncio.run(main())

In this example, get_data_from_api1 and get_data_from_api2 are run concurrently because they are executed within the same event loop. They do not block each other's execution.

Parallelism in Async Applications

Parallelism involves executing multiple tasks simultaneously on separate cores or threads. In a FastAPI application, parallelism is achieved by leveraging libraries like concurrent.futures to execute tasks in parallel.

For example, using the ThreadPoolExecutor:

PYTHON
from concurrent.futures import ThreadPoolExecutor

def get_data_from_api1():
    # Simulate API call
    import time
    time.sleep(2)
    return 'Data 1'

def get_data_from_api2():
    # Simulate API call
    import time
    time.sleep(2)
    return 'Data 2'

if __name__ == '__main__':
    with ThreadPoolExecutor(max_workers=2) as executor:
        future1 = executor.submit(get_data_from_api1)
        future2 = executor.submit(get_data_from_api2)

        print(future1.result())
        print(future2.result())

In this example, get_data_from_api1 and get_data_from_api2 are executed in parallel on different threads. Each function runs independently of the other.

#### Closing Summary

In summary, concurrency allows multiple tasks to appear as if they were running simultaneously within the same process, while parallelism involves executing multiple tasks simultaneously across different cores or threads. Both concepts can be effectively utilized in FastAPI applications using coroutines and async/await patterns for concurrency, and thread pools or processes for parallelism.

#### Common Mistakes
and Follow-Ups
Misunderstanding: Often, candidates confuse concurrency with parallelism, leading to incorrect implementation.
Resource Management: Not managing resources correctly can lead to issues like deadlocks or resource exhaustion.
Synchronization: Failing to properly synchronize tasks can result in race conditions.

When discussing these concepts, it's important to emphasize the differences and provide concrete examples of how they are implemented.

**Code:**

```

import asyncio

async def get_data_from_api1(): await asyncio.sleep(2) return 'Data 1'

async def get_data_from_api2(): await asyncio.sleep(2) return 'Data 2'

async def main(): task1 = asyncio.create_task(get_data_from_api1()) task2 = asyncio.create_task(get_data_from_api2())

CODE
Copy
result1, result2 = await asyncio.gather(task1, task2)
print(result1, result2)
Run the event loop

asyncio.run(main())

CODE
Copy

```python
from concurrent.futures import ThreadPoolExecutor

def get_data_from_api1():
    time.sleep(2)
    return 'Data 1'

def get_data_from_api2():
    time.sleep(2)
    return 'Data 2'

if __name__ == '__main__':
    with ThreadPoolExecutor(max_workers=2) as executor:
        future1 = executor.submit(get_data_from_api1)
        future2 = executor.submit(get_data_from_api2)

        print(future1.result())
        print(future2.result())
```

**Flow:**

1. Yes
2. No
3. Main
4. Is Task Concurrent?
5. Use asyncio
6. Use ThreadPoolExecutor
7. Fetch Data from API1
8. Fetch Data from API2
9. Submit Task to Executor
10. H
11. I
12. J
13. K
14. L

