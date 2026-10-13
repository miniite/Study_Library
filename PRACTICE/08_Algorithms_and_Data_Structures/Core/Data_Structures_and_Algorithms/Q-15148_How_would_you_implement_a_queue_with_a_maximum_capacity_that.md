---
type: "practice"
difficulty: "Intermediate"
---

# How would you implement a queue with a maximum capacity that blocks producers when full?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Bounded Blocking Queues](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Bounded_Blocking_Queues.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement a queue with a maximum capacity that blocks producers when full, we need to ensure two key aspects:

Enforce a fixed size limit on the queue.
Block producer threads once the queue reaches its capacity.

In this explanation, I will break down the steps from first principles and provide a step-by-step solution suitable for an intermediate-level engineer.

#### Triage

The core of the question is to design a queue that enforces a maximum number of elements (capacity) and blocks producers when full. This involves understanding thread synchronization and managing blocking behavior in concurrent programming. We assume we are working with Java, but similar principles apply to other languages as well.

#### First-Principles Solution

We will use a combination of an ArrayQueue or LinkedBlockingQueue, which already provides some concurrency support, along with custom logic for blocking producers when the queue is full. The solution involves creating a wrapper class around a standard queue implementation and overriding methods to add producer blocking.

Step-by-Step Implementation
1. Import Necessary Libraries

First, we need to import necessary libraries:

JAVA
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;
2. Define the BlockingQueue Class

We will define a LimitedCapacityBlockingQueue class that wraps around either an ArrayBlockingQueue or LinkedBlockingQueue. This wrapper class ensures that producers are blocked when the queue is full.

JAVA
public class LimitedCapacityBlockingQueue<T> {
    private final int capacity;
    private final BlockingQueue<T> backingQueue;

    public LimitedCapacityBlockingQueue(int capacity, boolean fair) {
        this.capacity = capacity;
        if (fair) {
            this.backingQueue = new LinkedBlockingQueue<>(capacity);
        } else {
            this.backingQueue = new ArrayBlockingQueue<>(capacity);
        }
    }

    // Method to add elements
    public void put(T element) throws InterruptedException {
        synchronized (backingQueue) {
            while (backingQueue.size() == capacity) {
                backingQueue.wait();  // Wait if the queue is full
            }
            backingQueue.add(element);  // Add element and notify waiting producers
            backingQueue.notifyAll();
        }
    }

    // Method to get elements
    public T take() throws InterruptedException {
        synchronized (backingQueue) {
            while (backingQueue.isEmpty()) {
                backingQueue.wait();  // Wait if the queue is empty
            }
            return backingQueue.poll(0, TimeUnit.MILLISECONDS);  // Get and remove element
        }
    }

    // Method to check if the queue is full
    public boolean isFull() {
        synchronized (backingQueue) {
            return backingQueue.size() == capacity;
        }
    }

    // Method to check if the queue is empty
    public boolean isEmpty() {
        synchronized (backingQueue) {
            return backingQueue.isEmpty();
        }
    }
}

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, we have created a LimitedCapacityBlockingQueue class that wraps around a standard queue implementation. This ensures that producers are blocked when the queue is full by using synchronization and wait/notify mechanisms.

#### Common Mistakes

Not Using Synchronization: Forgetting to synchronize access to the backing queue can lead to concurrency issues.
Incorrect Wait Conditions: Misusing wait() or notifyAll() methods can cause infinite loops.
Lack of Error Handling: Not properly handling exceptions and edge cases like thread interruption.
Triangulating the Solution

The interviewer might probe deeper into:

Choosing Between ArrayBlockingQueue and LinkedBlockingQueue: Discussing pros and cons in terms of performance, fairness, and memory usage.
Thread Safety and Performance Considerations: Exploring how different data structures can impact performance under concurrent access.
Alternative Approaches: Suggesting other methods like using a Semaphore or implementing custom synchronization mechanisms.

By covering these points, you demonstrate your understanding of the problem and your ability to think critically about potential solutions.

**Code:**

```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;

public class LimitedCapacityBlockingQueue<T> {
    private final int capacity;
    private final BlockingQueue<T> backingQueue;

    public LimitedCapacityBlockingQueue(int capacity, boolean fair) {
        this.capacity = capacity;
        if (fair) {
            this.backingQueue = new LinkedBlockingQueue<>(capacity);
        } else {
            this.backingQueue = new ArrayBlockingQueue<>(capacity);
        }
    }

    public void put(T element) throws InterruptedException {
        synchronized (backingQueue) {
            while (backingQueue.size() == capacity) {
                backingQueue.wait();  // Wait if the queue is full
            }
            backingQueue.add(element);  // Add element and notify waiting producers
            backingQueue.notifyAll();
        }
    }

    public T take() throws InterruptedException {
        synchronized (backingQueue) {
            while (backingQueue.isEmpty()) {
                backingQueue.wait();  // Wait if the queue is empty
            }
            return backingQueue.poll(0, TimeUnit.MILLISECONDS);  // Get and remove element
        }
    }

    public boolean isFull() {
        synchronized (backingQueue) {
            return backingQueue.size() == capacity;
        }
    }

    public boolean isEmpty() {
        synchronized (backingQueue) {
            return backingQueue.isEmpty();
        }
    }
}

None
```

