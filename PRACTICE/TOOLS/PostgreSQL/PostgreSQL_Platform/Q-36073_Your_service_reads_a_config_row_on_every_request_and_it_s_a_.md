---
type: "practice"
difficulty: "Deep"
---

# Your service reads a config row on every request and it's a Postgres hotspot; how would you decide between a materialized cache in-process, LISTEN/NOTIFY invalidation, and a short TTL, and how do you avoid stampedes?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Replica Lag and Read Your Writes](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Replica_Lag_and_Read_Your_Writes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a PostgreSQL hotspot caused by frequent reads from your service's config row, we need to decide between implementing an in-process cache, using LISTEN/NOTIFY for invalidation, or employing short TTLs with periodic refreshes. Each approach has its trade-offs and implications, especially concerning performance, scalability, and consistency.

Firstly, let’s clarify the problem: a high volume of requests are hitting the database to read a config row that rarely changes but must be up-to-date at all times. This situation requires a robust strategy to ensure timely updates while minimizing load on the database.

In-Process Cache: This approach involves storing the config data in memory within your application, reducing the need for frequent database queries.

#### Pros
Direct access to cached data improves latency and reduces database load.

#### Cons
If the cache is invalidated, it can lead to stale data being served until the next refresh.

LISTEN/NOTIFY: This method uses PostgreSQL’s notification system to notify clients of changes in the database.

#### Pros
It ensures that any changes made to the config row will be immediately reflected across all connected clients without manual polling.

#### Cons
Implementing LISTEN/NOTIFY can introduce complexity, especially if you need to handle concurrent connections and ensure proper cleanup.

Short TTL with Periodic Refreshes: This involves setting a short time-to-live for cached data and periodically fetching the latest configuration from the database.

#### Pros
Simplifies implementation compared to LISTEN/NOTIFY, and ensures that old data is eventually refreshed.

#### Cons
May introduce periodic delays as stale data is served during refresh times.

To decide between these options, we need to evaluate the trade-offs based on the expected frequency of config changes, the criticality of real-time updates, and the acceptable level of staleness.

First-principles reasoning:

Frequency of Changes: If the configuration rarely changes (e.g., once a month), an in-process cache or short TTL with periodic refreshes may be sufficient.
Critical Real-Time Updates: For highly critical applications where real-time updates are essential, LISTEN/NOTIFY might be the best choice despite added complexity.
System Scalability and Simplicity: If your system has a large number of nodes or clients, an in-process cache can simplify deployment and maintenance compared to managing LISTEN/NOTIFY across all instances.

Avoiding stampedes involves implementing proper synchronization mechanisms within the application to handle concurrent access to the cached data and database. For example:

Use atomic operations like SELECT FOR UPDATE when fetching from the database.
Implement a mutex or semaphore to control access during cache invalidation.

By carefully considering these factors, you can choose the most appropriate strategy for your use case. Common mistakes include over-relying on LISTEN/NOTIFY without proper error handling and not adequately managing stale data in TTL-based systems.

**Code:**

```python
# Example of a simple in-process cache with periodic refresh

import time
from threading import Lock, Timer

class ConfigCache:
    def __init__(self, db_query_func):
        self.cache = None
        self.last_update_time = 0
        self.ttl = 60  # seconds
        self.lock = Lock()

    def get_config(self):
        with self.lock:
            current_time = time.time()

            if not self.cache or (current_time - self.last_update_time) > self.ttl:
                # Fetch from database
                self.cache = db_query_func()
                self.last_update_time = current_time

                # Schedule next refresh
                Timer(self.ttl, self.get_config).start()

            return self.cache
```

**Example input / output:**

```text
Input:
- Request for config row

Output:
- Current configuration data (from cache or database)
```

**Flow:**

1. No
2. Yes
3. Initial request
4. Cache Hit?
5. Fetch from database
6. Update cache and notifylisteners
7. Return config data
8. Return cached config data

