---
type: "practice"
difficulty: "Intermediate"
---

# What metrics (hit ratio, eviction rate, p99 latency, memory) would you graph on a cache dashboard and why?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Cache Hit Ratio](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Cache_Hit_Ratio.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively monitor a caching system, we need to understand both its performance and efficiency. For a Backend SDE-2/3 role, key metrics like hit ratio, eviction rate, p99 latency, and memory usage are crucial. These metrics provide insights into how well the cache is performing in terms of hit efficiency, how effectively it's managing data, and overall resource utilization.

The question is probing your understanding of caching principles and your ability to measure its effectiveness. By focusing on these specific metrics, you can ensure that the cache system meets service level agreements (SLAs) and supports the application’s performance needs.

#### Triaging the Question

Metrics to Graph:

Hit Ratio
Eviction Rate
p99 Latency
Memory Usage

#### Assumptions

The cache system is designed for a high-concurrency, read-heavy workload.
The application has defined performance SLAs that need to be met.

#### First Principles Solution

Hit Ratio (Cache Hit Rate): This metric measures the proportion of requests that are served by the cache without needing to hit the underlying storage. A higher hit ratio indicates better caching efficiency, as more reads can be handled quickly.

Eviction Rate: Evictions refer to the number of items evicted from the cache per request. High eviction rates might indicate that too many entries are being replaced in the cache, leading to frequent misses. Monitoring this helps ensure that the cache is not overpopulated and that it's optimizing for hit rate.

p99 Latency: p99 latency measures the 99th percentile of response times. This metric is critical because while average latencies can be misleading, p99 gives a clearer picture of how well the system handles peak loads. High p99 latencies indicate that occasional long delays are happening, which could degrade user experience.

Memory Usage: Memory usage metrics help ensure that the cache doesn't consume too much memory and cause performance issues due to excessive resource consumption. This is particularly important when dealing with constraints on server resources.

#### Closing Summary

In summary, these metrics collectively provide a comprehensive view of the caching system's health. By monitoring hit ratio, eviction rate, p99 latency, and memory usage, you can ensure that your cache operates efficiently, meets performance SLAs, and avoids resource bottlenecks.

#### Common Mistakes/Follow-Ups

Mistake: Focusing only on overall hit rate without considering other metrics like eviction rate or p99 latency.

#### Follow-Up
Discuss the importance of balancing these metrics. For example, a high hit ratio might be misleading if the cache is frequently evicting important items.

Mistake: Ignoring memory usage and assuming that the cache will scale indefinitely without constraints.

#### Follow-Up
Emphasize the need for capacity planning and understanding the trade-offs between cache size and performance.

Mistake: Not distinguishing between local and distributed caching systems when defining metrics.

#### Follow-Up
Highlight that different types of caches have unique characteristics, and some metrics might be more relevant in one scenario than another (e.g., network latency impacts for distributed caches).

**Comparison:**

| Metric         | Description                                           | Why Important                 |
|----------------|-------------------------------------------------------|-------------------------------|
| Hit Ratio      | Proportion of cache hits to total requests            | Ensures efficient caching     |
| Eviction Rate  | Number of items evicted per request                   | Prevents overpopulation       |
| p99 Latency    | 99th percentile response time                        | Identifies peak performance   |
| Memory Usage   | Current memory usage                                  | Avoids resource bottlenecks   |

**Code:**

```python
# Example code to calculate hit ratio from cache logs
def calculate_hit_ratio(cache_logs):
    hits = sum(1 for log in cache_logs if log['hit'])
    total_requests = len(cache_logs)
    return (hits / total_requests) * 100

cache_logs = [
    {'timestamp': '2023-09-01T10:00:00', 'hit': True},
    {'timestamp': '2023-09-01T10:05:00', 'hit': False},
    {'timestamp': '2023-09-01T10:10:00', 'hit': True}
]
print(calculate_hit_ratio(cache_logs))
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Cache Logs | Hit Ratio (94.3%), Eviction Rate (60/100), p99 Latency (15 ms) |

**Data:**

```json
[
  {
    "cache_type": "LRU",
    "memory_usage_mb": 512,
    "hit_rate_percent": 94.3,
    "eviction_count": 60,
    "p99_latency_ms": 15,
    "timestamp": "2023-09-01T23:59:59"
  }
]
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Yes
8. No
9. Yes
10. No
11. Start
12. Are the metrics up to SLA?
13. Monitor and Adjust
14. Check Hit Ratio
15. Is Hit Ratio > 90%?
16. No Action Needed
17. Optimize Cache Size orEviction Strategy
18. Check Eviction Rate
19. Is Eviction Rate < 5%?
20. Monitor p99 Latency
21. Reduce cache entries/evict lessfrequently
22. Check p99 Latency
23. Is p99 Latency < 10ms?
24. Monitor Memory Usage
25. Optimize cache hit rate orreduce requests at peak times
26. Check Memory Usage
27. Is Memory Usage within limits?
28. Reduce memoryusage/upgrade hardware

