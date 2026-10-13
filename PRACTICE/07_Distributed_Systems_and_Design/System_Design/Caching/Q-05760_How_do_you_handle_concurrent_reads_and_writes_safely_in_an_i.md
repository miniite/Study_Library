---
type: "practice"
difficulty: "Deep"
---

# How do you handle concurrent reads and writes safely in an in-memory cache?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Write-through and Write-back Consistency](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Write-through_and_Write-back_Consistency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with concurrent access to an in-memory cache, ensuring safety and consistency is crucial. This problem involves managing both read and write operations concurrently without violating data integrity or causing race conditions. The key challenge lies in handling these operations safely, especially in a multi-threaded environment.

#### Triaging the Question

The question asks us to design a caching mechanism that can handle concurrent reads and writes effectively. Here, we are probing our knowledge of thread safety mechanisms, synchronization strategies, and how to manage shared state in a parallel computing context.

#### Assumptions
We are working with an in-memory key-value cache.
Multiple threads may access the cache simultaneously for reading and writing operations.
The cache should maintain data consistency under concurrent accesses.
Step-by-Step Solution
1. Synchronization Mechanism: Locks or Read-Write Locks

To handle concurrent reads and writes, we can use a synchronization mechanism that allows multiple readers but prevents any writer from accessing the cache until all readers have finished their operations.

Yes
No
Yes
No
Read Operation
Acquire Read Lock
Write Operation
Acquire Write Lock
Is Cache Full?
Mark as Inaccessible
Release Read Lock
Perform Read/Write
Release Read Lock
Is Cache Empty?
Mark as Inaccessible
Release Write Lock
Perform Read/Write
Release Write Lock
2. Thread-safe Data Structures

Utilizing thread-safe data structures like ConcurrentHashMap in Java can help manage concurrent reads and writes more efficiently.

JAVA
import java.util.concurrent.ConcurrentHashMap;

public class InMemoryCache {
    private final ConcurrentHashMap<String, Object> cache = new ConcurrentHashMap<>();

    public void put(String key, Object value) {
        // Synchronized block to ensure thread safety
        synchronized (cache) {
            cache.put(key, value);
        }
    }

    public Object get(String key) {
        return cache.get(key);
    }
}
3. Read-Write Locks for Fine-grained Control

Using ReentrantReadWriteLock can provide more granular control over read and write operations.

JAVA
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class InMemoryCache {
    private final ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
    private final ConcurrentHashMap<String, Object> cache = new ConcurrentHashMap<>();

    public void put(String key, Object value) {
        rwLock.writeLock().lock();
        try {
            cache.put(key, value);
        } finally {
            rwLock.writeLock().unlock();
        }
    }

    public Object get(String key) {
        rwLock.readLock().lock();
        try {
            return cache.get(key);
        } finally {
            rwLock.readLock().unlock();
        }
    }
}
4. Handling Cache Invalidation

For scenarios where the cache needs to be invalidated or refreshed, ensure that all readers and writers are aware of these changes.

JAVA
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class InMemoryCache {
    private final ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
    private final ConcurrentHashMap<String, Object> cache = new ConcurrentHashMap<>();

    public void put(String key, Object value) {
        rwLock.writeLock().lock();
        try {
            cache.put(key, value);
            // Notify all readers and writers about the update
            notifyAllReadersAndWriters();
        } finally {
            rwLock.writeLock().unlock();
        }
    }

    public Object get(String key) {
        rwLock.readLock().lock();
        try {
            return cache.get(key);
        } finally {
            rwLock.readLock().unlock();
        }
    }

    private void notifyAllReadersAndWriters() {
        // Logic to notify all readers and writers
    }
}

#### Closing Summary

Handling concurrent reads and writes in an in-memory cache requires careful synchronization to ensure data consistency. Using appropriate locking mechanisms like ReentrantReadWriteLock can help manage read-write operations effectively. Additionally, ensuring proper cache invalidation logic is crucial for maintaining the integrity of cached data.

#### Common Mistakes/Follow-ups
Incorrect Lock Usage: Not using locks properly or incorrectly can lead to race conditions.
Synchronization Overhead: Excessive locking can degrade performance. Consider other strategies like optimistic concurrency control where applicable.
Cache Invalidation Strategy: Ensuring that all parts of the system are aware of cache changes is crucial, especially in distributed environments.

**Comparison:**

```
Key	Value
key1	value1
key2	value2
```

**Code:**

```java
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class InMemoryCache {
    private final ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
    private final ConcurrentHashMap<String, Object> cache = new ConcurrentHashMap<>();

    public void put(String key, Object value) {
        rwLock.writeLock().lock();
        try {
            cache.put(key, value);
            notifyAllReadersAndWriters();
        } finally {
            rwLock.writeLock().unlock();
        }
    }

    public Object get(String key) {
        rwLock.readLock().lock();
        try {
            return cache.get(key);
        } finally {
            rwLock.readLock().unlock();
        }
    }

    private void notifyAllReadersAndWriters() {
        // Logic to notify all readers and writers
    }
}
```

**Example input / output:**

```
Input:
JAVA
Copy
cache.put("key1", "value1");
System.out.println(cache.get("key1")); // Output: value1
Output:
CODE
Copy
value1
```

**Data:**

```json
{
  "key1": "value1",
  "key2": "value2"
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Read Operation
6. Acquire Read Lock
7. Write Operation
8. Acquire Write Lock
9. Is Cache Full?
10. Mark as Inaccessible
11. Release Read Lock
12. Perform Read/Write
13. Release Read Lock
14. Is Cache Empty?
15. Mark as Inaccessible
16. Release Write Lock
17. Perform Read/Write
18. Release Write Lock

