---
type: "practice"
difficulty: "Expert"
---

# You are architecting caching for a platform that must survive a full region loss with an RPO near zero for critical reads. How do cross-region cache replication, invalidation propagation, and failover routing all fit together?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Multi-layer Caching Architecture](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Caching_Expensive_Queries/Multi-layer_Caching_Architecture.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect caching in a way that ensures zero-recovery-point-objective (RPO) for critical reads even with a full region loss, we need to carefully manage cross-region cache replication, invalidation propagation, and failover routing mechanisms. This solution involves deep coordination between multiple layers of the system to ensure data consistency across regions while maintaining high availability.

Firstly, let's clarify what each component entails:

Cross-Region Cache Replication ensures that cached data is consistently updated across different regions.
Invalidation Propagation guarantees that stale cache entries are removed when changes occur in the primary database.
Failover Routing directs read requests to a backup region if the primary region fails.

Now, let's delve into how these components interact and fit together from first principles:

Cross-Region Cache Replication

Cross-region cache replication involves copying cached data between regions. This can be done using real-time or near-real-time synchronization mechanisms. For instance, changes in one region’s cache are propagated to other regions via a messaging system like Kafka or AWS SNS/SQS. The challenge here is ensuring that the latency and throughput of this message bus do not degrade performance too much.

Invalidation Propagation

To ensure data consistency, invalidation signals must be sent out whenever an update occurs in the primary database. These signals trigger cache eviction in all relevant regions. This can be done using a dedicated invalidation service or by leveraging existing messaging infrastructure to broadcast changes. The key is designing a system that can handle high traffic and avoid cascading failures.

Failover Routing

Failover routing involves redirecting read requests to a backup region when the primary region fails. This requires robust health checks, monitoring systems, and smart load balancers capable of detecting regional outages and re-routing traffic accordingly. A common approach is to use DNS or network-level failovers that can automatically switch over to a secondary region.

Interactions Between Components

The interplay between these components ensures that data remains consistent while providing high availability:

Write Operation: When an update occurs in the primary database, it triggers invalidation signals which are propagated across all regions.
Read Operations: Normally read requests go to the nearest or most performant region. In a failover scenario, reads from failed regions are redirected to healthy ones.

#### Example
Workflow
Write to Primary Database: An update occurs in Region A.
Invalidate Cache: The system sends an invalidation signal via Kafka to all regions.
Replicate Data: Real-time replication updates caches in Regions B and C.
Health Check: If Region A fails, the health monitoring system detects this and triggers a failover.
Failover Routing: DNS or network-level load balancers re-route read requests from Region A to healthy regions.

#### Common Mistakes
and Follow-Ups
Inadequate Invalidation Propagation: Failing to propagate invalidation signals can lead to stale data in remote caches, causing issues when primary updates are applied.
Insufficient Latency Considerations: Real-time replication might not be feasible for extremely low RPOs due to network latency constraints.
Over-reliance on DNS Failovers: While simple and effective for many cases, over-relying on DNS-based failover can lead to issues with TTLs or complex routing rules.

In conclusion, a well-architected system for cross-region cache replication, invalidation propagation, and failover routing is crucial for ensuring zero-RPO reads in the face of regional failures. Each component must be carefully designed and integrated to provide seamless data consistency and high availability.

**Comparison:**

```
Component	Description
Cross-Region Cache Replication	Ensures consistent data across regions using real-time or near-real-time synchronization.
Invalidation Propagation	Signals changes in the primary database to other regions for cache eviction.
Failover Routing	Redirects read requests to healthy regions during a regional failure.
```

**Code:**

```python
# Example Python code snippet for real-time cache invalidation
import json

def send_invalidation_signal(region_id, item_key):
    # Use a message queue like AWS SNS/SQS to broadcast the signal
    message = {"region": region_id, "item_key": item_key}
    print(f"Sending invalidation signal: {json.dumps(message)}")
```

**Example input / output:**

```text
Input: Write to Primary Database (RegionA) with key "entry1"
Output:
- Invalidation Signal Sent from RegionA to Regions B and C
- Real-time Replication Updates Caches in B and C

Input: Read Request (RegionB)
Output:
- Check Health Status of RegionA -> Healthy
- Serve Cache Entry from RegionB if available; else fetch from Primary Database
```

**Data:**

```json
{
  "RegionA": {
    "CacheEntries": ["entry1", "entry2"],
    "HealthStatus": "Healthy"
  },
  "RegionB": {
    "CacheEntries": [],
    "HealthStatus": "Healthy"
  },
  "RegionC": {
    "CacheEntries": [],
    "HealthStatus": "Healthy"
  }
}
```

**Flow:**

1. Healthy
2. Unhealthy
3. Write to Primary Database
4. Health Check
5. Update Invalidation Queue
6. Replicate Data Across Regions
7. Read Request from RegionX
8. Health Status of RegionX
9. Serve Cache Entry
10. Fallback to Primary Database

