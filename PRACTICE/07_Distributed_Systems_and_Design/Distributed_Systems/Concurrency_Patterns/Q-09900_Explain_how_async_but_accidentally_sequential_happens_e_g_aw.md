---
type: "practice"
difficulty: "Deep"
---

# Explain how "async but accidentally sequential" happens — e.g., awaiting inside a loop instead of gathering. How do you spot it in code review and in a trace, and quantify the latency cost?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Contention and Slow-burn Incidents](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Contention_and_Slow-burn_Incidents.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with concurrency patterns and race conditions in backend development, one common pitfall is "async but accidentally sequential" behavior, which occurs when asynchronous operations are awaited sequentially within loops or functions instead of being gathered and executed concurrently. This pattern can significantly impact performance because it forces the execution to wait for each operation to complete before moving on, defeating the purpose of asynchrony.

#### Triaging the Question

The question is essentially asking us to understand how this pattern emerges in code, identify its presence during code reviews and trace analysis, and quantify the associated latency costs. We'll assume that we're dealing with a typical asynchronous JavaScript environment using Promises or async/await syntax.

#### First-Principles Solution
Code Review

When reviewing code, look for patterns like this:

JAVASCRIPT
for (let i = 0; i < 10; i++) {
    await someAsyncOperation(i);
}

Here, someAsyncOperation is awaited inside a loop. Each iteration waits until the previous operation completes before starting the next one.

Trace Analysis

In a trace or profiling tool, you would see each asynchronous call being executed sequentially:

Operation 1 starts and completes.
Operation 2 starts only after Operation 1 finishes.
This pattern continues for all operations up to the nth iteration.

The key insight is that while these operations are technically asynchronous in syntax, they are not truly parallel because of the sequential awaits. The cost here is essentially a linear increase in latency proportional to the number of iterations.

Quantifying Latency Cost

To quantify this latency cost, we can use a simple formula:

If each operation takes t milliseconds and there are n operations, the total time taken with "async but accidentally sequential" code would be ( n \times t ) milliseconds.
In contrast, if the same number of asynchronous operations were executed in parallel (without awaits inside a loop), the total execution time would ideally be close to t milliseconds.

#### Closing Summary
and Common Mistakes/Follow-Ups

This pattern often stems from developers incorrectly translating synchronous logic into async code without considering the concurrency benefits. To avoid it, refactor loops that contain awaits into gathering multiple promises with Promise.all or equivalent constructs:

JAVASCRIPT

#### cons
t operations = [];
for (let i = 0; i < 10; i++) {
    operations.push(someAsyncOperation(i));
}
await Promise.all(operations);

In the trace, this would show all asynchronous calls starting nearly simultaneously and completing in parallel.

#### Common mistakes
include:

Not recognizing that await inside a loop is not concurrency-friendly.
Overlooking the performance implications of sequential awaits.
Misunderstanding how Promises and async/await work under the hood.

#### Example Scenario

Imagine you have an e-commerce application where you need to fetch product details from a database for multiple users. If each request is processed sequentially due to await inside a loop, it could lead to significant delays as the server waits for each operation to complete before moving on to the next one.

#### Common Mistakes
Failing to distinguish between "async" and "parallel."
Overusing await in loops without proper concurrency handling.
Not considering the overhead of sequential execution versus parallel execution.

**Code:**

```javascript
// Example code with async but accidentally sequential pattern
for (let i = 0; i < 10; i++) {
    await fetchProductDetails(i);
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is each operationasynchronous?
5. Refactor to use Promise.all
6. Sequential await is correct here
7. Trace shows parallel execution
8. Trace shows sequentialexecution

