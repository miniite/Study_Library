---
type: "practice"
difficulty: "Expert"
---

# How do you keep a two-level near-cache-plus-remote-cache design coherent when one app instance's local copy goes stale after another instance writes?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Write-through and Write-back Consistency](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Write-through_and_Write-back_Consistency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

In a distributed system, maintaining coherence between a near-cache (local cache) and a remote cache is crucial to ensure that all instances have consistent data. When one app instance's local copy becomes stale due to another instance's write operation, it can lead to inconsistencies if not handled correctly. A two-level caching strategy—near-cache plus remote-cache—aims to balance performance with consistency. The key challenge is ensuring that the near-cache is updated atomically and efficiently.

#### Triage

The question probes your understanding of distributed cache coherence mechanisms and how they interact in a multi-instance system. We need to ensure that writes are propagated correctly from the remote cache back to all instances, including updating any stale local caches.

#### Assumptions
Instances: Multiple backend server instances.
Caches: Two-level caching: near-cache (local) and remote-cache (centralized).
Write Operations: Write operations can occur on any instance, potentially causing the local cache to become stale.
Read Operations: Reads should be served from the nearest available cache.

#### First Principles Solution

Identify the Stale Instance:

Detect that a write operation has occurred on another instance, making one of our local caches stale.

Determine the Remote Cache Version:

Query the remote cache to get the latest version of the data.

Update the Stale Near-Cache:

Send an update request to the instance that became stale with the new version from the remote cache.
Ensure this update is atomic and propagated correctly across all instances.

Propagate Updates Efficiently:

Use a distributed messaging system (e.g., Kafka, RabbitMQ) or a message broker for real-time updates.
Implement a consistent hashing strategy to distribute updates evenly among instances.

Handle Edge Cases:

Ensure proper handling of network partitions and transient failures.
Implement retries and fallback mechanisms for robustness.
Detailed Steps

Detection Mechanism:

Use a version number or timestamp in the cache entry to detect staleness.
When an instance detects its local cache is stale, it should query the remote cache for the latest version.

Update Request:

#### Cons
truct an update request containing the key and the new value from the remote cache.
Send this request to the instance that became stale via a reliable messaging system.

Atomic Update:

Upon receiving the update request, the target instance should atomically update its local near-cache.
Ensure all read operations from this instance now use the updated data.

Synchronization Mechanism:

Implement a synchronization mechanism to ensure that only one instance updates the cache at any given time to avoid conflicts.
Use distributed locks or a coordination service like ZooKeeper for managing concurrency.

Failure Handling:

Handle transient network failures by retrying update requests with exponential backoff.
Ensure data consistency even in the presence of partial failures using idempotent operations and versioning.

#### Closing Summary

To maintain coherence between near-cache and remote-cache in a multi-instance system, it is essential to implement an efficient and robust mechanism for detecting staleness and updating local caches. Using distributed messaging systems, consistent hashing, and synchronization mechanisms can help achieve this goal while ensuring high performance and reliability.

#### Common Mistakes/Follow-Ups
Failing to Handle Network Partitions: Not implementing retries or fallbacks can lead to data inconsistencies during network partitions.
Lack of Atomicity: Failing to ensure atomic updates can result in partial updates, leading to inconsistent state across instances.
Inadequate Synchronization Mechanisms: Missing distributed locks or coordination services can cause race conditions and conflicts.

#### Follow-Ups
How would you implement the versioning strategy for ensuring correct staleness detection?
What are some trade-offs between consistency and availability in your design?

**Code:**

```python
# Example of detecting staleness and updating local cache

import uuid

def get_version_number(cache_entry):
    return int(cache_entry['version'])

class CacheManager:
    def __init__(self, remote_cache_client):
        self.remote_cache_client = remote_cache_client

    def update_local_cache(self, key, value, version):
        # Update the local cache atomically
        if self.get_remote_version(key) > get_version_number(value):
            return False  # Stale data detected

        self.local_cache[key] = {'value': value, 'version': version}
        return True

    def get_remote_version(self, key):
        remote_data = self.remote_cache_client.get(key)
        if not remote_data:
            return 0
        return int(remote_data['version'])
```

**Data:**

```json
{
  "key": "example_key",
  "value": "new_value",
  "version": 2
}
```

**Flow:**

1. Yes
2. No
3. Instance detects staleness
4. Query Remote Cache
5. Get Latest Version
6. Update Local Cache?
7. Send Update Request to StaleInstance
8. Use Remote Data
9. Update Local Cache Atomically

