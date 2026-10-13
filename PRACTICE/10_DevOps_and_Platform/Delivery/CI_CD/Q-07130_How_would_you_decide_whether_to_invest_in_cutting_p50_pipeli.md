---
type: "practice"
difficulty: "Deep"
---

# How would you decide whether to invest in cutting p50 pipeline time or p99 pipeline time, given developers feel the tail but the median drives throughput math; what data would settle it and what's the second-order effect of each choice on developer behavior?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ CI Performance Investment](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/CI_Cost_Optimization/CI_Performance_Investment.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide whether to invest in reducing the p50 (median) or p99 (tail) pipeline time, we need to understand the trade-offs and potential impact on overall system performance. Given that developers feel more about the tail latency but throughput is primarily driven by median times, we must balance these concerns carefully.

#### Triage

The question is probing our understanding of latency distribution, throughput vs. tail latency trade-off, and how these decisions affect developer experience and system performance. We need to consider both first-order effects (immediate benefits) and second-order effects (long-term impacts on behavior).

#### Assumptions
Developers primarily care about the worst-case scenarios, i.e., p99.
Throughput is heavily influenced by the median latency.

#### First-Principles Solution

Collecting Data:

We need to gather real-time performance metrics for both p50 and p99 latencies over a representative time period.
Analyze historical data to understand the distribution of response times and identify any patterns or outliers.

Impact Analysis:

Evaluate how reducing median latency (p50) would impact overall throughput.
Use a queuing theory model or simulation tools like Golang's gopsutil for workload analysis.
Assess the effect on tail latency by implementing strategies such as optimization, caching, and parallelization.

Developer Behavior:

Understand how developers perceive different latencies. For instance, if the median time is within acceptable limits but p99 times are excessively high, this could lead to developer dissatisfaction.

#### Cons
ider using surveys or user feedback mechanisms to gauge developer satisfaction with current latency levels.

Second-Order Effects:

Investigate how optimizations for tail latency might inadvertently affect median latencies and vice versa.
Evaluate the impact on system stability, resource utilization, and overall maintainability of the codebase.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Investing in reducing p99 latency can significantly enhance developer experience by addressing critical performance issues. However, this should not come at the cost of throughput, which is driven by median latencies. A balanced approach that leverages both metrics will yield optimal results.

#### Common Mistakes
Ignoring the importance of tail latencies and focusing solely on average or median values.
Overestimating the impact of p99 improvements without validating through performance testing.
Failing to consider the broader implications on developer productivity and system stability.

#### Follow-Ups
What specific tools would you use for performance monitoring?
How do you ensure that the chosen optimizations do not introduce new bottlenecks or bugs?
Can you provide an example of a project where similar trade-offs were made, and what was the outcome?

**Comparison:**

| Time       | P50 Latency (ms) | P99 Latency (ms) | Throughput (req/s) |
|------------|------------------|------------------|--------------------|
| 2023-10-01T00:00Z | 20              | 100             | 500               |
| 2023-10-01T01:00Z | 25              | 150             | 450               |

**Code:**

```python
# Example code for gathering performance metrics using Golang's gopsutil
import psutil

def gather_performance_metrics():
    # Collect CPU usage
    cpu_percent = psutil.cpu_percent(interval=1)

    # Collect memory usage
    mem_info = psutil.virtual_memory()
    memory_usage = mem_info.percent

    # Collect network I/O metrics
    net_io = psutil.net_io_counters()
    bytes_sent, bytes_recv = net_io.bytes_sent, net_io.bytes_recv

    return {
        "cpu_percent": cpu_percent,
        "memory_usage": memory_usage,
        "bytes_sent": bytes_sent,
        "bytes_recv": bytes_recv
    }
```

**Example input / output:**

```text
Input:
- Gather performance metrics from Golang application using gopsutil library.

Output:
- CPU usage: 20%
- Memory usage: 75%
- Network sent bytes: 10,000 KB/s
- Network received bytes: 8,000 KB/s

Analysis:
- Median latency (p50): 20 ms
- Tail latency (p99): 100 ms
- Throughput: 500 requests/second
```

**Data:**

```json
{
  "p50_latency": [
    {"timestamp": "2023-10-01T00:00Z", "value": 20},
    {"timestamp": "2023-10-01T01:00Z", "value": 25}
  ],
  "p99_latency": [
    {"timestamp": "2023-10-01T00:00Z", "value": 100},
    {"timestamp": "2023-10-01T01:00Z", "value": 150}
  ],
  "throughput": [
    {"timestamp": "2023-10-01T00:00Z", "requests_per_second": 500},
    {"timestamp": "2023-10-01T01:00Z", "requests_per_second": 450}
  ]
}
```

**Flow:**

1. Rendering diagram…

