---
type: "practice"
difficulty: "Deep"
---

# You must implement "wait until N events happen, then proceed once." Compare a CountDownLatch, a Phaser/CyclicBarrier, and a counting semaphore. When does the reusable barrier matter and what's the reset race?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Contention and Slow-burn Incidents](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Contention_and_Slow-burn_Incidents.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When implementing a concurrency pattern that requires waiting until N events occur before proceeding, we often face the challenge of ensuring synchronization and coordination among threads. In this context, we can compare three different patterns: CountDownLatch, Phaser/CyclicBarrier, and counting semaphores. Each has its own strengths and use cases.

TL;DR
CountDownLatch: Useful for one-time wait until N events occur.
Phaser/CyclicBarrier: Ideal for reusable, multiple rounds of waiting.
Counting Semaphore: Flexible but more complex to manage.

#### Triage
The Question at Hand

The question is essentially asking us to compare and contrast CountDownLatch, Phaser, and counting semaphores in terms of their use cases, trade-offs, and common scenarios where each would be most suitable. We need to consider factors like reusability, simplicity, and the specific requirements of the synchronization problem.

#### Assumptions
We assume a scenario where multiple threads perform some work and must wait until all have completed before proceeding.
The focus is on understanding when a reusable barrier (Phaser) might be necessary versus when simpler constructs suffice.

#### First-Principles Solution
CountDownLatch: One-Time Wait Until N Events Occur

A CountDownLatch allows one or more threads to wait for a set of events to occur. Each time an event occurs, the count is decremented. The waiting thread proceeds once the count reaches zero. This pattern is typically used in one-time scenarios where coordination among threads needs to happen only once.

Code Example:

JAVA
CountDownLatch latch = new CountDownLatch(N);
for (int i = 0; i < N; i++) {
    new Thread(() -> {
        // Perform some work...
        latch.countDown();
    }).start();
}
latch.await(); // Main thread waits until all events occur.
Phaser: Reusable, Multiple Rounds of Waiting

A Phaser is a specialized synchronization pattern designed for scenarios where threads need to coordinate multiple times. Unlike CountDownLatch, which can only be used once, Phaser allows threads to synchronize in rounds and reset the phase count between iterations.

Code Example:

JAVA
Phaser phaser = new Phaser(N);
for (int i = 0; i < N; i++) {
    new Thread(() -> {
        // Perform some work...
        phaser.arriveAndAwaitAdvance(); // Advance to next phase.
    }).start();
}
phaser.arriveAndDeregister(); // Main thread waits until all threads advance.
Counting Semaphore: Flexible Synchronization

A counting semaphore can be used for more flexible synchronization. It allows a fixed number of permits, which can be acquired and released by multiple threads. While it offers greater flexibility, managing the state correctly is more complex.

Code Example:

JAVA
Semaphore semaphore = new Semaphore(N);
for (int i = 0; i < N; i++) {
    new Thread(() -> {
        try {
            semaphore.acquire(); // Acquire a permit.
            // Perform some work...
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        semaphore.release(); // Release the permit.
    }).start();
}
semaphore.acquire(N); // Main thread waits until all permits are acquired.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Reusable Barrier: Use Phaser when you need to synchronize threads across multiple rounds. It is particularly useful in scenarios like parallel job scheduling or iterative tasks where synchronization happens repeatedly.
One-Time Wait: For a single-time wait, use CountDownLatch. This pattern is simpler and more straightforward.
Flexibility vs Complexity: A counting semaphore offers greater flexibility but requires careful management to avoid deadlocks or resource leaks.

#### Common mistakes
include:

Misusing a one-time pattern like CountDownLatch in scenarios that require multiple rounds of synchronization.
Overcomplicating the solution with Semaphore when simpler constructs like CountDownLatch or Phaser would suffice.
Failing to understand the reset race condition in Phaser, which can lead to unexpected behavior if not handled correctly.
Common Follow-Ups
Handling Failures: How do you ensure that all threads respect the synchronization pattern when one fails?

#### Performance Considerations
Which solution would be more performant for a large number of threads and why?
Scalability: In what scenarios might Phaser outperform other constructs, and in which it may not?

**Code:**

```

// CountDownLatch example CountDownLatch latch = new CountDownLatch(N); for (int i = 0; i < N; i++) { new Thread(() -> { // Perform some work... latch.countDown(); }).start(); } latch.await(); // Main thread waits until all events occur.

CODE
Copy

```java
// Phaser example
Phaser phaser = new Phaser(N);
for (int i = 0; i < N; i++) {
    new Thread(() -> {
        // Perform some work...
        phaser.arriveAndAwaitAdvance(); // Advance to next phase.
    }).start();
}
phaser.arriveAndDeregister(); // Main thread waits until all threads advance.
JAVA
Copy
// Semaphore example
Semaphore semaphore = new Semaphore(N);
for (int i = 0; i < N; i++) {
    new Thread(() -> {
        try {
            semaphore.acquire(); // Acquire a permit.
            // Perform some work...
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        semaphore.release(); // Release the permit.
    }).start();
}
semaphore.acquire(N); // Main thread waits until all permits are acquired.
```

