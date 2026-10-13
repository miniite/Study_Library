---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd design a distributed inference cache for repeated identical prompts to cut GPU cost, including the cache-stampede risk when a popular prompt's cached response expires under heavy load, and how request coalescing prevents thousands of simultaneous identical generations.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cache Invalidation Across Regions](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Cache_Invalidation_Across_Regions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a distributed inference cache that minimizes GPU cost by handling repeated identical prompts efficiently, we need to address several key challenges, including avoiding cache-stampede risks and managing concurrent requests effectively. This solution leverages multiple layers of caching mechanisms, intelligent request coalescing strategies, and robust state management.

The core idea is to build a multi-tiered cache system that can handle high concurrency while ensuring consistency and reducing GPU utilization. Specifically, we will implement an LRU (Least Recently Used) cache at the frontend to filter out redundant requests, followed by a distributed cache layer for storing model outputs, and finally, a robust eviction strategy to manage cache expiration without causing stampedes.

**Comparison:**

```
Layer	Functionality
Frontend LRU	Caches recent responses to avoid redundant
Cache	GPU invocations
Distributed	Stores and retrieves model outputs across nodes,
Cache	ensuring high availability
```

**Code:**

```python
from functools import lru_cache
import time

# Simple in-memory LRU cache decorator
@lru_cache(maxsize=128)
def cached_function(prompt):
    # Simulate GPU inference delay
    time.sleep(0.5)
    return f"Inference result for prompt: {prompt}"
```

**Example input / output:**

```text
Input:
- Prompt: "What is the capital of France?"

Output:
1. Frontend LRU Cache hits -> Directly returns cached response.
2. Backend Distributed Cache misses -> Calls inference function, caches result, and stores in both frontend and backend cache.

Scenario with multiple requests:
3. 1000 identical prompts arrive simultaneously.
   - Frontend LRU Cache: ~95% of requests are cached and served directly from cache.
   - Remaining 5% calls inference, stored in distributed cache.
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Expired
6. Not Expired
7. User Request
8. Is Prompt Cached?
9. Cached Response
10. Invoke Inference
11. Result Stored in DistributedCache?
12. Respond with Stored Result
13. Infer, Store Result
14. Distributed Cache Eviction
15. Check Expiration Time
16. Re-run Inference

