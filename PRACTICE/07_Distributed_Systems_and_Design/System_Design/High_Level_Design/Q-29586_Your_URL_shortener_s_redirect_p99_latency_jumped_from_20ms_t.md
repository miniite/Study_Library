---
type: "practice"
difficulty: "Deep"
---

# Your URL shortener's redirect p99 latency jumped from 20ms to 400ms in one region — walk through how you'd diagnose whether it's cache miss rate, DB hotspot, or replication lag.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Fan-out Trade-offs](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Fan-out_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To diagnose why your URL shortener's redirect p99 latency jumped from 20ms to 400ms in one region, we need to systematically evaluate potential issues like cache miss rate, database hotspotting, or replication lag. Let’s break down the problem step by step using first principles.

Firstly, understand that a high p99 latency jump typically indicates an extreme case of latency spikes rather than normal operation. This suggests something unusual is occurring in our system. We need to ensure we are looking at the right metrics and have accurate data about where the latency is coming from.

#### Triage
What's Being Probed

The interviewer is probing your ability to methodically diagnose performance issues, understand cache behavior, database operations, and network replication delays. You’ll be expected to reason through potential causes, identify relevant metrics, and perform a logical analysis of each component in the URL shortener system.

#### Assumptions
The latency jump happened suddenly and specifically in one region.
We are using a distributed system with multiple caches and databases across regions.
There is an existing monitoring setup that captures p99 latencies for both cache hits and misses, database access times, and network delays.
Step-by-Step Diagnosis
1. Cache Miss Rate Analysis

The first step is to check the cache miss rate in that region. High cache miss rates can significantly increase latency as every request now hits the backend instead of a fast cache lookup.

PLAINTEXT
# Example Cache Metrics
Cache Miss Rate:
Before: 5% (normal)
Now: 80% (abnormally high)

This indicates that most requests are not hitting our local cache, forcing a slower database fetch.
2. Database Hotspotting

Next, analyze the database metrics to see if there is any hotspotting leading to increased latency. This involves checking for:

High contention on specific rows or tables.
Slow queries running more frequently than usual.
PLAINTEXT
# Example Database Metrics
Database Query Timeouts:
Before: 0% (normal)
Now: 25%

This suggests that certain database operations are taking longer, potentially due to increased load or resource contention.
3. Replication Lag Analysis

Replication lag can cause delays in data consistency across regions. We need to check the replication status and latency between nodes.

PLAINTEXT
# Example Replication Metrics
Replication Latency:
Before: 20ms (normal)
Now: 100ms (significantly high)

This increase could indicate that updates are not propagating quickly enough, leading to inconsistencies.

#### Closing Summary

By systematically analyzing cache hit rates, database query performance, and replication lag, we can pinpoint the root cause of the increased latency. If cache miss rates are significantly higher, it’s likely due to a misconfiguration or issue with our caching strategy. If database operations are timing out more frequently, there may be issues like slow queries or resource contention. And if replication lag is high, network delays or resource constraints could be the culprits.

#### Common Mistakes
and Follow-Ups
Failing to collect enough data: Ensure you have accurate and complete metrics before drawing conclusions.
Overlooking distributed nature: Assume that issues are local when they might affect multiple components.
Ignoring external factors: Consider network latency, DNS resolution times, or other environmental variables.

By methodically diagnosing each component, we can identify the true cause of increased latency and take appropriate actions to mitigate it.

