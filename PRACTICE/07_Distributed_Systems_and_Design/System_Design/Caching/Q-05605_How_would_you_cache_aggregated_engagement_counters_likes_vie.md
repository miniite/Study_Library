---
type: "practice"
difficulty: "Deep"
---

# How would you cache aggregated engagement counters (likes, views) with high write rates by batching increments in memory and periodically flushing, while surviving process crashes?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cache Failover and Resilience](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Cache_Invalidation/Cache_Failover_and_Resilience.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address caching aggregated engagement counters like likes and views with high write rates, we need to design a system that can efficiently batch updates in memory, periodically flush them to a persistent cache or database, and ensure robustness even if processes crash. This involves careful consideration of data consistency, performance, and fault tolerance.

The core idea is to use an in-memory data structure for batching updates, followed by periodic flushing to the main cache layer (e.g., Redis) or directly to storage. We must also account for the possibility of crashes and ensure that any unflushed updates are handled gracefully upon restart.

**Code:**

```

import time from threading import Lock

class AggregatedEngagementCounter: def init(self, key): self.key = key self.lock = Lock() self.local_cache = 0 self.unflushed_updates = 0

CODE
Copy
def increment(self, delta=1):
    with self.lock:
        self.local_cache += delta
        self.unflushed_updates += delta

def flush(self):
    global counter_store
    with self.lock:
        if self.unflushed_updates > 0:
            counter_store[self.key] = self.local_cache + self.unflushed_updates
            self.local_cache = 0
            self.unflushed_updates = 0
CODE
Copy

```python
import threading
import time

class CacheFlusher(threading.Thread):
    def __init__(self, interval=60, counter_store=None):
        super().__init__()
        self.interval = interval
        self.counter_store = counter_store
        self.stopped = False

    def run(self):
        while not self.stopped:
            time.sleep(self.interval)
            with self.lock:
                for key, value in list(counter_store.items()):
                    AggregatedEngagementCounter.flush(key)

    def stop(self):
        self.stopped = True
PYTHON
Copy
def handle_write_request(key, delta=1):
    counter = AggregatedEngagementCounter(key)
    counter.increment(delta)
```

**Example input / output:**

```text
No input-output examples are necessary here as the focus is on design and architecture rather than specific requests and responses.
```

**Flow:**

1. Yes
2. Yes
3. AggregatedEngagementCounter Increment
4. Is Local Cache Full?
5. Batch Update in Memory
6. Unflushed Updates > 0
7. Flush to Main Cache/Storage
8. Clear Local Cache andUnflushed Updates
9. Handle Write Request

