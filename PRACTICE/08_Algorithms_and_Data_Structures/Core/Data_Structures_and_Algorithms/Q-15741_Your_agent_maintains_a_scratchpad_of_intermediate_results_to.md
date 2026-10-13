---
type: "practice"
difficulty: "Intermediate"
---

# Your agent maintains a scratchpad of intermediate results tools can reference by handle; what structure gives O(1) lookup by handle while bounding memory as the trajectory grows, and how do you decide what to evict?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ LRU Cache Design](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/LRU_Cache_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, we need a data structure that supports constant-time (O(1)) lookups by unique handles while managing memory constraints efficiently as more results are added over time. This is a classic scenario where using a hash map in combination with a Least Recently Used (LRU) cache can provide an optimal solution.

The question is essentially asking for a mechanism to store and retrieve intermediate results quickly based on a unique handle, while ensuring that the storage remains bounded by evicting less recently used or older entries when the memory limit is reached. The key aspects are:

Lookup Efficiency: We need O(1) time complexity for lookups.
Memory Bounding: The system should not grow indefinitely and must manage to free up space as needed.

The primary data structure we will use is a hash map, which allows us to achieve O(1) lookup times by using the handle as a key. However, since the hash map alone does not manage memory constraints, we need an additional LRU cache mechanism. This combination ensures efficient storage and retrieval while managing memory usage.

Let's break down the solution step-by-step:

#### Triage
Question Probing: The interviewer is likely testing your understanding of data structures, algorithms, and memory management.

#### Assumptions
We are using a programming language that supports hash maps and LRU cache implementations (e.g., Python with lru_cache or collections.OrderedDict, Java with ConcurrentHashMap).
The handle is unique for each intermediate result.

#### First Principles Solution
Hash Map: Use a hash map to store the actual results indexed by their handles.
LRU Cache: Implement an LRU cache mechanism that keeps track of recently used entries and evicts them based on memory constraints.

#### Implementation
Example (Python)

Here's how we can implement this in Python:

PYTHON
from collections import OrderedDict

class Scratchpad:
    def __init__(self, max_size):
        self.max_size = max_size
        self.cache = OrderedDict()

    def put(self, handle, result):
        # Add the new result to the cache
        if handle not in self.cache:
            if len(self.cache) >= self.max_size:
                # Evict the least recently used item first
                self.cache.popitem(last=False)
        self.cache[handle] = result

    def get(self, handle):
        # Retrieve and update the access time of the result
        if handle in self.cache:
            value = self.cache[handle]
            del self.cache[handle]
            self.cache[handle] = value  # Reinsert to mark it as recently used
            return value
        return None

# Example usage
scratchpad = Scratchpad(max_size=5)
scratchpad.put("handle1", "result1")
scratchpad.put("handle2", "result2")
print(scratchpad.get("handle1"))  # Output: result1, and the cache now has (handle1, result1) marked as recently used

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: Using a combination of a hash map for fast lookups and an LRU cache to manage memory efficiently ensures that we meet the requirements.

#### Common Mistakes
Failing to implement the eviction mechanism correctly, leading to suboptimal performance or memory leaks.
Not considering edge cases like trying to get a handle that has never been put into the scratchpad.

#### Follow-Ups
How would you handle concurrent access from multiple threads?
What if the intermediate results are large and cannot be stored in memory? Would you consider using an external storage solution?

