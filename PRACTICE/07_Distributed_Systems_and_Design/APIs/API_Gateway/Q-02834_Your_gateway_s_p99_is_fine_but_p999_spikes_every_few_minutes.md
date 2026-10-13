---
type: "practice"
difficulty: "Deep"
---

# Your gateway's p99 is fine but p999 spikes every few minutes correlated with GC or config reload; how do you separate proxy-internal jitter from upstream jitter, and what do you tune first?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Gateway Scaling for Traffic Spikes](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Connection_Reuse_and_Client_Errors/Gateway_Scaling_for_Traffic_Spikes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue where your API Gateway's P99 is stable but P999 spikes every few minutes correlated with garbage collection (GC) or config reloads, we need to distinguish between proxy-internal jitter and upstream jitter. The key here is to isolate these two sources of variability and understand their respective impacts on performance. By doing so, we can make informed decisions about where to focus our optimization efforts.

First, let's triage the problem:

Understanding the Question: The interviewer is probing your ability to diagnose complex issues in an API Gateway by distinguishing between internal and external factors that affect latency.

#### Assumptions
The gateway and upstream services are monitored for response times.
There is a clear correlation between GC or config reloads and P999 spikes.
Step-by-Step Solution

Isolate Internal Jitter:

Analyze Gateway Metrics: Monitor the internal operations of the API Gateway, such as request handling time, processing latency, and any bottlenecks introduced by the gateway itself.
Disable Config Reloads Temporarily: To isolate internal jitter from external factors, temporarily disable config reloads to see if P999 spikes persist. If they do not spike, it indicates that proxy-internal operations are likely contributing to the issue.

Isolate External Jitter:

Analyze Upstream Service Metrics: Check the response times of upstream services to understand their behavior during GC or config reloads.
Simulate Config Reloads: Manually trigger config reloads and monitor how they impact upstream service latency. This will help in understanding if spikes are caused by external factors.

Tuning Strategy:

Optimize Gateway Configuration: If internal jitter is the culprit, tune gateway-related parameters such as buffer sizes, connection timeouts, and caching policies.
Adjust Upstream Services: If upstream services are to blame, work with the service owners to optimize their configurations or reduce GC pause times.

#### Closing Summary

To summarize, we need a systematic approach to distinguish between internal and external factors affecting the API Gateway’s latency. By disabling config reloads and analyzing gateway metrics separately from upstream metrics, we can focus on tuning the appropriate components. Typically, focusing first on reducing internal jitter (via gateway optimization) is more straightforward due to better control over these factors.

#### Common Mistakes
Failing to properly isolate internal and external jitter.
Ignoring the impact of GC pauses directly on upstream services without considering how they affect overall latency.
Overlooking the importance of tuning both gateway and upstream components based on observed metrics.

**Comparison:**

```
Metric	Before Disable Config Reload	After Disable Config Reload
P99	120 ms	85 ms
P999	300 ms	70 ms
```

**Code:**

```python
# Example code for monitoring P99/P999 metrics
import time

def monitor_gateway_metrics(gateway):
    # Simulate periodic metrics collection
    while True:
        start = time.time()
        response_time = gateway.process_request()
        end = time.time()
        total_time = (end - start) * 1000  # in milliseconds

        if total_time > 99.0:  # P99 threshold
            print(f"P99: {total_time} ms")

        if total_time > 999.0:  # P999 threshold
            print(f"P999: {total_time} ms")

# Example API Gateway class
class SimpleGateway:
    def process_request(self):
        time.sleep(0.1)  # Simulate processing delay
        return 50  # Simulated response time

monitor_gateway_metrics(SimpleGateway())
```

**Example input / output:**

```text
Input:
Monitor the gateway metrics during a config reload.

Output:
P99: 120 ms (before reload)
P999: 300 ms (before reload)

After disabling config reloads:
P99: 85 ms (after disable)
P999: 70 ms (after disable)
```

**Data:**

```json
{
  "metrics": [
    {"timestamp": "2023-10-01T14:30:00Z", "p99": 85, "p999": 95},
    {"timestamp": "2023-10-01T14:31:00Z", "p99": 78, "p999": 65}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Monitor Gateway Metrics
4. Is Config Reload Enabled?
5. Disable Config ReloadsTemporarily
6. Analyze Internal Jitter
7. Analyze Upstream ServiceMetrics
8. Simulate Config Reloads
9. Tune Upstream Services
10. Tune Gateway Configuration

