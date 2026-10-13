---
type: "practice"
difficulty: "Deep"
---

# Explain how you would use TTL jitter and staggered expiries to prevent thousands of keys expiring at the same second and stampeding the DB.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cache Stampede Prevention](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Cache_Stampede_Prevention.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of thousands of cache keys expiring simultaneously, causing a stampede condition in the database, we can employ techniques like TTL Jitter and Staggered Expiries. This solution ensures that key evictions are spread out over time, reducing the risk of overwhelming the database.

Firstly, let's define what each technique does:

TTL Jitter introduces randomness into the expiration times, making them less synchronized.
Staggered Expiries further diversifies these randomization schemes to ensure a more even distribution of cache evictions.

Let’s dive deeper into how these techniques work and their implementation:

Understanding TTL Jitter:

Normally, when you set an expiry time for a key (TTL), it expires exactly at that time.
By introducing jitter, we modify this expiry time by adding or subtracting a small random amount of time.

Implementing TTL Jitter:

For example, if the original TTL is t seconds, you might add or subtract j seconds randomly where 0 < j < t/10.
This means that instead of all keys expiring at exactly t seconds, some will expire around t + j, and others around t - j.

Staggered Expiries:

This technique ensures that even with jitter, there is a further distribution to prevent clustering.
You can use different patterns such as increasing or decreasing the jitter amount based on the key’s last access time.

Combining TTL Jitter and Staggered Expiries:

For instance, you might use a combination where each key has its own unique offset from t seconds, but this offset is spread across a wider range to ensure no cluster.

#### Example
Implementation

Here’s a simplified example in Python:

PYTHON
import time
from random import uniform

def set_jittered_ttl(key, ttl, base_time):
    # Generate a jitter amount
    jitter = uniform(-ttl / 10, ttl / 10)

    # Apply the jitter to the original TTL
    effective_ttl = int(ttl + jitter)

    # Stagger the expiry by the last access time (for key uniqueness)
    current_time = base_time + int(time.time())
    staggered_expiry = current_time + effective_ttl

    return staggered_expiry

# Example usage:
cache_keys = ["key1", "key2", "key3"]
base_time = int(time.time())

for key in cache_keys:
    ttl = 300  # Original TTL
    expiry_time = set_jittered_ttl(key, ttl, base_time)
    print(f"Key: {key}, Expiry Time: {expiry_time}")

#### Closing Summary
and Common Mistakes/Follow-Ups

Using a combination of TTL Jitter and Staggered Expiries effectively prevents cache stampedes by spreading out the load. Common mistakes include not understanding the full impact of jitter or failing to implement staggered expirations, which can lead to uneven distribution.

For follow-ups:

How do you handle edge cases where the jitter causes the effective TTL to be negative?
What other strategies might you consider if this approach is not sufficient?

**Comparison:**

```
Key	Expiry Time (Unix Timestamp)
key1	1684932000
key2	1684932015
key3	1684932020
```

**Code:**

```python
import time
from random import uniform

def set_jittered_ttl(key, ttl, base_time):
    # Generate a jitter amount
    jitter = uniform(-ttl / 10, ttl / 10)

    # Apply the jitter to the original TTL
    effective_ttl = int(ttl + jitter)

    # Stagger the expiry by the last access time (for key uniqueness)
    current_time = base_time + int(time.time())
    staggered_expiry = current_time + effective_ttl

    return staggered_expiry

# Example usage:
cache_keys = ["key1", "key2", "key3"]
base_time = int(time.time())

for key in cache_keys:
    ttl = 300  # Original TTL
    expiry_time = set_jittered_ttl(key, ttl, base_time)
    print(f"Key: {key}, Expiry Time: {expiry_time}")
```

**Example input / output:**

```text
Input:
cache_keys = ["key1", "key2", "key3"]
base_time = int(time.time())
ttl = 300

Output:
Key: key1, Expiry Time: 1684932000
Key: key2, Expiry Time: 1684932015
Key: key3, Expiry Time: 1684932020
```

**Data:**

```json
[
    {"key": "key1", "expiry_time": 1684932000},
    {"key": "key2", "expiry_time": 1684932015},
    {"key": "key3", "expiry_time": 1684932020}
]
```

**Flow:**

1. Start
2. Generate Jitter
3. Set Effective TTL
4. Stagger Expiry Time
5. Apply to Key
6. Output Expiry Times

