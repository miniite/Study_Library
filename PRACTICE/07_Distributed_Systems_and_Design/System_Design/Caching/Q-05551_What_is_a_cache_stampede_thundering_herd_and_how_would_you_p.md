---
type: "practice"
difficulty: "Deep"
---

# What is a cache stampede (thundering herd) and how would you prevent it?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Cache Stampede Prevention](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Cache_Stampede_Prevention.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Cache stampedes, also known as thundering herds, occur when multiple requests hit a shared cache simultaneously upon its eviction or failure to serve a request due to expiration or invalidation. This can lead to a flood of backend requests, overwhelming the system and potentially causing performance issues or even a complete denial-of-service (DoS) situation.

To prevent cache stampedes, we need to understand their root causes: race conditions and the sudden influx of requests when a cache is invalidated or expired. Here’s how you can address these issues step-by-step from first principles:

Understand the Problem: When a cache entry expires or is invalidated, multiple clients might hit it at the same time, causing them to bypass the cache and go directly to the backend service.
Synchronous Approach with Locking: One common approach is to use a distributed lock before accessing the cache. This ensures that only one client can perform an expensive operation (like fetching data from the database) while others wait until the lock is released. However, this can introduce contention and reduce overall throughput.
Asynchronous Approach with Queues: Another method involves using a message queue to handle requests asynchronously. When a cache entry expires or is invalidated, you enqueue requests instead of sending them immediately. The backend processes these requests one at a time, reducing the risk of overloading.

Let’s break down each solution in more detail:

Asynchronous Approach with Queues
Cache Invalidation: When a cache entry needs to be expired, rather than directly invalidating it and causing immediate requests, you enqueue a message.
Message Queue Processing: A worker process consumes these messages from the queue and handles them one at a time, ensuring that only one backend request is generated per invalidate event.
Backend Service
Cache Invalidation
Enqueue Message

#### Cons
ume Messages
Generate Single Request
Cache Entry Expires/Invalidates
Queue
Process Worker
Service Request
Implementing the Solution

Here’s a simplified example using Python with a Redis cache and a Celery worker to handle queue processing:

PYTHON
from celery import Celery
import redis

# Initialize Celery and Redis clients
app = Celery('tasks', broker='redis://localhost:6379/0')
r = redis.Redis(host='localhost', port=6379, db=0)

@app.task
def fetch_data(key):
    # Simulate fetching data from a database or other source
    return {'data': 'fetched'}

# Example invalidation function that uses Redis and Celery
def invalidate_cache(key):
    r.delete(key)  # Delete cache entry
    fetch_data.apply_async((key,))  # Enqueue request for data

#### Closing Summary
+ Common Mistakes/Follow-Ups

By using a message queue to handle cache invalidations asynchronously, you can prevent the "thundering herd" problem. However, there are some common pitfalls:

Queue Backpressure: If the queue gets overwhelmed, it might still cause performance issues.
Latency: Asynchronous processing adds latency compared to synchronous approaches, which might not be acceptable in real-time systems.

In more complex scenarios, you might need to combine different strategies or use advanced caching techniques like region-based invalidation, where only relevant regions of the cache are invalidated, reducing the number of backend requests.

