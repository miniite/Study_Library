---
type: "practice"
difficulty: "Deep"
---

# How would you decide whether to invest platform effort in cutting p50 versus p99 pipeline time, given developers viscerally feel the tail but the median drives throughput math; what data would settle it, and what's the second-order behavioral effect (batching, context-switching) of each choice on how engineers work?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ CI Performance Investment](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/CI_Cost_Optimization/CI_Performance_Investment.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Deciding whether to invest in cutting p50 or p99 pipeline time involves understanding the trade-offs between developer productivity and system performance, especially given that developers focus more on tail latencies but median times drive overall throughput. Data such as historical latency distributions, job batch sizes, and context-switching metrics are crucial. The second-order effects include changes in how engineers work, potentially leading to different coding practices or deployment strategies.

#### Triaging the Question

The question is about optimizing a CI/CD pipeline by deciding where to invest platform effort—whether it's reducing p50 (median) pipeline time or p99 tail latency. Developers often feel pain more acutely at the tails because these latencies directly impact user experience and developer morale, but median times are critical for overall throughput.

#### Assumptions
Median times significantly affect system throughput.
Tail latencies have a high impact on perceived performance and user satisfaction.
Both p50 and p99 metrics are relevant to the CI/CD pipeline's efficiency.

#### First-Principles Solution
Step 1: Data Collection and Analysis

Collect historical data on pipeline latencies, job batch sizes, and context-switching rates. Use tools like Prometheus or Grafana for monitoring system performance metrics over time. Analyze this data to understand the frequency and magnitude of p50 vs. p99 latencies.

Step 2: Quantify the Impact

Calculate the impact on throughput by reducing p50 latency versus p99 latency using statistical methods. For example, if median times are significantly longer than tail latencies, investing in reducing medians might yield higher overall performance improvements due to better resource utilization and faster average processing.

Step 3: Evaluate Second-Order Effects

#### Cons
ider the second-order effects of each decision on how engineers work:

Reducing p50 Pipeline Time: This can lead to more frequent deployments, which may reduce context-switching but could also introduce more frequent manual intervention or automated retries.
Reducing p99 Tail Latency: This can improve user satisfaction and developer morale, potentially leading to less urgency for developers during longer pipeline runs.
Step 4: Cost-Benefit Analysis

Perform a cost-benefit analysis by estimating the resources required to reduce both latencies versus their respective impact on throughput. Use metrics like Mean Time Between Failures (MTBF) and Mean Time To Repair (MTTR) to gauge the long-term benefits of each investment.

#### Closing Summary
and Common Mistakes/Follow-Ups

Investing in reducing p99 latency can have a significant positive impact on user experience and developer satisfaction, but it may require more complex pipeline configurations. Conversely, reducing p50 latency could enhance overall throughput but might not significantly affect tail latencies. The decision should be based on a combination of data-driven analysis and understanding the second-order effects.

#### Common mistakes
include:

Ignoring median performance metrics.
Failing to consider the impact on developer workflows and productivity.
Overlooking potential context-switching issues that arise from frequent short pipeline runs.
Second-Order Behavioral Effects
Batching: Reducing p50 might encourage larger batch sizes, reducing context-switching but potentially increasing the load on resources during long runs.
Context-Switching: Focusing on tail latencies can reduce developer frustration and improve user experience, leading to more stable and reliable deployments.

#### Conclusion

To make an informed decision, collect detailed performance data, analyze its impact on throughput, and consider the broader implications for developers' workflows. The choice between p50 and p99 optimization should balance immediate pain points with long-term system efficiency.

**Comparison:**

```
70%
30%
Pipeline Latency Reduction Impact
P50 Reduction
P99 Reduction
```

**Code:**

```python
# Example code to collect pipeline latency metrics using Prometheus API
import requests

def fetch_latency_metrics(url):
    response = requests.get(f"{url}/api/v1/query_range", params={
        'query': 'histogram_quantile(0.5, sum(rate(pipeline_latency[1m])) by (le))',
        'start': 'now()-30s'
    })
    p50_data = response.json()['data']['result'][0]['values']

    response = requests.get(f"{url}/api/v1/query_range", params={
        'query': 'histogram_quantile(0.99, sum(rate(pipeline_latency[1m])) by (le))',
        'start': 'now()-30s'
    })
    p99_data = response.json()['data']['result'][0]['values']

    return {
        'p50': p50_data,
        'p99': p99_data
    }
```

**Example input / output:**

| Timestamp | P50 Latency (ms) | P99 Latency (ms) |
|-----------|-----------------|-----------------|
| 1632475200 | 50              | 150             |
| 1632482400 | 55              | 145             |
| 1632489600 | 60              | 140             |

**Data:**

```yaml
# Sample data schema for pipeline latency metrics
pipeline-latency-metrics:
  - metric: p50
    values:
      - [timestamp, value]
  - metric: p99
    values:
      - [timestamp, value]
```

**Flow:**

1. Yes
2. No
3. Collect Metrics
4. Is Median Crucial?
5. Optimize P50
6. Optimize P99
7. Frequent Deployments,Reduced Context Switching
8. Improved User Experience,Developer Satisfaction

