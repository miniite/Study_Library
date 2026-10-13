---
type: "practice"
difficulty: "Intermediate"
---

# A client caches API results in an ever-growing JS Map for "instant" repeat access; how do you reason about the browser-tab memory complexity over a long session and how you'd bound it?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Streaming vs In-memory Space](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Streaming_vs_In-memory_Space.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with caching API responses using Map in a browser tab, understanding time and space complexity is crucial to ensure that your application performs well even as data grows over a long session. The key here is balancing cache size with memory usage while ensuring quick access times.

#### Triaging the Question

The question is asking about how to manage the growing size of an Map object in JavaScript, which acts as our cache for API results. We need to consider both time (how quickly can we retrieve data) and space (the amount of memory used by the cache). The interviewer likely wants to understand your approach to setting up a caching mechanism that scales well over long sessions.

#### Assumptions
Data Growth: Assume that the API responses grow in size or number over time.
Browser Memory Constraints: Recognize that browser tabs have limited memory and that excessive caching can lead to performance issues or crashes.
Access Patterns: Consider typical access patterns for cached data, such as frequent reads and infrequent writes.

#### First-Principles Solution

To reason about the space complexity, we need to consider how our cache grows over time and if it exceeds browser memory limits. For time complexity, we want quick access to cached items, ideally in constant or near-constant time (O(1)).

Space Complexity Analysis

The Map object used for caching API responses typically has a space complexity of O(n), where n is the number of cached entries. As more API results are fetched and stored, this size can grow significantly. To manage memory usage:

Memory Budget: Define a maximum memory budget in bytes or kilobytes that your cache should not exceed.
Cache Eviction Policy: Implement an eviction policy to remove less frequently accessed data as the cache grows.
Time Complexity Analysis

For time complexity, using Map provides O(1) average-time access due to its hash table nature. However, in practice, performance can degrade if collisions occur or if too many entries are added leading to frequent rehashing operations.

#### Implementation
Steps

To implement an effective caching mechanism:

Define a Memory Limit: Set a maximum allowed memory size for the cache.
Use Map for Caching: Use a JavaScript Map to store API results and their corresponding timestamps or access frequencies.
Eviction Policy:
LRU (Least Recently Used): Evict the least recently used items first.
LFU (Least Frequently Used): Evict the items with the lowest frequency of access.

#### Example
Implementation

Here’s a simple implementation in JavaScript:

JAVASCRIPT
class LruCache {

#### cons
tructor(limit) {
    this.cache = new Map();
    this.limit = limit;
    this.size = 0;
  }

  get(key) {
    if (!this.cache.has(key)) return null;

#### cons
t value = this.cache.get(key);
    this.updateAccessFrequency(key);

    return value;
  }

  set(key, value) {
    if (this.cache.has(key)) {
      this.cache.delete(key);
    }
    this.cache.set(key, value);
    this.size += 1;

    // Evict least recently used item if cache exceeds limit
    if (this.size > this.limit) {

#### cons
t firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
      this.size -= 1;
    }
  }

  updateAccessFrequency(key) {
    // Update access frequency, e.g., by moving the entry to the end

#### cons
t value = this.cache.get(key);
    this.cache.delete(key);
    this.cache.set(key, value);
  }
}

#### Closing Summary
and Common Mistakes/Follow-Ups

To manage memory complexity effectively:

Define a clear memory limit.
Use appropriate eviction policies like LRU or LFU.
Regularly monitor cache size to ensure it doesn’t exceed the defined limits.

#### Common mistakes
include not setting any memory limits, using only Map without considering its performance characteristics, and failing to implement an effective eviction strategy. Follow-ups might explore specific trade-offs between different caching strategies and their impacts on application performance.

**Code:**

```javascript
class LruCache {
  constructor(limit) {
    this.cache = new Map();
    this.limit = limit;
    this.size = 0;
  }

  get(key) {
    if (!this.cache.has(key)) return null;

    const value = this.cache.get(key);
    this.updateAccessFrequency(key);

    return value;
  }

  set(key, value) {
    if (this.cache.has(key)) {
      this.cache.delete(key);
    }
    this.cache.set(key, value);
    this.size += 1;

    // Evict least recently used item if cache exceeds limit
    if (this.size > this.limit) {
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
      this.size -= 1;
    }
  }

  updateAccessFrequency(key) {
    // Update access frequency, e.g., by moving the entry to the end
    const value = this.cache.get(key);
    this.cache.delete(key);
    this.cache.set(key, value);
  }
}
```

