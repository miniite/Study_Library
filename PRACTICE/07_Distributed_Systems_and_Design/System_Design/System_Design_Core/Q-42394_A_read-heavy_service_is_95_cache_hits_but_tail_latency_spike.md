---
type: "practice"
difficulty: "Expert"
---

# A read-heavy service is 95% cache hits but tail latency spiked after you added a new region. How would you localize whether the cause is cache node hot-keys, cross-region replication lag, or connection-pool exhaustion?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Tail Latency After Expansion](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Tail_Latency_After_Expansion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To diagnose and resolve the issue with high tail latency in a read-heavy service after adding a new region, we need to systematically analyze potential causes such as cache node hot keys, cross-region replication lag, or connection-pool exhaustion. By understanding these factors from first principles and leveraging diagnostic tools, we can pinpoint the root cause and implement appropriate solutions.

Firstly, let's understand what each of these issues might look like:

Cache Node Hot Keys: These are keys in your cache that experience a disproportionately high number of requests, leading to contention.
Cross-Region Replication Lag: Data replication between regions may not be perfectly synchronized, causing delays when fetching data from the secondary region.
Connection-Pool Exhaustion: If too many connections are being made to databases or other services, it can lead to delays and backpressure.

To diagnose this issue, we'll start by collecting detailed metrics and logs to identify patterns that might indicate one of these issues. Here’s a step-by-step approach:

Collect Metrics:

Cache hit rate per cache node
Latency distribution for read operations in each region
Number of cross-region requests and their latency
Connection pool utilization and failure rates

Analyze Metrics:

Identify if any specific cache nodes are under heavy load (hot keys)
Check the distribution of read latencies to see if there’s a sudden increase in tail latency
Analyze cross-region request patterns and their impact on overall latency

Investigate Hypotheses:

Cache Node Hot Keys: If certain cache nodes are consistently under high load, this could be causing the increased tail latency.
Cross-Region Replication Lag: Examine if there’s a significant delay in fetching data from other regions and how often this happens.
Connection-Pool Exhaustion: Check for any connection failures or timeouts that might indicate resource exhaustion.

Implement Diagnostics:

Introduce diagnostic logging to capture the exact operations causing high tail latency.
Use distributed tracing tools like Jaeger or Zipkin to trace requests and identify bottlenecks.
Monitor database performance metrics such as query times, connection counts, and slow queries.

#### Solution
s Based on Diagnosis:

If it’s hot keys in the cache, consider caching strategies that distribute load more evenly or use advanced techniques like adaptive caching.
For replication lag, optimize data replication mechanisms to ensure faster synchronization.
Address connection-pool issues by tuning pool sizes and timeouts, or implementing connection pooling with better error handling.

**Comparison:**

```
Metric	Region A	Region B
Cache Hits/Node	Node1: 980, Node2: 965	Node3: 970, Node4: 950
Read Latency	Avg: 3ms, Max: 5ms	Avg: 400ms, Max: 700ms
```

**Code:**

```python
# Example code for logging hot keys in a cache node
def log_cache_access(cache_key):
    if is_hot_key(cache_key):  # Assume this function checks if the key is hot
        logger.info(f"Hot key accessed: {cache_key}")
```

**Example input / output:**

```text
Input:
- Read request to region A
Output:
- Cache hit in node1 with latency of 3ms

Input:
- Cross-region read request from region B to region A
Output:
- Latency of 800ms due to replication lag
```

**Data:**

```json
{
  "metrics": {
    "cache_hits_per_node": {"node1": 980, "node2": 965},
    "read_latency_distribution": [{"region1": [3ms, 5ms], "region2": [400ms, 700ms]}],
    "cross_region_requests": {"count": 100, "latency_avg": 800ms}
  },
  "logs": [
    "Hot key accessed: user_id_12345",
    "Connection failure detected: db_connection_pool_exhausted"
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Start
8. Is cache node under heavyload?
9. Identify and adjust hot keys
10. Is cross-region replication lagsignificant?
11. Tune replication strategy
12. Is connection pool exhausted?
13. Tune or expand connectionpool
14. Diagnose and resolve otherissues
15. End

