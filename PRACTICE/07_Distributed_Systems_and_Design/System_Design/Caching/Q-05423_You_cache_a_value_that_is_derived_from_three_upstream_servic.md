---
type: "practice"
difficulty: "Expert"
---

# You cache a value that is derived from three upstream services. When one upstream changes, how do you know which cached entries to invalidate without over-invalidating?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cache Invalidation](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Cache_Invalidation/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

In this scenario, we need to efficiently invalidate caching entries when an upstream service updates a value that is part of a composite key derived from three services. The goal is to minimize over-invalidating while ensuring cache coherence.

#### Triaging the Question

The question revolves around managing cache invalidation in a distributed system where data is derived from multiple upstream sources. The challenge is to invalidate only the necessary cache entries when any of these upstream services update their data, without causing unnecessary cache flushes or inconsistencies.

Key Assumptions:
We have three upstream services that contribute to a single cached value.
Each service updates its data independently and infrequently.
We use a composite key derived from outputs of all three services.
Cache invalidation should be triggered only when necessary, minimizing performance overhead.
Step-by-Step Solution:
1. Understand the Composite Key:
Define a unique composite key for each cached value that includes identifiers or outputs from the three upstream services.
For example, if the services are ServiceA, ServiceB, and ServiceC, the cache entry might be invalidated when any of these services update their respective data.
2. Versioning the Cache:
Introduce a versioning mechanism for each service’s output to track changes.
When an upstream service updates its data, it increments its version number.
The composite key can include this version number or a timestamp, ensuring that only relevant cache entries are invalidated.
3. Cache Invalidation Strategy:
Use a distributed cache like Redis with pub-sub capabilities to listen for changes from the upstream services.
Implement a custom listener that triggers invalidation logic when it receives updates.
For example, if ServiceA sends an update message, invalidate all cache entries that include any data derived from ServiceA.
4. Edge Cases and Trade-offs:
Performance Overhead: Listening for updates can introduce latency. Ensure the overhead is minimal by optimizing the pub-sub setup.
Over-Invalidation: Avoid unnecessary cache invalidations to prevent frequent cache reloads, which can impact performance.

#### Example
Implementation:
Updates Data
Updates Data
Updates Data
ServiceA
Cache Invalidation Trigger
ServiceB
ServiceC
Invalidate Cache Entries
5. Detailed Workflow:

Step 1: Define a composite key structure:

JSON
{
    "serviceAId": "unique-id-A",
    "serviceBId": "unique-id-B",
    "serviceCId": "unique-id-C"
}

Step 2: Each service maintains its version number in the cache. When a service updates, it increments this version.

Step 3: The composite key can be:

JSON
{
    "compositeKey": "A:1|B:2|C:3",
    "timestamp": "2023-10-05T14:30:00Z"
}

Step 4: When a service updates, it sends an update message to a central pub-sub system. The listener processes this message and invalidates cache entries based on the composite key.

6. Code Example (Pseudo-code):
PYTHON
from redis import Redis

redis_client = Redis()

def invalidate_cache(composite_key):
    # Invalidate all cache entries matching the composite key
    keys_to_invalidate = [f"cache:{composite_key}"]
    redis_client.delete(*keys_to_invalidate)

def listen_for_updates():
    pubsub = redis_client.pubsub()
    pubsub.subscribe('serviceA', 'serviceB', 'serviceC')

    for message in pubsub.listen():
        if message['type'] == 'message':
            update_data = message['data']
            composite_key = get_composite_key_from_update(update_data)
            invalidate_cache(composite_key)

def get_composite_key_from_update(data):
    # Extract and generate the composite key based on the update data
    return f"{data.serviceAId}:{data.serviceVersionA}|{data.serviceBId}:{data.serviceVersionB}|{data.serviceCId}:{data.serviceVersionC}"

#### Closing Summary

By implementing a versioned composite key strategy and leveraging pub-sub for real-time updates, we can efficiently invalidate cache entries without over-invalidating. This approach ensures that the cache remains coherent while minimizing performance overhead.

#### Common Mistakes
and Follow-Ups:
Mistake: Not using versioning or timestamping leads to frequent over-invalidations.

#### Follow-up
Discuss how to handle edge cases where services might not always update their data, ensuring robustness in the cache invalidation logic.

