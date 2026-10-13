---
type: "practice"
difficulty: "Intermediate"
---

# You cache anomaly scores for repeated identical telemetry signatures to cut compute, and when a popular signature's cache entry expired under load, thousands of identical scorings fired at once. Explain the cache-stampede risk and how request coalescing prevents the herd.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cache Invalidation Across Regions](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Cache_Invalidation_Across_Regions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Cache stampedes in distributed systems can occur when multiple requests hit a cache miss simultaneously after its entry has expired, leading to an overloaded backend server due to all those requests being processed at once. To mitigate this risk, request coalescing is employed as a strategy to ensure that even under high concurrency, only one instance of the backend processing runs.

To understand this in more depth, let's break down the problem and solution step by step.

**Comparison:**

```
Time (sec)	Cache State	Requests	Backend Load
0	Full	1	Low
60	Expired	1,2,3,4,5	High
```

**Code:**

```python
# Example scenario of cache stampede
cache_expiry = 60  # Cache expires after 60 seconds

def process_request(signature):
    if signature in cache:
        return cached_result
    else:
        result = compute_anomaly_score(signature)  # Expensive computation
        cache[signature] = result
        return result

# Simulated request flow with cache expiry
requests = [1, 2, 3, 4, 5]
for sig in requests:
    process_request(sig)
```

**Example input / output:**

```text
Given the following input:
- `cache_expiry` = 60 seconds
- `requests` = [1, 2, 3, 4, 5]

If at time t=60 seconds after cache expiry, all these requests hit the `process_request` function simultaneously and find that their signatures are not in the cache, they will all trigger expensive backend computations. This can overwhelm the system.
```

**Data:**

```json
{
  "cache_expiry": 60,
  "requests": [1, 2, 3, 4, 5],
  "cache": {}
}
```

**Flow:**

1. Yes
2. No
3. Cache Expiry
4. Cache Hit?
5. Cached Result
6. Compute Score and Cache
7. Backend Load
8. Multiple Requests ExpireSimultaneously
9. All Miss Cache, Compute inParallel

