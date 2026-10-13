---
type: "practice"
difficulty: "Expert"
---

# How would you design a multi-level cache (L1 in-process, L2 shared) with correct invalidation, and what data structure and stamping avoids serving a value the L2 already evicted?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ LRU Cache Design](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/LRU_Cache_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a multi-level cache system with correct invalidation (L1 in-process and L2 shared), we need to focus on maintaining consistency, minimizing latency, and ensuring efficient data access. The key challenge is ensuring that when an item is evicted from the L2 cache, it's also invalidated in the L1 caches of all processes without causing unnecessary performance overhead.

#### Triaging the Question

The core of this question lies in understanding how to manage a hierarchical caching system effectively while ensuring data consistency and minimizing cache misses. We need to address:

Data Structure: What data structures will we use for both levels?
Invalidation Mechanism: How will we handle invalidations efficiently without causing contention or unnecessary delays?
Stamping and Timestamping: How can we ensure that a value is not served by L1 if it has been evicted from L2?
First Principles Approach

Data Structures:

L1 Cache (In-process): We will use an in-memory data structure like a hash map or a dictionary for fast access.
L2 Cache (Shared): A shared memory structure such as a Hash Table, but with support for invalidation mechanisms.

Invalidation Mechanism:

The L2 cache needs to provide a way to invalidate entries so that they are not served by the L1 caches.
We need to design an efficient invalidation protocol to avoid frequent communication between processes and to minimize latency.

Stamping and Timestamping:

Each value stored in both levels of the cache will have a timestamp or version number.
This allows us to check if the value has been evicted from L2 by comparing timestamps before serving an item from L1.
Detailed Solution
Step 1: Design the Data Structures

L1 Cache: Use a hash map where keys are unique and values contain both the actual data and a timestamp.

PYTHON
class L1Cache:
    def __init__(self):
        self.cache = {}

    def get(self, key):
        if key in self.cache:
            return (self.cache[key], True)  # Return value and hit status
        else:
            return None

    def set(self, key, value, timestamp):
        self.cache[key] = (value, timestamp)

L2 Cache: Use a similar hash map structure with support for invalidation.

PYTHON
class L2Cache:
    def __init__(self):
        self.cache = {}

    def get(self, key):
        if key in self.cache:
            return (self.cache[key][0], True)  # Return value and hit status

    def set(self, key, value, timestamp):
        self.cache[key] = (value, timestamp)

    def invalidate(self, key):
        if key in self.cache:
            del self.cache[key]
Step 2: Implement Invalidation Protocol
Shared Memory: Use a shared memory mechanism like Redis or Memcached for the L2 cache to ensure it can be accessed and invalidated by multiple processes.
Notification Mechanism: When an item is evicted from L2, use notifications (e.g., Pub/Sub) to inform all relevant L1 caches.
Step 3: Handling Timestamps
Each value in both caches will have a timestamp. Before serving from the L1 cache, compare the timestamps:
PYTHON
def get_value(self, key):
    l1_value = self.l1_cache.get(key)
    if l1_value is not None and (l2_value := self.l2_cache.get(key))[0] != l1_value[0]:
        return l2_value  # Serve from L2 as the timestamp has changed
    else:
        return l1_value or self.l2_cache.get(key)  # Serve from L1 if hit, else get from L2 and update L1

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, we designed a multi-level cache system with an in-process (L1) and shared memory (L2) tier. We used hash maps for both caches to ensure fast access. The key was implementing correct invalidation by using timestamps to ensure that values are not served from the L1 cache if they have been evicted from the L2 cache.

#### Common Mistakes

Failing to consider the invalidation mechanism, which can lead to stale data being served.
Not using timestamps correctly, leading to incorrect value serving decisions.
Overlooking the shared memory requirement for efficient invalidation across processes.

#### Follow-Ups
How would you handle cache eviction policies (LRU, LFU) in this setup?
What are the trade-offs between different data structures and their impact on performance?
How would you scale this system to handle more requests and processes?

**Code:**

```python
class L1Cache:
    def __init__(self):
        self.cache = {}

    def get(self, key):
        if key in self.cache:
            return (self.cache[key][0], True)  # Return value and hit status
        else:
            return None

    def set(self, key, value, timestamp):
        self.cache[key] = (value, timestamp)

class L2Cache:
    def __init__(self):
        self.cache = {}

    def get(self, key):
        if key in self.cache:
            return (self.cache[key][0], True)  # Return value and hit status

    def set(self, key, value, timestamp):
        self.cache[key] = (value, timestamp)

    def invalidate(self, key):
        if key in self.cache:
            del self.cache[key]

def get_value(l1_cache, l2_cache):
    l1_value = l1_cache.get(key)
    if l1_value is not None and (l2_value := l2_cache.get(key))[0] != l1_value[0]:
        return l2_value  # Serve from L2 as the timestamp has changed
    else:
        return l1_value or l2_cache.get(key)  # Serve from L1 if hit, else get from L2 and update L1
```

**Example input / output:**

```text
# Example usage
l1_cache = L1Cache()
l2_cache = L2Cache()

# Set values in both caches
l1_cache.set('key1', 'value1', 10)
l2_cache.set('key1', 'value2', 20)

# Get value from the system
print(get_value(l1_cache, l2_cache))  # Output: ('value2', True)
```

**Flow:**

1. Yes
2. No
3. Valid
4. Invalid
5. User Request
6. Is L1 Hit?
7. L1 Serve
8. Is L2 Valid?
9. L2 Serve and Invalidate L1
10. L2 Get, Update L1

