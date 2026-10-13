---
type: "practice"
difficulty: "Deep"
---

# Two clients send the same expensive RAG query within milliseconds and you pay for two full pipelines. How would you add in-flight request coalescing on the server so duplicate concurrent queries share one generation?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Partial Failure in Fan-out APIs](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Partial_Failure_in_Fan-out_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of in-flight request coalescing, we need to ensure that if multiple clients send the same expensive RAG (Retrieval-Augmented Generation) query within milliseconds, only a single pipeline is executed and shared among all concurrent requests. This problem can be approached using techniques like rate limiting, caching, or distributed locking mechanisms.

First, let's understand what the interviewer is probing: The focus here is on handling concurrency in REST APIs efficiently to avoid unnecessary computation and resource wastage. Key assumptions include that the RAG query results are relatively stable over a short period (milliseconds), meaning that if two requests are for the same query within this time frame, they should share the result.

Step-by-Step Solution
Identify Request Uniqueness: We need to determine whether the incoming request is unique or has already been processed.
Rate Limiting: Use rate limiting to control the frequency of requests and ensure that only one request per unique query is allowed within a short period (e.g., 500 milliseconds).
Caching Intermediate Results: Cache intermediate results to reduce redundant computations.
Distributed Locks: Implement distributed locks to ensure mutual exclusion in shared state updates, preventing race conditions.
Detailed Implementation

Rate Limiting:

Leaky Bucket Algorithm: This algorithm allows a fixed number of requests per time window and discards excess requests.
PYTHON
from time import time
from collections import deque

class LeakyBucket:
    def __init__(self, max_tokens: int, refill_rate: float):
        self.max_tokens = max_tokens
        self.tokens = max_tokens
        self.refill_time = 0.0
        self.deque = deque()

    def add_token(self):
        if time() > self.refill_time:
            self.tokens = self.max_tokens
            self.refill_time = time()
        if len(self.deque) >= self.max_tokens:
            return False

        now = time()
        while len(self.deque) and (now - self.deque[0]) > 1.0 / refill_rate:
            self.deque.popleft()

        if len(self.deque) < self.max_tokens:
            self.deque.append(now)
            return True

Caching Intermediate Results:

Use a caching layer (e.g., Redis or Memcached) to store intermediate results.
PYTHON
import redis

cache = redis.Redis(host='localhost', port=6379, db=0)

def get_query_result(query):
    cached_result = cache.get(query)
    if cached_result:
        return cached_result.decode('utf-8')
    else:
        result = generate_rag_result(query)  # Assume this function generates the RAG result
        cache.set(query, result, ex=500)  # Cache for 500 milliseconds
        return result

def generate_rag_result(query):
    # Expensive computation logic here
    pass

Distributed Locks:

Use distributed locks to ensure that only one pipeline runs at a time.
PYTHON
import redis_lock

lock = redis_lock.Lock(redis.Redis(host='localhost', port=6379, db=0), 'rag_query_lock')

def process_rag_request(query):
    if not lock.acquire(blocking=False):
        return "Query already being processed"

    try:
        result = get_query_result(query)
        # Further processing here
        return result
    finally:
        lock.release()

#### Closing Summary
and Common Mistakes

In summary, to implement in-flight request coalescing for duplicate concurrent queries, we can use a combination of rate limiting, caching, and distributed locking. Rate limiting helps control the frequency of requests, caching reduces redundant computations, and distributed locks ensure mutual exclusion.

#### Common mistakes
include:

Failing to consider edge cases like network latency or cache misses.
Not implementing proper error handling for race conditions in distributed systems.
Overlooking the need for consistency in shared state updates across multiple nodes.

