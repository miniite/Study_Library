---
type: "practice"
difficulty: "Deep"
---

# Implement a bounded producer-consumer queue by hand. Where exactly do you need condition variables (or wait/notify), and what bug appears if you use notify instead of notifyAll, or forget the wait-in-a-loop rule?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Bulkheads and Thread Pools](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Bulkheads_and_Thread_Pools.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement a bounded producer-consumer queue, we need to manage shared state carefully to ensure thread safety. The key concurrency patterns here involve Condition variables (or wait, notify, and notifyAll) to synchronize producers and consumers without causing race conditions or deadlocks.

We will use Java as our example language, but the principles are broadly applicable in other concurrent programming contexts. The core challenge is ensuring that the queue does not overflow for producers and is fully consumed by consumers while maintaining visibility of state changes across threads.

**Comparison:**

```
Condition	Wait/Notify Behavior
notFull	put() waits if full; take() signals
notEmpty	take() waits if empty; put() signals
```

**Code:**

```java
import java.util.LinkedList;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class BoundedQueue<T> {
    private final Lock lock = new ReentrantLock();
    private final Condition notFull = lock.newCondition();
    private final Condition notEmpty = lock.newCondition();
    private final LinkedList<T> queue = new LinkedList<>();
    private static final int CAPACITY = 10; // Maximum capacity

    public void put(T item) throws InterruptedException {
        lock.lock();
        try {
            while (queue.size() == CAPACITY) {
                notFull.await(); // Wait if the queue is full
            }
            queue.addLast(item);
            notEmpty.signalAll(); // Notify all waiting consumers
        } finally {
            lock.unlock();
        }
    }

    public T take() throws InterruptedException {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                notEmpty.await(); // Wait if the queue is empty
            }
            T item = queue.removeFirst();
            notFull.signalAll(); // Notify all waiting producers
            return item;
        } finally {
            lock.unlock();
        }
    }
}
```

**Example input / output:**

```text
// Example usage:
BoundedQueue<Integer> queue = new BoundedQueue<>();

try {
    queue.put(1); // Producer thread operation
    Thread.sleep(100);
    int value = queue.take(); // Consumer thread operation, should return 1
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();
}
```

**Flow:**

1. Produce Item
2. Is Queue Full?
3. X Full, Wait
4. Queue Put Item
5. Notify All Consumers
6. Y Not Full, Add Item
7. Take Item
8. Is Queue Empty?
9. X Empty, Wait
10. Queue Take Item
11. Notify All Producers

