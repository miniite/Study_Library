---
type: "practice"
difficulty: "Intermediate"
---

# Explain how the LRU (Least Recently Used) eviction policy works.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Eviction Policies and TTL](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Eviction_Policies_and_TTL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

LRU caching is a fundamental technique used to manage cache memory efficiently, ensuring that frequently accessed items remain in the cache while less recently used items are evicted. In this explanation, I'll break down how LRU works from first principles and discuss its implementation details.

The core idea behind LRU is simple: when the cache reaches its capacity and a new item needs to be stored, we remove the least recently used item. This policy helps maintain a high probability that frequently accessed items remain in the cache, thereby reducing the number of disk or database accesses required.

To implement LRU caching, we typically use a combination of a hash map for quick lookups and an ordered list (or linked list) to keep track of the order of usage. Here's how it works step-by-step:

Initialization: Start with an empty cache that can hold up to N items.
Insertion:
If the key is not already in the cache, add the new item.
If the key is present, update its value and mark it as recently used (e.g., by moving it to the front of the list).
Eviction:
When the cache reaches capacity, insert a new item while removing the least recently used item from both the hash map and the ordered list.

The key insight is that by maintaining order in the list, we can efficiently determine which items have been least recently accessed.

Now let's dive into an example to see this in action:

