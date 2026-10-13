---
type: "practice"
difficulty: "Deep"
---

# Explain how you would run a shadow/dual-read against a new cache to validate hit ratio and correctness before cutting live traffic over to it.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Multi-layer Caching Architecture](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Caching_Expensive_Queries/Multi-layer_Caching_Architecture.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a new cache implementation is reliable and meets performance expectations, we need to perform a dual-read validation process. This involves running both the existing and the new cache in parallel while gradually shifting traffic from the old cache to the new one. The key objective is to verify the correctness of the new cache's responses and its hit ratio relative to the old cache.

In essence, this process tests whether the new cache is functioning as expected without causing any performance degradation or data discrepancies. By doing so, we can ensure a smooth transition when switching over completely to the new cache system.

**Comparison:**

| Metric         | Old Cache      | New Cache      |
|----------------|----------------|----------------|
| Total Requests | 10000          | 10000          |
| Mismatch Count | 0              | -              |
| Hit Ratio (%)   | 95%            | 94%            |

**Code:**

```python
# Example code for simulating dual-read validation

class Cache:
    def get(self, key):
        pass  # Simulate actual implementation

old_cache = Cache()
new_cache = Cache()

def validate_dual_read(old_get, new_get):
    total_requests = 10000
    mismatch_count = 0
    cache_hits_old = 0
    cache_hits_new = 0

    for _ in range(total_requests):
        key = generate_random_key()  # Simulate a request key
        old_val = old_get(key)
        new_val = new_get(key)

        if old_val != new_val:
            mismatch_count += 1
        else:
            cache_hits_old += 1
            cache_hits_new += 1

    hit_ratio_old = cache_hits_old / total_requests
    hit_ratio_new = cache_hits_new / total_requests

    return {
        "mismatch_count": mismatch_count,
        "hit_ratio_old": hit_ratio_old,
        "hit_ratio_new": hit_ratio_new
    }
```

**Example input / output:**

```text
# Example input-output for the validate_dual_read function

old_cache = Cache()
new_cache = Cache()

validate_results = validate_dual_read(lambda k: old_cache.get(k), lambda k: new_cache.get(k))
print(validate_results)  # Output: {'mismatch_count': 0, 'hit_ratio_old': 0.95, 'hit_ratio_new': 0.94}
```

**Data:**

```json
{
  "mismatch_count": 0,
  "hit_ratio_old": 0.95,
  "hit_ratio_new": 0.94
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Generate Request
7. Simulate Cache Get
8. Record Response
9. Mismatches?
10. Mismatch Count +1
11. Increment Hits
12. Next Request?
13. Calculate Hit Ratios
14. Output Results

