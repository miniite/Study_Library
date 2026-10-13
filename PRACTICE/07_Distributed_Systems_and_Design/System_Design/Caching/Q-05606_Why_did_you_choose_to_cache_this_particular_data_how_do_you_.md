---
type: "practice"
difficulty: "Intermediate"
---

# Why did you choose to cache this particular data — how do you decide what is worth caching?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ What to Cache](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/What_to_Cache.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding whether to cache data, we must consider several factors including performance benefits, cost, and risk. The choice of which data to cache depends on its usage frequency, access patterns, and how it impacts overall system performance.

Firstly, let's triage the question. The interviewer is probing your understanding of caching mechanisms, the trade-offs involved in deciding what to cache, and your ability to analyze real-world scenarios for optimization. Common assumptions include knowing that caching can significantly reduce database load and improve response times but also increases memory usage and might introduce complexity.

Step-by-Step Solution

Evaluate Usage Frequency:

Determine how often a piece of data is accessed. High-frequency accesses justify caching since the cost of fetching from cache is lower than hitting the slower underlying store.

Analyze Access Patterns:

Check if the access pattern is consistent or varies over time. For example, user profiles that are frequently accessed during login might be good candidates for caching.

#### Cons
ider Data Lifespan and Freshness:

Determine how often the data changes. If the data rarely changes or can be refreshed infrequently (e.g., every 15 minutes), it's a strong candidate for caching.

Impact on Performance:

Assess whether caching improves performance enough to offset potential drawbacks, such as increased memory usage and the overhead of maintaining cache coherence.

#### Example Scenario

Let’s consider a scenario where we’re building an e-commerce website that frequently displays product information. We want to decide if it makes sense to cache this data.

Usage Frequency: Products are accessed often by customers.
Access Patterns: Product details are typically accessed when browsing, searching, or adding products to the cart.
Data Lifespan: Product descriptions and images may change infrequently (once a day).
Performance Impact: Reducing database queries can significantly improve page load times.

Based on these factors, caching product information would be beneficial. However, we must also consider:

Memory Usage: Caching all products could consume substantial memory.
Cache Coherence: Ensuring the cache stays up-to-date without causing unnecessary reloads or stale data issues.
Cost-Benefit Analysis: The performance gains should outweigh the cost of maintaining a cache.

#### Trade-offs
and Considerations

Memory Management:

Use caching strategies like LRU (Least Recently Used) to automatically evict less frequently accessed items.

Cache Refresh Mechanism:

Implement background jobs or use in-memory databases like Redis with TTLs (Time To Live) to ensure data freshness.

Failover and Recovery:

Design the system to handle cache misses gracefully, such as by automatically fetching from the database and then updating the cache.

Scalability:

For highly scalable systems, consider distributed caching solutions to avoid single points of failure.

#### Closing Summary

Choosing what data to cache involves a nuanced analysis that balances performance gains against potential costs like increased memory usage. By evaluating factors such as access frequency, lifespan, and impact on system performance, we can make informed decisions about which data to cache and how to manage it effectively.

#### Common mistakes
include over-caching (using too much memory), under-caching (missing opportunities for performance improvements), or not accounting for cache coherence and refresh mechanisms. Always consider the broader implications of your caching strategy in terms of overall system architecture, maintenance, and scalability.

**Comparison:**

| Performance Gains | 15% Improvement in Page Load Times | Reduced Load on the Database by 20% |

**Code:**

```python
# Example code to demonstrate a simple LRU Cache using Python's collections library

from collections import OrderedDict

class LRUCache:
    def __init__(self, capacity):
        self.cache = OrderedDict()
        self.capacity = capacity

    def get(self, key):
        if key not in self.cache:
            return -1
        else:
            self.cache.move_to_end(key)
            return self.cache[key]

    def set(self, key, value):
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value
        if len(self.cache) > self.capacity:
            self.cache.popitem(last=False)

# Usage example
cache = LRUCache(2)
cache.set('one', 1)
cache.set('two', 2)
print(cache.get('one'))       # returns 1
cache.set('three', 3)         # evicts key 'two'
print(cache.get('two'))       # returns -1 (not found)
cache.set('four', 4)          # evicts key 'one'
print(cache.get('one'))       # returns -1 (not found)
print(cache.get('three'))     # returns 3
print(cache.get('four'))      # returns 4
```

**Example input / output:**

```text
Input:
- Request for product details with id = 1

Output:
- Cached response for product 'Product A' is returned directly from cache.
- Cache miss occurs for other requests, and data is fetched from the database.

Dry-run:
- For frequent access to product details (e.g., login page), a hit rate of over 90% could be observed with caching enabled.
```

**Data:**

```json
{
  "products": [
    { "id": 1, "name": "Product A", "description": "Description of Product A" },
    { "id": 2, "name": "Product B", "description": "Description of Product B" }
  ]
}
```

**Flow:**

1. Yes
2. No
3. User Accesses Product Details
4. Is Data Cached?
5. Fetch from Cache
6. Fetch from Database
7. Update Cache with Fresh Data
8. Return Response to User

