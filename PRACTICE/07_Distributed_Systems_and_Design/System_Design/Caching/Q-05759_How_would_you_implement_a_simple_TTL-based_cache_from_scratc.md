---
type: "practice"
difficulty: "Deep"
---

# How would you implement a simple TTL-based cache from scratch using a HashMap?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Eviction Policies and TTL](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Eviction_Policies_and_TTL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement a simple TTL (Time To Live) based cache using a HashMap, we need to balance memory efficiency with performance, while also ensuring that entries expire according to their TTL values. Here’s an overview of our approach and the key steps involved.

Firstly, let's define what a TTL-based cache does: it stores data in memory but ensures that each entry has a finite lifespan before being evicted. This is essential for scenarios like reducing database hits or managing hot data in real-time applications.

#### Triaging the Question

The core of this question revolves around understanding and implementing a caching mechanism with expiration capabilities. The interviewer might be probing your knowledge on:

Data structures (specifically, HashMap).
Memory management.
Understanding cache eviction policies.
Handling concurrent access scenarios.
Error handling and edge cases.

#### Assumptions
We are using Java for this implementation as it provides strong type safety and robust collection support.
The HashMap will be used to store key-value pairs where the value is a custom object encapsulating the actual data and its TTL.
We need to handle concurrent access using thread-safe mechanisms.
Step-by-Step Solution
Define Cache Entry Class: A class to hold both the value and its expiration time.
Implement Thread-Safe HashMap: Use ConcurrentHashMap for concurrent access.
TTL Calculation and Eviction Logic: Schedule entries for removal using a timer framework like ScheduledExecutorService.
Concurrency Handling: Ensure thread safety by locking mechanisms or using atomic operations.
1. Define Cache Entry Class
JAVA
public class CacheEntry {
    private final String value;
    private final Instant expirationTime;

    public CacheEntry(String value, long ttlMillis) {
        this.value = value;
        this.expirationTime = Instant.now().plus(ttlMillis);
    }

    public boolean isExpired() {
        return Instant.now().isAfter(expirationTime);
    }
}
2. Implement Thread-Safe HashMap
JAVA
import java.util.concurrent.ConcurrentHashMap;

public class TTLCache<K, V> {
    private final ConcurrentHashMap<K, CacheEntry<V>> cache = new ConcurrentHashMap<>();
    private final ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);

    public void put(K key, V value, long ttlMillis) {
        if (key == null || value == null) {
            throw new IllegalArgumentException("Key and Value must not be null");
        }
        CacheEntry<V> entry = new CacheEntry<>(value, ttlMillis);
        cache.put(key, entry);
        // Schedule a task to remove the entry
        scheduler.schedule(() -> removeEntry(key), ttlMillis, TimeUnit.MILLISECONDS);
    }

    public V get(K key) {
        CacheEntry<V> entry = cache.get(key);
        if (entry != null && !entry.isExpired()) {
            return entry.getValue();
        }
        return null;
    }

    private void removeEntry(K key) {
        cache.remove(key);
    }
}
3. Handling Concurrent Access and Scheduling

The ScheduledExecutorService is used to schedule removal tasks, ensuring that even if the main thread is busy or blocked, entries still get evicted on time.

#### Closing Summary
+ Common Mistakes/Follow-Ups

This implementation ensures that cache entries are stored in a thread-safe manner and automatically removed when their TTL expires. The use of ScheduledExecutorService helps manage expiration without blocking the main application logic.

#### Common Mistakes

Forgetting to handle null checks.
Not considering edge cases where TTL might be zero or negative.
Ignoring concurrency issues, leading to race conditions.
Misusing remove() which does not trigger scheduled tasks.
Triangulation with Follow-Ups

The interviewer might probe further by asking about:

Alternative eviction policies (e.g., LRU).
Performance optimizations for very large caches.
Handling of non-deterministic TTLs or variable expiration times.
Integration with other caching layers or distributed systems.

