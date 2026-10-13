---
type: "practice"
difficulty: "Deep"
---

# Describe how you would introduce request coalescing at the service layer and prove it reduced DB load during a stampede in staging.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cache Stampede Prevention](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Cache_Stampede_Prevention.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To mitigate the "stampede" effect, we need to implement request coalescing at the service layer, which will help reduce the number of redundant database queries by consolidating multiple requests into a single batch during high traffic periods. We'll use an in-memory cache with time-to-live (TTL) and concurrency control mechanisms to ensure data consistency.

#### Triage

The question is asking us to design a strategy for request coalescing at the service layer to reduce database load during heavy traffic, specifically focusing on the "stampede" effect where many requests are issued almost simultaneously. The goal is to prove that this approach can effectively mitigate the number of redundant database queries and thus reduce overall DB load.

#### First-Principles Solution
Identify the Problem: High traffic leading to a sudden influx of requests hitting the service layer, causing a rapid increase in database access.
Implement Request Coalescing:
Use an in-memory cache (e.g., Redis) to store and manage frequently accessed data.
Implement a request queue to buffer incoming requests and process them in batches.
Concurrency Control: Ensure that the system can handle concurrent operations without causing data inconsistency.
Monitoring and Validation: Set up monitoring to track database load before and after implementing coalescing.
Step-by-Step Solution
1. In-Memory Cache with TTL

Cache Design:

Fetch Data
Trigger
Database
Cached Data
TTL Expiration
Evict Cached Data

#### Example
Cache Schema:

Key: user_id
Value: JSON containing user data
2. Request Coalescing

Queue Implementation:

PYTHON
from collections import deque
import time

class RequestQueue:
    def __init__(self, max_size=100):
        self.queue = deque(maxlen=max_size)

    def add_request(self, request):
        self.queue.append(request)
        # Optionally limit the rate of requests to avoid overwhelming the cache
        if len(self.queue) > 50:  # Example threshold
            time.sleep(0.1)  # Simple backoff strategy

    def process_batch(self):
        batch = list(self.queue)
        self.queue.clear()
        return batch

#### Example
Workflow:

A request comes in and is added to the queue.
Periodically, a batch of requests is processed from the queue.
3. Concurrency Control

Using Semaphores or Locks:

PYTHON
from threading import Lock

class SafeQueue:
    def __init__(self):
        self.lock = Lock()
        self.queue = deque()

    def add_request(self, request):
        with self.lock:
            self.queue.append(request)

    def process_batch(self):
        batch = list(self.queue)
        self.queue.clear()
        return batch
4. Monitoring and Validation

Setup Metrics:

Monitor database query counts before and after implementing coalescing.
Use APM tools to track request latency, error rates, and cache hit ratios.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Implementing request coalescing involves multiple layers of caching and queuing mechanisms. It’s crucial to choose the right in-memory store with appropriate TTL settings and concurrency control strategies to ensure data consistency and performance. Common mistakes include not setting proper thresholds for queue backlog, failing to handle edge cases like network failures, and not properly validating the effectiveness through rigorous testing.

#### Follow-ups
might involve discussing:

How to handle different types of requests (read-heavy vs write-heavy).
Strategies for handling stale cache data.
Integrating with existing logging and monitoring tools.

**Comparison:**

```
Metric	Before	After
Database Query Count (req/hr)	10,000	5,000
Cache Hit Ratio	N/A	85%
Average Latency	40ms	2ms
```

**Code:**

```python
from collections import deque
import time

class RequestQueue:
    def __init__(self, max_size=100):
        self.queue = deque(maxlen=max_size)

    def add_request(self, request):
        self.queue.append(request)
        # Optionally limit the rate of requests to avoid overwhelming the cache
        if len(self.queue) > 50:  # Example threshold
            time.sleep(0.1)  # Simple backoff strategy

    def process_batch(self):
        batch = list(self.queue)
        self.queue.clear()
        return batch

class SafeQueue:
    def __init__(self):
        self.lock = Lock()
        self.queue = deque()

    def add_request(self, request):
        with self.lock:
            self.queue.append(request)

    def process_batch(self):
        batch = list(self.queue)
        self.queue.clear()
        return batch
```

**Example input / output:**

```text
Before Coalescing:
- Database Query Count: 10,000 requests/hour

After Coalescing:
- Database Query Count: 5,000 requests/hour
- Cache Hit Ratio: 85%
- Average Latency: 2ms vs 40ms before
```

**Data:**

```json
{
  "user_id": 12345,
  "username": "john_doe",
  "email": "johndoe@example.com"
}
```

**Flow:**

1. Request Arrival
2. Queue Add Request
3. Batch Processed
4. E
5. DB Query
6. Process Response
7. D
8. Cached Data Returned

