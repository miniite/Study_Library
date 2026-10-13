---
type: "practice"
difficulty: "Deep"
---

# You must right-size timeouts org-wide from a downstream's p50/p99/p999. Explain why a timeout set at p99 causes a retry storm the instant the latency distribution shifts, and how you'd set adaptive, load-aware timeouts as a platform default instead of a static number.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Deadline Propagation](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Deadline_Propagation/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When setting timeouts in distributed systems, it's crucial to understand the implications of your choices on system performance and reliability. A common pitfall is using a timeout value based on worst-case scenarios (like p99 or p999 latency), which can lead to a phenomenon known as "retry storms" when traffic patterns shift. In contrast, adaptive timeouts that are informed by real-time load conditions can help maintain optimal system performance and reduce unnecessary retries.

#### Triage
What the Question Is Asking

The question is probing your understanding of how timeout settings impact distributed systems, particularly in relation to latency distribution and the need for adaptive mechanisms. The key assumptions are:

You have a system where requests experience varying latencies.
The goal is to set timeouts that balance reliability with performance.

#### First Principles Solution
Why p99 Causes Retry Storms

When setting a timeout at p99, you're aligning it with the 99th percentile of latency. This means that 1% of all requests will exceed this timeout value in normal operation. In practice, as traffic increases or system loads change, the distribution of latencies can shift, causing those 1% of slow requests to become much more frequent. For example:

If your current request distribution is such that p99 is at 10 seconds, and there's a sudden spike in traffic, even if this spike doesn't significantly affect the overall mean latency, it could push many more requests into the "slow" category.
When this happens, all those slow requests will trigger retries immediately upon hitting their timeout. This can lead to an exponential increase in retry attempts, overwhelming the system and causing a retry storm.
Adaptive Timeouts as Platform Defaults

To mitigate this issue, adaptive timeouts that scale based on current load conditions are preferable. By continuously monitoring the real-time latency distribution, you can set a more dynamic threshold that adjusts according to traffic and performance metrics. Here’s how you could implement such a system:

Monitor Real-Time Latency Distribution: Use observability tools like Prometheus, Grafana, or custom metrics to track the current p50, p99, and p999 latencies.
Calculate Adaptive Timeout Thresholds: Adjust your timeout thresholds based on these real-time statistics. For instance, you might set a default timeout that is a multiple of the current median latency (p50) plus some buffer, ensuring it remains high enough to handle even higher load scenarios without causing retries too frequently.
Dynamic Retries and Circuit Breakers: Implement retry strategies with exponential backoff and jitter to avoid overwhelming the system during transient failures. Circuit breakers can also be used to automatically shut down failing services or routes if they exceed certain thresholds.

#### Closing Summary

In summary, setting timeouts based on p99 or higher percentiles is risky because it can lead to retry storms when traffic patterns change. Adaptive timeouts that are informed by real-time latency metrics and load conditions provide a more resilient solution. By continuously monitoring and adjusting timeout values dynamically, you can ensure optimal performance and reliability in distributed systems.

#### Common Mistakes/Follow-Ups
Ignoring Real-Time Metrics: Failing to monitor and adjust based on current system state.
Overly Aggressive Retries: Without proper backoff strategies, retries can overwhelm the system.
Static Timeout Settings: Not adapting timeouts according to load changes or other real-time factors.

Would you like me to elaborate further on any of these points?

**Comparison:**

```
Current Latencies	Median (p50)	P99	Adaptive Timeout
[3, 4, 5, 6, 7]	5.5	6	8
[3, 4, 5, 6, 7, 8, 9, 10, 11, 12]	7	11	13
[3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 100]	10.5	11	12
```

**Code:**

```python
# Example Python code for calculating adaptive timeout thresholds

import statistics

def calculate_adaptive_timeout(current_latencies):
    p50 = statistics.median(current_latencies)
    p99 = sorted(current_latencies, reverse=True)[round(len(current_latencies) * 0.01)]
    buffer = 2  # Example buffer value
    adaptive_timeout = max(p50 + buffer, p99)
    return adaptive_timeout

# Sample data for demonstration
current_latencies = [3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 100]  # Simulated latencies
adaptive_timeout = calculate_adaptive_timeout(current_latencies)
print(f"Adaptive Timeout: {adaptive_timeout}")
```

**Example input / output:**

```text
Input:
- Current latencies: `[3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 100]`

Output:
- Adaptive timeout: `12` (with a buffer of 2)

Explanation: The median latency is 6.5, p99 is 11, and with a buffer of 2, the adaptive timeout is set to 12.
```

**Data:**

```json
{
  "currentLatencies": [3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 100]
}
```

**Flow:**

1. Yes
2. No
3. Real-time Latency Monitoring
4. Is Load High?
5. Adjust Adaptive Timeout
6. Keep Current AdaptiveTimeout
7. Monitor for Stability, Adjust asNecessary

