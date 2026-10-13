---
type: "practice"
difficulty: "Deep"
---

# When would you pick a concurrent data structure (ConcurrentHashMap, lock-free queue) over external locking around a plain one? What do the concurrent structures NOT protect (compound operations) that still bite people?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Concurrent Data Structures](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Concurrent_Data_Structures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Concurrency patterns and race conditions are critical in ensuring data integrity when multiple threads access shared resources. When choosing between concurrent data structures like ConcurrentHashMap or lock-free queues, and external locking mechanisms around plain data structures (like synchronized blocks), it's essential to understand their trade-offs and limitations.

The primary advantage of concurrent data structures is that they abstract away much of the complexity associated with managing locks manually. However, they do not protect against all types of race conditions, particularly those involving compound operations or non-atomic steps within a single thread.

**Code:**

```java
// Example: A simple counter using synchronized block vs ConcurrentHashMap
class Counter {
    private int count = 0;

    public void incrementSync() {
        synchronized (this) {
            count++;
        }
    }

    public void incrementConcurrent() {
        count++; // Not thread-safe!
    }
}

public class ConcurrentMapExample {
    private final ConcurrentHashMap<Integer, Integer> map = new ConcurrentHashMap<>();

    public void incrementCounter(int key) {
        map.put(key, 1); // Thread-safe
    }
}
```

**Flow:**

1. Thread A
2. Read-Modify-Write
3. Synchronize
4. Update Count
5. Thread B
6. Read-Modify-Write
7. Synchronize
8. Update Count
9. Count Mismatch

