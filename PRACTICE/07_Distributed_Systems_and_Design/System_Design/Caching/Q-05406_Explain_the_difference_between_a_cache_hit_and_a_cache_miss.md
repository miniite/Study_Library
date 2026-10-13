---
type: "practice"
difficulty: "Basic"
---

# Explain the difference between a cache hit and a cache miss.

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Cache Hit Ratio](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Cache_Hit_Ratio.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with caching, understanding the concepts of a "cache hit" and a "cache miss" is crucial. A cache hit occurs when the data requested by an application or user is found in the cache; conversely, a cache miss happens when the required data is not present in the cache.

This question is probing your fundamental knowledge of caching mechanisms and their impact on performance. You'll need to explain these concepts clearly and provide some context about how they work together in a caching system.

**Code:**

```python
# Example code snippet illustrating cache hit/miss

cache = {}

def fetch_data(key):
    if key in cache:
        print(f"Cache Hit for key {key}")
    else:
        print(f"Cache Miss for key {key}")
        # Simulate fetching data from a slower storage (e.g., database or network)
        data = simulate_fetch_from_slower_storage(key)
        cache[key] = data
        return data

def simulate_fetch_from_slower_storage(key):
    # Simulated function to fetch data, replace with actual implementation
    import time
    time.sleep(1)  # Simulate delay
    return f"Data for key {key}"
```

**Example input / output:**

```

Input:

PYTHON
Copy
fetch_data("user123")

Output:

CODE
Copy
Cache Miss for key user123
Data for key user123
Cache Hit for key user123

Input:

PYTHON
Copy
fetch_data("user123")

Output:

CODE
Copy
Cache Hit for key user123
```

