---
type: "practice"
difficulty: "Expert"
---

# You have a hot lock protecting a map that's read 100x more than written. Walk me through converting it to a lock-free/CAS-based read path and how you validate correctness with a stress + linearizability check.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Lock-free Read Paths](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Lock-free_Read_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

In this scenario, we need to convert a hot lock protecting a shared map into a lock-free/cas-based system while ensuring read operations are highly efficient and maintaining correctness. We'll achieve this by using atomic compare-and-swap (CAS) instructions for reads and leveraging concurrent data structures like ConcurrentHashMap or custom CAS-based structures. Validation will involve stress testing and checking linearizability to ensure the system behaves correctly under high load.

#### Triaging the Question

The question is asking us to refactor a scenario where a shared map, accessed 100x more for reads than writes, uses a hot lock (a heavily contended lock). Our goal is to reduce contention by converting read operations into a lock-free/CAS-based approach. We need to ensure that this change does not compromise the correctness of the system and validate it using stress testing and linearizability checks.

#### Assumptions
The map contains key-value pairs, where keys are unique.
Read operations significantly outnumber write operations (100:1 ratio).
The system needs to maintain strong consistency guarantees for reads and writes.
We have a concurrent environment with multiple threads accessing the map.

#### First-Principles Solution
Step 1: Define the Problem

The current scenario uses a hot lock, which is inefficient due to high contention during read operations. We need to replace this lock with CAS-based operations to reduce contention and improve performance.

Step 2: Analyze Read-Write Operations

Read operations should be fast and non-blocking, while write operations might still require locks but should minimize blocking time.

Step 3: Implement Lock-Free Reads Using CAS

To achieve lock-free reads, we can use atomic variables or concurrent collections like ConcurrentHashMap in Java. Here's how we can implement this:

Use AtomicReference for Controlling Map State:

We can maintain a reference to the map using an AtomicReference<ConcurrentMap<K,V>>. This allows us to atomically swap the map object during updates.

Optimistic Locking with CAS:

For reads, we attempt to get the latest version of the map without holding any lock.
If the read is successful (i.e., no concurrent update occurred), we process the read operation.
If a concurrent update occurs, we retry the read until it succeeds.

Write Operations with Locks:

Write operations will still use locks but aim to minimize blocking time by using fine-grained locking strategies or tryLock methods where possible.
Step 4: Example Implementation

Here's an example implementation in Java:

JAVA
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicReference;

public class LockFreeMap<K, V> {
    private final AtomicReference<ConcurrentHashMap<K, V>> mapRef = new AtomicReference<>(new ConcurrentHashMap<>());

    public V get(K key) {
        while (true) {
            ConcurrentHashMap<K, V> map = mapRef.get();
            V value = map.get(key);
            if (value != null) return value;
            // If no value found and concurrent update might have happened, retry
        }
    }

    public void put(K key, V value) {
        synchronized (this) {  // Use a lock to ensure atomic updates
            ConcurrentHashMap<K, V> map = mapRef.get();
            if (map.putIfAbsent(key, value) == null) {
                return;  // Successful update, no need for further action
            }
            while (!mapRef.compareAndSet(map, new ConcurrentHashMap<>(map))) {  // Atomically replace the map
                // Concurrency might have modified the map, retry
            }
        }
    }
}
Step 5: Validate Correctness

To validate correctness, we need to ensure that our system behaves as expected under stress and maintains linearizability.

Stress Testing:

Simulate high read/write patterns using a stress test tool.
Monitor the performance metrics such as latency, throughput, and contention levels.

Linearizability Check:

Ensure that every operation appears to complete atomically at some point between the start and finish of any enclosing operations.
Use tools like JUnit or custom assertions to verify that read operations return values consistent with historical writes.
Step 6: Common Mistakes
Incorrectly implementing CAS operations, leading to infinite retries without updates.
Failing to handle concurrent modifications properly, causing data corruption.
Overlooking the importance of maintaining linearizability under all scenarios.

#### Closing Summary
and Follow-Ups:

By converting the lock to a CAS-based approach, we can significantly reduce contention during read operations while ensuring correctness. Validating this through stress testing and linearizability checks is crucial to ensure that our system behaves as expected in production.

#### Common mistakes
include improper handling of concurrent updates and inadequate validation of atomicity and consistency.

**Code:**

```java
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicReference;

public class LockFreeMap<K, V> {
    private final AtomicReference<ConcurrentHashMap<K, V>> mapRef = new AtomicReference<>(new ConcurrentHashMap<>());

    public V get(K key) {
        while (true) {
            ConcurrentHashMap<K, V> map = mapRef.get();
            V value = map.get(key);
            if (value != null) return value;
            // If no value found and concurrent update might have happened, retry
        }
    }

    public void put(K key, V value) {
        synchronized (this) {  // Use a lock to ensure atomic updates
            ConcurrentHashMap<K, V> map = mapRef.get();
            if (map.putIfAbsent(key, value) == null) {
                return;  // Successful update, no need for further action
            }
            while (!mapRef.compareAndSet(map, new ConcurrentHashMap<>(map))) {  // Atomically replace the map
                // Concurrency might have modified the map, retry
            }
        }
    }
}
```

