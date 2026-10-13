---
type: "practice"
difficulty: "Deep"
---

# You need to cancel a group of related async tasks if any one fails. Compare hand-rolled cancellation with asyncio.TaskGroup and how cancellation propagates and is caught.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Asyncio Concurrency](<../../../../TOOLS/Python/Python_Language/Asyncio_Concurrency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of cancelling a group of related asynchronous tasks when any single task fails, we can compare two approaches: a hand-rolled implementation using asyncio.Event and asyncio.wait_for, versus leveraging asyncio.TaskGroup. Both methods have their strengths and trade-offs. In this response, I'll break down the concepts from first principles and walk through both implementations.

#### Triaging the Question

The core of the question is to ensure that if any task in a group fails, all other tasks are cancelled as well. This involves:

Handling Task Failures: Ensuring proper exception handling.
Task Group Management: Managing multiple asynchronous tasks efficiently.
Cancellation Propagation: Making sure the cancellation signal reaches all relevant tasks.

#### Assumptions
and Context
We will use Python 3.7 or later, which supports asyncio.TaskGroup.
Tasks are I/O-bound; hence, they can be cancelled based on their failure.
The task group may contain a large number of tasks (e.g., 100+).

#### First Principles Solution
Hand-Rolled Cancellation with asyncio.Event and asyncio.wait_for

The idea here is to use an event to signal cancellation and wait for all tasks until the event is set.

PYTHON
import asyncio

class AsyncTask:
    def __init__(self, task_func):
        self.task_func = task_func
        self.event = asyncio.Event()

    async def run(self):
        try:
            await self.task_func()
        except Exception as e:
            print(f"Task failed: {e}")
            # Signal cancellation to other tasks
            self.event.set()

async def cancel_group(tasks):
    # Wait for all tasks to finish or any one of them to fail and signal cancellation
    done, pending = await asyncio.wait(
        [task.run() for task in tasks],
        return_when=asyncio.FIRST_EXCEPTION,
    )
    if pending:
        for task in pending:
            task.cancel()
        await asyncio.gather(*pending)

# Example usage
async def example_task():
    import random
    import time

    await asyncio.sleep(random.uniform(0, 1))
    print("Task completed")

tasks = [AsyncTask(example_task) for _ in range(5)]
await cancel_group(tasks)
Using asyncio.TaskGroup for Cancellation Propagation

Python’s asyncio.TaskGroup is a high-level construct designed to manage and coordinate multiple tasks. It handles cancellation for you, making the implementation simpler.

PYTHON
import asyncio

async def example_task():
    import random
    import time

    await asyncio.sleep(random.uniform(0, 1))
    print("Task completed")

async def cancel_group(task_group):
    try:
        async with task_group as group:
            tasks = [group.spawn(example_task()) for _ in range(5)]
    except asyncio.CancelledError:
        print("Group cancelled")
        # Cancel all pending tasks
        for task in (await group.cancel()):
            await task

# Example usage
async def main():
    task_group = asyncio.TaskGroup()
    try:
        await cancel_group(task_group)
    finally:
        task_group.cancel()

asyncio.run(main())

#### Closing Summary
and Common Mistakes/Follow-Ups
Hand-Rolled Solution: This approach requires more boilerplate code but gives you fine-grained control over how tasks are managed. It's suitable when custom behavior is necessary.
TaskGroup Approach: This simplifies the cancellation logic by handling it at a higher level. However, it may not be as flexible in edge cases.

#### Common Mistakes
Forgetting to handle asyncio.CancelledError properly can lead to unexpected behavior.
Not understanding how tasks are cancelled within a TaskGroup.
Misusing the return_when parameter in asyncio.wait.
Additional Considerations
Performance: The performance overhead of using TaskGroup is minimal, but it’s essential to test both approaches for specific use cases.
Concurrency and Scalability: Both methods should scale well, but TaskGroup might be slightly more efficient due to internal optimizations.

**Code:**

```

import asyncio

class AsyncTask: def init(self, task_func): self.task_func = task_func self.event = asyncio.Event()

CODE
Copy
async def run(self):
    try:
        await self.task_func()
    except Exception as e:
        print(f"Task failed: {e}")
        # Signal cancellation to other tasks
        self.event.set()

async def cancel_group(tasks): done, pending = await asyncio.wait( [task.run() for task in tasks], return_when=asyncio.FIRST_EXCEPTION, ) if pending: for task in pending: task.cancel() await asyncio.gather(*pending)

Example usage

async def example_task(): import random import time

CODE
Copy
await asyncio.sleep(random.uniform(0, 1))
print("Task completed")

tasks = [AsyncTask(example_task) for _ in range(5)] await cancel_group(tasks)

CODE
Copy

```python
import asyncio

async def example_task():
    import random
    import time

    await asyncio.sleep(random.uniform(0, 1))
    print("Task completed")

async def cancel_group(task_group):
    try:
        async with task_group as group:
            tasks = [group.spawn(example_task()) for _ in range(5)]
    except asyncio.CancelledError:
        print("Group cancelled")
        # Cancel all pending tasks
        for task in (await group.cancel()):
            await task

# Example usage
async def main():
    task_group = asyncio.TaskGroup()
    try:
        await cancel_group(task_group)
    finally:
        task_group.cancel()

asyncio.run(main())
```

