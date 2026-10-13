---
type: "practice"
difficulty: "Intermediate"
---

# How would you implement a simple LRU cache to memoize expensive feature computations, and which two data structures back it?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ LRU Cache Design](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/LRU_Cache_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement an LRU (Least Recently Used) cache for memoizing expensive feature computations, we need to ensure that frequently accessed features are kept in memory while less used ones get evicted based on their recency of use. The two primary data structures involved are a hash table and a doubly linked list. A hash table allows us to quickly look up the most recent access time or value for each key, whereas a linked list helps maintain the order of usage so that we can efficiently remove the least recently used elements.

Firstly, let's outline the assumptions:

The cache has a fixed capacity.
The features are identified by keys (e.g., feature names).
We need to memoize results of computations on these features.

Now, I will walk you through implementing this solution step-by-step using first principles:

Define the Cache Class: This class will manage both the hash table and the linked list.
Initialize Structures: Set up a hash map for O(1) key lookups and a doubly linked list to maintain the order of usage.
Insert Operation: When inserting a new value, we check if it already exists in the cache; if so, update its value and move it to the head (most recently used). If not, insert it at the head while ensuring the cache does not exceed capacity by removing the least recently used item first.
Get Operation: Retrieve the value for a given key. If found, update the usage order by moving it to the head.

Here is a basic implementation in Python:

PYTHON
class LRUCache:
    def __init__(self, capacity):
        self.capacity = capacity
        self.cache = {}
        self.order = []  # Doubly linked list represented as an array for simplicity

    def get(self, key):
        if key in self.cache:
            self._move_to_head(key)
            return self.cache[key]
        else:
            return -1

    def set(self, key, value):
        if key in self.cache:
            self._move_to_head(key)
            self.cache[key] = value
        else:
            if len(self.order) == self.capacity:
                del_key = self.order.pop()
                del self.cache[del_key]
            self.order.insert(0, key)
            self.cache[key] = value

    def _move_to_head(self, key):
        # Remove the node from its current position
        idx = self.order.index(key)
        self.order.remove(key)

        # Insert it at the head of the list
        self.order.insert(0, key)

# Example usage:
cache = LRUCache(2)
cache.set('feature1', 10)
cache.set('feature2', 20)
print(cache.get('feature1'))  # Returns 10 and moves 'feature1' to head

