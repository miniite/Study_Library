---
type: "practice"
difficulty: "Intermediate"
---

# You cache a value with TTL 60s. A request comes at second 59. What are the possible outcomes and why?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Eviction Policies and TTL](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Eviction_Policies_and_TTL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you cache a value with a Time-To-Live (TTL) of 60 seconds, it means that the cache will expire after 60 seconds from the time it was inserted. In this scenario, a request comes in at second 59. We need to consider several factors to determine the possible outcomes.

#### Triaging the Question

The core of the question is understanding what happens when a request arrives just before or just after the cache's expiration. The interviewer is probing your knowledge of cache behavior and how it interacts with time-based operations. Key assumptions include:

The TTL is 60 seconds.
The cache was inserted at second 0, making it expire at second 60.

#### First Principles Solution
Cache Insertion: Let's assume the cache was inserted at second 0. This means the cache will be available until second 60.
Request Arrival at Second 59: Since the request arrives just one second before the expiration, we need to consider two scenarios:
The behavior of the cache when a request comes in just before its TTL expires.
How long does it take for the cache to expire after receiving a request.
Scenario 1: Cache Expiration Before Request Handling
Scenario: If the cache is set to expire immediately upon request, the value might be considered expired and a new retrieval or computation would be required. However, in most caching implementations, this scenario is unlikely because caches typically serve requests until they are explicitly invalidated.
Scenario 2: Cache Validity Until Request Handling Completes
Scenario: More commonly, if the cache receives a request at second 59, it will remain valid throughout the handling of that request. The cache will not expire during this process and can be used for subsequent operations until second 60.

#### Conclusion

Given typical caching behavior:

Outcome: The cached value should still be available when the request is handled at second 59.
Explanation: Most caching systems are designed to ensure that a request arriving before the TTL expires will use the current cache contents. The cache does not expire mid-request, but only after the entire process completes.

#### Closing Summary
and Common Mistakes/Follow-Ups

In most cases, the cache value should still be available for use when the request arrives at second 59. However, it’s important to consider edge cases like network latency or delays in processing that could affect this outcome.

#### Common Mistakes

Assuming the cache will expire mid-request.
Ignoring the possibility of a new retrieval if the cache is set up to expire on request arrival.

#### Follow-Ups
The interviewer might ask about specific caching strategies, such as conditional refreshes or how to handle different types of TTL (absolute vs. sliding).

**Code:**

```python
# Example of handling a cache with a 60-second TTL in Python using Flask and Redis

from flask import Flask
import redis

app = Flask(__name__)
redis_client = redis.StrictRedis(host='localhost', port=6379, db=0)

@app.route('/get_value')
def get_value():
    key = 'example_key'

    # Check if the cache is still valid
    cached_value = redis_client.get(key)
    if cached_value:
        return f"Cached Value: {cached_value.decode('utf-8')}"
    else:
        # Simulate an expensive operation and store in cache with 60-second TTL
        value = "Expensive Computation Result"
        redis_client.setex(key, 60, value)
        return f"Computed Value: {value}"
```

**Data:**

```json
{
    "key": "example_key",
    "ttl": 60,
    "value": "Expensive Computation Result"
}
```

