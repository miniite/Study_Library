---
type: "practice"
difficulty: "Deep"
---

# What is the difference between average latency and percentile latency and why does it matter for alerting?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Composite Monitors](<../../../../TOOLS/Datadog/Datadog_Platform/Composite_Monitors.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the difference between average latency and percentile latency, and why they matter for alerting in AIOps, let's start by breaking down these concepts from first principles.

TL;DR

Average latency gives you an overall idea of performance, while percentile latency helps identify outliers that could be impacting user experience. Understanding both is crucial for effective monitoring and alerting, ensuring robust service health.

#### Triaging the Question

This question probes your understanding of fundamental metrics used in AIOps, specifically in the context of Datadog. Average latency measures the central tendency of response times, whereas percentile latency (e.g., 95th or 99th percentile) highlights the tail behavior—important for identifying and addressing slow performance issues.

#### First-Principles Solution
Average Latency

Average latency is calculated as: [ \text{Average Latency} = \frac{\sum_{i=1}^{n} t_i}{n} ] where ( t_i ) represents the response time for each request, and ( n ) is the total number of requests. This metric provides a straightforward average value that can give you an overall sense of how fast your system performs most of the time.

Percentile Latency

Percentile latency measures the performance at different points in the distribution. For example, 95th percentile means that 95% of all requests have response times below this value. This is calculated as: [ \text{95th Percentile} = Q_{0.95} ] where ( Q_x ) is the quantile function at the ( x )-th percentile.

Why It Matters for Alerting
Identifying Outliers: Average latency can hide slow-performing requests that are significantly longer than average. For instance, if 95% of your requests take 10ms but 2% take 3 seconds each, the average might be around 11ms, making it seem like everything is fine.
Service Quality: Users often experience the worst-case scenarios (tail behavior), not the average. Alerting on percentile latency ensures you can address performance issues that affect the majority of users.

#### Closing Summary

In summary, while average latency provides a general idea of system performance, percentile latency is essential for identifying slow response times that could degrade user experience and service quality. Effective alerting in AIOps requires monitoring both to ensure comprehensive health checks.

#### Common Mistakes/Follow-Ups
Overreliance on Average Latency: Many teams only monitor average latency, which can lead to missing critical performance issues.
Ignoring Outliers with Percentile Metrics: Failing to set alerts for high percentiles (e.g., 95th or 99th) means you might not catch slow responses that impact a significant number of users.

For further discussion, consider how these metrics can be integrated into Datadog's monitoring stack and the trade-offs involved in choosing which percentiles are most relevant.

**Comparison:**

| Response Times | 10ms | 12ms | 15ms | 3s (3000ms) | 20ms | 25ms |
|----------------|------|------|------|------------|------|------|
| Average Latency| 18.00|      |      |            |      |      |
| 95th Percentile| 3000 |      |      |            |      |      |

**Code:**

```python
# Example code to calculate average and percentile latency using Python
import numpy as np

def calculate_latencies(response_times):
    # Calculate average latency
    average_latency = np.mean(response_times)

    # Calculate 95th percentile latency
    percentile_95 = np.percentile(response_times, 95)

    return average_latency, percentile_95
```

**Example input / output:**

```text
# Sample input and output
response_times = [10, 12, 15, 3000, 20, 25]

average_latency, percentile_95 = calculate_latencies(response_times)

print(f"Average Latency: {average_latency:.2f}")
print(f"95th Percentile: {percentile_95:.2f}")

# Output:
# Average Latency: 18.00
# 95th Percentile: 3000.00
```

**Data:**

```json
{
  "response_times": [10, 12, 15, 3000, 20, 25]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Calculate Average Latency
6. Response Times
7. Compute Mean
8. Average Latency = 18.00ms
9. Calculate 95th Percentile
10. Response Times
11. 95th Percentile = 3000ms
12. No Data or Invalid Input

