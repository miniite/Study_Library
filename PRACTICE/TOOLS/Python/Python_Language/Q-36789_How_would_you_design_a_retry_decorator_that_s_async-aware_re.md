---
type: "practice"
difficulty: "Deep"
---

# How would you design a retry decorator that's async-aware, respects a deadline budget across attempts, and doesn't retry non-idempotent or 4xx errors?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Retries with Deadline Budgets](<../../../../TOOLS/Python/Python_Language/Dead_Letter_and_Replay_Systems/Retries_with_Deadline_Budgets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an asynchronous retry decorator in Python that manages deadlines and skips certain error types, we need to ensure our solution is both robust and efficient. This involves handling async calls, respecting a budget of retries over time, and intelligently deciding whether to retry based on the nature of the errors. The key steps are:

Understanding Async Decorators: We'll leverage Python's asyncio library for asynchronous operations.
Deadline Management: Track the remaining budget using a deadline parameter.
Error Handling: Define conditions under which retries should not occur (non-idempotent and 4xx errors).

Here’s a step-by-step breakdown of how we can implement this:

Step 1: Set Up Basic Decorator Structure
We start by defining the core structure of our decorator, including parameters for retry count, initial deadline, delay between retries, etc.

Step 2: Asynchronous Function Handling
Ensure that the decorated function is properly awaited and its result managed.

Step 3: Deadline and Retry Logic
Implement logic to track time spent and manage retries within the given budget.

Step 4: Error Conditions for Non-Retry
Define specific error conditions under which no retry should occur, like non-idempotent operations or HTTP errors in the range 400-499.

