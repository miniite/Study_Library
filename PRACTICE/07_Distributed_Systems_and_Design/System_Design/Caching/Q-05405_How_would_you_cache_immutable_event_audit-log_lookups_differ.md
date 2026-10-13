---
type: "practice"
difficulty: "Deep"
---

# How would you cache immutable event/audit-log lookups differently from mutable entity records?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Eviction Policies and TTL](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Eviction_Policies_and_TTL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When caching immutable event or audit-log lookups versus mutable entity records, we need to consider different caching strategies and eviction policies based on the nature of the data. For immutable event logs, which rarely change once created, we can leverage more aggressive caching techniques to minimize read latency. Conversely, for mutable entity records that frequently update, a careful balance between freshness and performance is crucial.

To begin, let's summarize the core solution: we will use Least Recently Used (LRU) or Time-to-Live (TTL) caches for both types of data but with different configurations tailored to their distinct characteristics. The key idea is to maximize efficiency while ensuring that stale data doesn't linger too long in a cache.

**Comparison:**

| Cache Type | Characteristics | Configuration Example |
|------------|-----------------|-----------------------|
| Event Log  | Immutable       | LRU Cache (max_size=1000) |
| Entity     | Mutable         | TTL Cache (max_size=500, ttl_seconds=60) |

**Code:**

```python
# Example LRU Cache Implementation using Python's collections module
from collections import LRUCache

class EventLogCache:
    def __init__(self, max_size):
        self.cache = LRUCache(max_size)

    def get(self, key):
        return self.cache.get(key, None)

    def set(self, key, value):
        self.cache[key] = value

# Example TTL Cache Implementation
import time

class TTLCache:
    def __init__(self, max_size, ttl_seconds):
        self.cache = {}
        self.max_size = max_size
        self.ttl_seconds = ttl_seconds
        self.timestamp = {}

    def get(self, key):
        if key in self.cache and self._is_valid(key):
            return self.cache[key]
        return None

    def set(self, key, value):
        now = time.time()
        self.cache[key] = (now + self.ttl_seconds, value)
        self.timestamp[key] = now

    def _is_valid(self, key):
        if key not in self.timestamp:
            return False
        if self.timestamp[key] < time.time():
            del self.cache[key]
            del self.timestamp[key]
            return False
        return True
```

**Example input / output:**

| Input | Output |
|-------|--------|
| `get_event_log(123)` with cache hit | Event log data from cache |
| `get_event_log(456)` with cache miss | No event log found, fetch and store in LRU cache |
| `set_entity_record(789, "new record")` with TTL cache | Record set; entry added to TTL cache |

**Data:**

```json
{
  "event_log_cache": {
    "max_size": 1000,
    "ttl_seconds": null
  },
  "entity_record_cache": {
    "max_size": 500,
    "ttl_seconds": 60
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Cache Lookup
6. LRU Cache hit?
7. Store in LRU Cache
8. TTL Cache hit?
9. Fetch from DB, store in TTLand LRU cache
10. TTL Cache expires entry, fetchfrom DB, update LRU cache

