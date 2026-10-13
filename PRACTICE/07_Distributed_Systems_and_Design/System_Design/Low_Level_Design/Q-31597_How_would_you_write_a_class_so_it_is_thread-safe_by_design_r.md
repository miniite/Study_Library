---
type: "practice"
difficulty: "Expert"
---

# How would you write a class so it is thread-safe by design rather than by adding locks everywhere?

**Difficulty:** Expert · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Lock Granularity Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/Lock_Granularity_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a class to be thread-safe, the goal is to ensure that multiple threads can safely access shared resources without causing data corruption or race conditions. This is often achieved by using synchronization mechanisms like locks, but doing so everywhere in your code can lead to excessive overhead and complexity. A better approach is to design the class with thread safety in mind from the beginning. In this answer, I'll outline a general strategy for writing thread-safe classes at an expert level.

#### Triaging the Question

The core of the question revolves around designing a class that is inherently thread-safe without relying on pervasive locking mechanisms. This involves understanding the synchronization requirements, identifying critical sections where thread safety is necessary, and implementing these sections efficiently. The interviewer might be probing your knowledge of concurrency principles, design patterns like Reader-Writer Locks or optimistic concurrency control, and the trade-offs between different approaches.

#### First-Principles Solution
Identify Critical Sections: Determine which parts of the class need to be synchronized. Typically, methods that access shared state or modify instance variables are critical sections.
Choose Appropriate Synchronization Mechanisms: Use higher-level synchronization primitives like Reader-Writer Locks (RWLock) or atomic operations where possible.
Minimize Contention: Reduce the number of lock acquisitions by minimizing the scope and duration of locks.
Designing a Thread-Safe Class

Let's consider an example class CacheManager that stores items in a cache and provides methods for adding, removing, and retrieving items:

PYTHON
from threading import Lock, RLock

class CacheManager:
    def __init__(self):
        self.cache = {}
        # Use ReadWriteLock to allow concurrent reads while blocking writes
        self.lock = RLock()

    def add_item(self, key: str, value: any) -> None:
        with self.lock.write_lock():
            if key not in self.cache:
                self.cache[key] = value

    def remove_item(self, key: str) -> bool:
        with self.lock.write_lock():
            return self.cache.pop(key, None) is not None

    def get_item(self, key: str) -> any:
        with self.lock.read_lock():
            return self.cache.get(key)

#### Closing Summary
and Common Mistakes

In this example, we used RLock (a recursive lock), which allows the same thread to acquire a read or write lock multiple times without deadlocking. This is particularly useful in scenarios where reads are more frequent than writes.

#### Common mistakes
include:

Overusing locks: Locking every method can lead to performance bottlenecks.
Inappropriate locking granularity: Not using the correct type of lock (read/write) can lead to unnecessary contention.
Failing to handle concurrent access correctly: Missing a critical section or not properly managing read and write operations.
Triaging Follow-Ups

The interviewer might ask about specific scenarios, such as handling high-contention cases, integrating with other thread-safe libraries, or discussing the implications of your design choices on system performance and scalability.

**Code:**

```python
from threading import RLock

class CacheManager:
    def __init__(self):
        self.cache = {}
        self.lock = RLock()

    def add_item(self, key: str, value: any) -> None:
        with self.lock.write_lock():
            if key not in self.cache:
                self.cache[key] = value

    def remove_item(self, key: str) -> bool:
        with self.lock.write_lock():
            return self.cache.pop(key, None) is not None

    def get_item(self, key: str) -> any:
        with self.lock.read_lock():
            return self.cache.get(key)
```

