---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between eviction and expiration in a cache?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Eviction Policies and TTL](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Eviction_Policies_and_TTL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In the context of caching, understanding the differences between eviction and expiration is crucial to managing memory efficiently and ensuring data freshness. Eviction refers to the process where cached items are removed from the cache based on certain policies, such as least recently used (LRU) or least frequently used (LFU). Expiration, on the other hand, involves setting a time-to-live (TTL) for cached items so that they automatically get invalidated after a specified duration. Let's delve into these concepts step by step.

**Code:**

```python
# Example of cache eviction and expiration in Python using functools.lru_cache

from functools import lru_cache
import time

@lru_cache(maxsize=10)  # Eviction based on max size (LRU)
def compute_heavy(x):
    print(f"Computing {x}")
    return x * x

# Example of cache expiration
import datetime

def expiring_cache(expiration_time):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            now = datetime.datetime.now()
            if func.cache_info().misses == 0 and (now - func.cache_clear_time).total_seconds() > expiration_time:
                return func(*args, **kwargs)  # Cache expired, compute again
            else:
                return func.cache_value  # Return cached value
        return wrapper
    return decorator

@expiring_cache(expiration_time=5)
def heavy_compute(x):
    print(f"Computing {x}")
    time.sleep(2)  # Simulate heavy computation
    return x * x

# Example usage:
print(compute_heavy(10))  # First call, will compute and cache the result
time.sleep(30)            # Sleep for over 30 seconds to demonstrate expiration
print(compute_heavy(10))  # Second call within eviction limit, will reuse cached value
```

