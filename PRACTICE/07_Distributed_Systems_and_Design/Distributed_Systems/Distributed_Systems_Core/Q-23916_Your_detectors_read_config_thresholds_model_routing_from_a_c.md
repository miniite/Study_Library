---
type: "practice"
difficulty: "Intermediate"
---

# Your detectors read config (thresholds, model routing) from a config store on the hot path and a store blip spiked scoring latency fleet-wide. Explain how you'd cache config locally with a TTL and safe fallback so the config store is never in the critical detection path.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Shared Config as Single Point of Failure](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Shared_Config_as_Single_Point_of_Failure.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need to ensure that our distributed system can efficiently handle configuration updates while minimizing latency during critical detections. The core idea is to cache configurations locally with a Time-to-Live (TTL) and provide a safe fallback mechanism in case the local cache expires or becomes stale.

#### Triaging the Question

The question requires us to design a caching strategy for configuration data used by detectors, ensuring that these detectors do not experience performance degradation due to frequent updates in the config store. The key points are:

Config Cache: Store configurations locally with a TTL.
Safe Fallback Mechanism: Ensure there's a reliable fallback option if the local cache is invalid or unavailable.

#### First Principles Solution
Local Caching: Implement caching for configuration data, leveraging an in-memory store like Redis or Memcached.
TTL Management: Set a reasonable TTL to balance freshness and resource consumption.
Safe Fallback: Establish a fallback mechanism that minimizes the impact on detection performance.
Step-by-Step Solution
1. In-Memory Cache Setup

Use an in-memory store like Redis for local caching of configurations. Redis is chosen due to its high performance, reliability, and ease of use for cache management.

Detector
Local Cache (Redis)
2. TTL Implementation

Set a TTL on cached entries based on the expected frequency of config updates. For example, if configurations are updated every 15 minutes, setting a TTL of 10-15 minutes would be appropriate.

B
Set TTL: 10-15 mins
3. Safe Fallback Mechanism

Implement a safe fallback mechanism to ensure that even if the local cache expires, detection performance is not severely impacted. This could involve using a remote read-only replica or fetching fresh data from a backup store.

B
Safe Fallback: RemoteReplica/Backup Store
4. Monitoring and Notification

Set up monitoring to notify when the local cache expires, allowing for proactive refreshes if necessary.

C
Monitoring & Notifications

#### Closing Summary
+ Common Mistakes/Follow-Ups

Summary: By implementing a local caching strategy with TTL management and a safe fallback mechanism, we can ensure that our detectors remain performant even during frequent configuration updates. Monitoring and proactive measures are crucial to maintain system health.

#### Common Mistakes/Follow-Ups

Incorrect TTL Settings: Setting the TTL too short can cause frequent cache invalidations, leading to increased latency.
Over-reliance on Fallback Mechanism: Not having a robust fallback mechanism or using an unreliable one can lead to degraded performance.
Lack of Monitoring: Failing to set up proper monitoring and notifications can result in undetected issues during critical times.

#### Example
Code Block

Here’s a basic example of how you might implement this in Python:

PYTHON
import redis

def get_config_from_redis(key):
    r = redis.Redis(host='localhost', port=6379, db=0)
    config = r.get(key)
    if not config:
        # Fallback to remote store or backup cache
        config = fetch_from_remote_store(key)
        r.set(key, config, ex=600)  # TTL of 10 minutes
    return config

def fetch_from_remote_store(key):
    # Implement this function to retrieve data from a reliable fallback source
    pass

#### Conclusion

This solution ensures that the config store is not part of the critical path for detectors, providing a balanced approach between performance and reliability.

