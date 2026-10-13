---
type: "practice"
difficulty: "Deep"
---

# Your cache hit ratio suddenly dropped from 95% to 60% at 2am. Walk me through your investigation.

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Cache Hit Ratio](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Cache_Hit_Ratio.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When we see a sudden drop in our cache hit ratio, it suggests that something significant has changed in our system behavior or infrastructure. At 2 am, when traffic is typically low, this could indicate either an issue with the data freshness or a problem with how requests are being routed to the cache. The first step is to understand why and where the discrepancy might be occurring.

To investigate, we need to consider several key aspects:

Cache Configuration Changes: Check if any recent changes were made to the cache configuration.
Data Freshness: Verify that the data in the cache is still valid.
Request Routing: Ensure that requests are being directed to the cache correctly.

Here’s a detailed step-by-step approach:

1. Review Recent Changes
Cache Configuration: Look at recent code deployments, configuration changes, or any new features that might have impacted the cache behavior.
Environment Logs: Check logs for any system-wide events that occurred around the time of the drop.
2. Analyze Cache Data Freshness
Expiry Times: Ensure that data in the cache is expiring correctly and that there are no issues with the expiration logic.
Data Source: Verify if the underlying database or data source has changed, leading to stale or incorrect data being cached.
3. Check Request Routing
Cache Headers: Inspect HTTP headers like ETag, Last-Modified, If-None-Match, and If-Modified-Since to ensure they are correctly configured.
Load Balancer/Router Configuration: Review the configuration of any load balancers or routers that might be routing requests differently at night.
4. Performance Metrics
Cache Miss Rate: Track the rate of cache misses to understand if it has increased significantly.
Hit Time and Miss Time: Analyze the time taken for hits vs. misses to identify potential bottlenecks.
5. Debugging Tools and Logs
Monitoring Tools: Use tools like Prometheus, Grafana, or New Relic to monitor cache metrics in real-time.
Custom Logging: Implement custom logging around cache operations to capture detailed information about each request and its outcome.
Detailed Steps
Check Recent Changes:
Review code repositories for recent commits or pull requests related to caching.
Look at configuration files like redis.conf, nginx.conf, etc., for any changes.
Examine Data Freshness:
Run a script to check the expiry times of cached data.
Validate that the cache is being updated with fresh data from the database.
Inspect Request Routing:
Use network tools like Wireshark or Charles Proxy to inspect HTTP headers and request patterns.
Check load balancer logs for any changes in routing rules.

#### Closing Summary

By systematically reviewing recent changes, verifying data freshness, and ensuring proper request routing, we can identify the root cause of the cache hit ratio drop. Common pitfalls include overlooking recent configuration changes or failing to properly validate that cached data is up-to-date.

**Comparison:**

```
Time	Cache Miss Rate
2am	34.5%
```

**Code:**

```python
# Example script to check cache expiry times
import redis

cache = redis.Redis(host='localhost', port=6379, db=0)
keys = cache.keys()

for key in keys:
    print(f"Key: {key.decode('utf-8')}, Expiry Time: {cache.ttl(key)}")
```

**Example input / output:**

```text
Input:
- Recent code deployments and config changes.
- Logs from recent system events.

Output:
- Identified that a recent database schema change was causing stale data to be cached.
- Configured the cache expiration logic to ensure proper data freshness.
- Updated load balancing rules to route requests more effectively during low traffic periods.
```

**Data:**

```json
{
  "cacheMissRate": {
    "2am": 34.5,
    "previousHour": 6.7
  },
  "hitTimeVsMissTime": [
    {"time": "00:00", "hitTime": 10, "missTime": 2},
    {"time": "03:00", "hitTime": 5, "missTime": 4}
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
7. Start Investigation
8. Recent Changes?
9. Review Code/Config
10. Data Freshness?
11. Check Expiry Logic
12. Request Routing?
13. Inspect Load Balancer Rules
14. Complete Investigation
15. Implement Fixes
16. Update Expiry Logic
17. Route Requests Correctly

