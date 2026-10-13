---
type: "practice"
difficulty: "Intermediate"
---

# You're reviewing a PR that adds a "quick" synchronized block inside a loop that also does a network call. What's the concurrency red flag, and how would you coach the author to fix it?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Contention and Slow-burn Incidents](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Contention_and_Slow-burn_Incidents.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When reviewing a pull request (PR) that introduces a synchronized block within a loop performing network calls, the primary concern is race conditions and potential deadlocks. The concurrency red flag here is that the synchronized block could lead to a bottleneck, where multiple threads are waiting on each other, thereby significantly reducing performance. To address this, we need to understand how synchronization works and explore alternative patterns.

Let's break down the issue step by step:

Understanding the Problem: A loop with network calls inside it is already inherently sequential because of I/O operations, but adding a synchronized block can cause contention if multiple threads attempt to enter this block simultaneously.
Synchronization Red Flag: In Java, for instance, using synchronized on a method or block will create a lock that only one thread can hold at any time. If multiple threads try to call the synchronized method concurrently, they will queue up and wait until the lock is released. This can lead to performance degradation as all other threads are blocked.

#### Assumptions
We assume that the network calls inside the loop do not depend on each other's results (i.e., they are independent). If there are dependencies, we need to handle them appropriately.

Now, let’s explore a few possible solutions:

Use Asynchronous Calls: Instead of synchronizing, consider using asynchronous methods like async/await or futures to handle the network calls. This allows threads to continue executing while waiting for I/O operations to complete.
Threading Patterns:
Thread Pool with ExecutorService: Use a thread pool to manage background tasks. This can help in handling multiple network calls efficiently without blocking the main execution flow.
Non-blocking IO Libraries: Consider using non-blocking frameworks like Netty or similar libraries that are designed to handle I/O operations efficiently.

Here’s an example of how you might refactor the code:

JAVA
// Original Synchronized Block
for (int i = 0; i < data.length; i++) {
    synchronized (lockObject) { // Potential bottleneck here
        networkCall(data[i]);
    }
}

// Refactored Asynchronous Call Example
ExecutorService executor = Executors.newFixedThreadPool(10); // Adjust thread count as needed

for (int i = 0; i < data.length; i++) {
    Future<Void> future = executor.submit(() -> {
        networkCall(data[i]); // Non-blocking call
        return null;
    });
}

// Ensure proper shutdown of the ExecutorService after use
executor.shutdown();

By using asynchronous calls, we can ensure that threads do not block each other and can continue processing while waiting for I/O operations to complete.

In summary, the key issue is avoiding unnecessary synchronization in a loop with network calls. Instead, leveraging asynchronous patterns can help maintain concurrency and improve performance.

**Code:**

```java
// Example of original and refactored code
```

