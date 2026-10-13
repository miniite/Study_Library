---
type: "practice"
difficulty: "Intermediate"
---

# Your cache uses TTL-based expiry but you observe correlated expiries causing periodic DB spikes. How do you diagnose that pattern and apply jitter without losing freshness guarantees?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cache Stampede Prevention](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Cache_Stampede_Prevention.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a caching system where TTL-based expiry is causing periodic database (DB) spikes, it's crucial to understand both the root cause and how to mitigate it while maintaining cache freshness. The key steps involve diagnosing the pattern of expirations, applying jitter to spread out these expirations, and validating that this approach doesn't compromise data freshness.

First, let's break down what we're dealing with: a caching system where the TTL (Time To Live) is set for each cached item. This results in periodic cache misses, which should ideally be handled by fetching fresh data from the database. However, if these cache misses occur at regular intervals, they can cause repeated and unneeded queries to the DB, leading to performance issues.

To diagnose this pattern, you need to:

Collect Metrics: Gather detailed metrics on cache hits, cache misses, and times of DB spikes.
Analyze Logs: Review logs for patterns in when DB access is being triggered.
Visualize Data: Use tools like Prometheus or Grafana to visualize the frequency and timing of cache expirations.

Once you have a good understanding of the pattern, you can apply jitter to spread out the cache expiration times. Jitter involves adding a small random delay to the TTL value, ensuring that all cached items do not expire at exactly the same time.

Here's how to implement this step-by-step:

Understand the Current Expiry Mechanism: Typically, your caching system sets an absolute expiry time for each item (e.g., 5 minutes from now). Jitter involves adding a random delay in the range of 0 to some maximum value (e.g., 30 seconds).

Calculate Jitter: For each cache entry, generate a random number within a specified jitter range and add it to the original TTL.

Update Expiry Times: Store these updated expiry times with the cached items, ensuring that they are still fresh but not all expiring at once.

Monitor Impact: After implementing jitter, continuously monitor the cache hit rate and DB query frequency to ensure that freshness guarantees are maintained without causing unnecessary load on the database.

Tune Jitter Parameters: Fine-tune the jitter range based on performance metrics. Too much jitter can lead to stale data, while too little might not provide enough relief from periodic spikes.

By following these steps, you can effectively mitigate DB spikes caused by correlated expirations without compromising cache freshness guarantees.

