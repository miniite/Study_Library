---
type: "practice"
difficulty: "Intermediate"
---

# What is FIFO cache eviction and when might it be a bad choice?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Cache Stampede Prevention](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Cache_Stampede_Prevention.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

FIFO (First-In, First-Out) cache eviction means that the first item inserted into the cache will be the first one to be removed if space in the cache is needed. While simple and straightforward, FIFO caching can sometimes lead to suboptimal performance due to its strict adherence to a "first come, first served" policy.

The question is probing your understanding of different cache eviction policies and when specific strategies might not be optimal. You should assume you are discussing this concept in the context of backend systems managing data or resource requests efficiently.

Let's break down the reasoning step-by-step:

#### Overview
FIFO caching removes the oldest item first, which can lead to performance issues if frequently accessed items are older than less important ones.
Core Concept: FIFO is straightforward but not always the best choice because it doesn’t consider the importance or recency of data.
Step-by-Step Reasoning:

#### Cons
ider a scenario where you have a cache managing user session states, and frequently accessed users are being evicted before infrequently used ones.
In another case, when caching database query results, if the most recent queries (which might be more relevant now) are older than less recent but critical queries, FIFO can lead to poor performance.

Now let's summarize this with a few common mistakes and follow-ups:

#### Common Mistakes

Assuming that FIFO is always the best choice.
Not considering other eviction policies like LRU (Least Recently Used), which might be more appropriate in many scenarios.

#### Follow-Ups

Discussing how different cache sizes affect FIFO performance.
Exploring when other policies like LFU (Least Frequently Used) might be better suited.

**Comparison:**

| Operation | Cache State |
|-----------|-------------|
| Get A     | Miss        |
| Put B     | Hit ['A']   |
| Get B     | Hit         |
| Put C     | Hit ['A', 'B'] |
| Get C     | Hit         |
| Put D     | Evicts 'A'  ['B', 'C']|

**Code:**

```python
# Example of a simple FIFO cache implementation using a list to simulate cache behavior
class FIFOCache:
    def __init__(self, capacity):
        self.capacity = capacity
        self.cache = []

    def get(self, key):
        if key in self.cache:
            return f"Cache hit: {key}"
        else:
            return "Cache miss"

    def put(self, key):
        if len(self.cache) >= self.capacity:
            # Evict the oldest item
            self.cache.pop(0)
        self.cache.append(key)

# Example usage
cache = FIFOCache(capacity=3)
print(cache.get('A'))  # Cache miss
print(cache.put('A'))
print(cache.get('B'))  # Cache miss
print(cache.put('B'))
print(cache.get('C'))  # Cache miss
print(cache.put('C'))
print(cache.get('D'))  # Evicts 'A', Cache miss
```

**Example input / output:**

```text
Input:
- Cache capacity = 3
- Operations: [get('A'), put('B'), put('C'), put('D')]

Output (FIFO cache):
1. get('A'): Cache miss
2. put('B'): Cache hits ['A']
3. get('B'): Cache hit
4. put('C'): Cache hits ['A', 'B']
5. get('C'): Cache hit
6. put('D'): Evicts 'A' (since capacity is full), Cache hits ['B', 'C']
7. get('D'): Cache miss
```

**Data:**

```json
{
    "cache_capacity": 3,
    "operations": [
        {"key": "A", "operation": "get"},
        {"key": "B", "operation": "put"},
        {"key": "C", "operation": "put"},
        {"key": "D", "operation": "put"}
    ]
}
```

**Flow:**

1. Get A
2. Miss
3. Put B
4. HIT ['A']
5. Get B
6. HIT
7. Put C
8. HIT ['A', 'B']
9. Get C
10. HIT
11. Put D
12. Evicts A, HIT ['B', 'C']
13. Get D
14. Miss

