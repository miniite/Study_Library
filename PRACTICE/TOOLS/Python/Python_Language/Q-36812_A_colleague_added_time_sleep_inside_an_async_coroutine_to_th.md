---
type: "practice"
difficulty: "Intermediate"
---

# A colleague added time.sleep() inside an async coroutine 'to throttle' and throughput collapsed. Explain what happened and what they should have used.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Asyncio Concurrency](<../../../../TOOLS/Python/Python_Language/Asyncio_Concurrency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a colleague inserted time.sleep() in an asynchronous coroutine to "throttle" and manage throughput, it inadvertently caused the entire async operation to be serialized instead of running concurrently. This is because Python's time.sleep() function is synchronous and blocks the execution thread for the specified duration. In the context of an async environment, this meant that each async task would wait for the previous one to complete before proceeding, leading to a significant performance drop.

To understand the issue more clearly, let's break down what happened step by step:

Synchronous time.sleep(): When you use time.sleep() in Python, it blocks the current thread completely. This is fine in synchronous code where only one task can run at a time per thread.
Async Context Misunderstanding: In an async environment, tasks are supposed to be handled concurrently using event loops and coroutines, which allow multiple tasks to run without blocking each other. However, when time.sleep() was inserted into an async coroutine, it effectively turned the async task into a synchronous one, causing it to block the event loop.
Threading Confusion: The colleague might have misunderstood that sleep in Python is synchronous and thus did not realize its impact on concurrent execution. In contrast, they should be using async-native mechanisms designed for non-blocking operations.

Now, let's solve this issue by reasoning from first principles:

Identify the Problem: The root cause was the blocking nature of time.sleep() inside an async coroutine.
Use Async-Aware Sleep: Replace time.sleep() with asyncio.sleep(). This function is designed to work within an event loop and does not block the execution; instead, it puts the task to sleep without affecting other tasks in the event loop.
Ensure Non-Blocking Operations: Make sure all operations inside the async coroutine are non-blocking (i.e., use asynchronous versions of I/O, network, or file operations).

Here’s a step-by-step solution:

Update the Code:

PYTHON
import asyncio

async def throttle_coroutine():
    await asyncio.sleep(0.5)  # Use asyncio.sleep() instead of time.sleep()
    print("Coroutine finished")

loop = asyncio.get_event_loop()
tasks = [throttle_coroutine(), throttle_coroutine()]
loop.run_until_complete(asyncio.wait(tasks))

Explanation: The asyncio.sleep() function does not block the event loop, allowing other coroutines to run concurrently during the sleep period.

Testing: Run the updated code in an async environment and observe that both tasks can run concurrently without blocking each other.

In summary, the colleague's mistake was using a synchronous sleep mechanism (time.sleep()) in an asynchronous context, which serialized execution and reduced throughput. The correct approach is to use asyncio.sleep() to maintain non-blocking behavior and ensure concurrent task execution.

