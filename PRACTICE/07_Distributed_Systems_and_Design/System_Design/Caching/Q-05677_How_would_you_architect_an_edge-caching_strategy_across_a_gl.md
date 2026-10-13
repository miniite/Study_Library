---
type: "practice"
difficulty: "Expert"
---

# How would you architect an edge-caching strategy across a global CDN plus your own regional caches so that a single purge event propagates deterministically everywhere within an SLA you can defend?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Multi-layer Caching Architecture](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Caching_Expensive_Queries/Multi-layer_Caching_Architecture.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect an edge-caching strategy that ensures deterministic propagation of cache purges globally, we need to consider multiple layers including our regional caches and a global CDN. The primary challenge is ensuring consistent and timely updates across all cache nodes without violating service level agreements (SLAs). Here’s how we can approach this:

#### Overview
We'll design a caching architecture that includes both regional edge caches and a global CDN, with mechanisms to ensure deterministic purge propagation. This involves using a centralized purging mechanism, implementing versioning for cache keys, and leveraging redundant network paths.

#### Triage
The core of the problem is ensuring consistency in cache updates across distributed nodes. We need to design an efficient update strategy that minimizes latency and maximizes reliability while adhering to SLAs. Key assumptions include having a reliable message broker service for communication and consistent timestamps for purges.

#### First-Principles Solution
Centralized Purging Service: Implement a central service responsible for initiating purge requests. This service will act as the single source of truth for cache invalidations.
Versioning Cache Keys: Use versioned cache keys to differentiate between different states or revisions of cached content. This allows us to maintain historical versions without overwriting them, ensuring that we can always revert to previous states if necessary.
Global Message Broker: Utilize a reliable message broker (e.g., RabbitMQ, Kafka) for distributing purge events from the central service to all regional edge caches and CDN nodes. The message should include the versioned cache key and any relevant metadata.
Redundant Network Paths: Ensure that there are multiple redundant network paths between the central purging service and all cache nodes to minimize single points of failure and latency.

#### Closing Summary
+ Common Mistakes/Follow-Ups:
The architecture should be resilient against failures in any individual node or network segment, ensuring high availability.
It's crucial to regularly monitor and test the purge propagation to validate SLAs and make adjustments as necessary.

#### Follow-ups
could include discussing how to handle large-scale purges, potential bottlenecks in the message broker, and how to integrate this with existing security protocols.

**Comparison:**

```
Cache Node	Message Received	Action Taken
Regional Edge Cache A	Yes	Invalidate cache
Global CDN B	Yes	Invalidate cache
Regional Edge Cache C	No (latency)	Wait for retry
```

**Code:**

```python
# Example of a simplified purging service
import uuid

class PurgeService:
    def __init__(self):
        self.cache_keys = {}

    def initiate_purge(self, cache_key, version):
        self.cache_keys[cache_key] = version
        # Simulate message sending to global broker
        send_to_broker(cache_key, version)

def send_to_broker(cache_key, version):
    print(f"Sending purge event for {cache_key} with version {version}")
```

**Example input / output:**

```text
Input: PurgeService().initiate_purge("example_resource", 1)
Output: Sending purge event for example_resource with version 1
```

**Data:**

```json
{
  "purge_event": {
    "cache_key": "example_resource",
    "version": 1,
    "timestamp": "2023-10-05T14:48:00Z"
  }
}
```

**Flow:**

1. Central Purging Service
2. Initiate Purge
3. Publish to Broker
4. Regional Edge Cache 1
5. Global CDN
6. Regional Edge Cache 2
7. Update Cache
8. Update Cache
9. Update Cache

