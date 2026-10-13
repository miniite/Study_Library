---
type: "practice"
difficulty: "Intermediate"
---

# What's the difference between rate() and irate() in PromQL and when would picking the wrong one distort an anomaly detector's input signal?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Prometheus TSDB and Scrape Health](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases/Prometheus_TSDB_and_Scrape_Health.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Rate and Instantaneous Rate (irate) are both functions in Prometheus Query Language (PromQL) used to calculate derivative values from time series data. However, they differ significantly in their approach to handling rate calculation over different time periods, which can lead to subtle but important differences when feeding signals into anomaly detectors.

TL;DR: Use rate() for consistent relative rates over a fixed window, and irate() for instantaneous rates at the last sample point, ensuring your anomaly detector receives appropriate and accurate data.

#### Triaging the Question

The question is essentially asking about the distinction between two PromQL functions used to calculate derivatives of time series data. The goal here is to understand how these different approaches can affect the input signal of an anomaly detector, potentially leading to misleading or incorrect alerts.

#### Assumptions

Anomaly detection relies on accurate derivative calculations.
We are dealing with a time series database (Prometheus) where rate() and irate() functions are used for rate calculation.
The anomaly detector is sensitive to the specific values of these rates.

#### First Principles Solution
1. Understanding Rate (rate())

rate() computes the derivative value by dividing the change in the metric’s value over a specified interval (window) by the duration of that window. It provides a relative rate of change and ensures consistency across different time periods by averaging the changes within the window.

#### Example

PROMQL
rate(counter[5m])

This calculates the average growth rate of counter over the last 5 minutes, which is useful for metrics like counts or sums that increase monotonically.

2. Understanding Instantaneous Rate (irate())

irate() computes the derivative value at the last sample point within a specified interval. It provides an instantaneous rate rather than an average change over the window. This can be more suitable when you want to capture sudden changes in the metric values, but it is less reliable for metrics that do not follow a consistent trend.

#### Example

PROMQL
irate(counter[5m])

This calculates the derivative at the last sample point within the 5-minute interval, which might be useful for detecting spikes or drops in real-time data.

3. Implications on Anomaly Detection

#### Cons
istency: rate() provides a more consistent view of the metric's growth rate over time, making it easier to detect anomalies that follow a predictable pattern.
Spikes and Drops: irate() can better capture sudden changes in the metric values, which are crucial for detecting unexpected spikes or drops. However, these might not always be meaningful if they occur at random intervals.
4. Case Study: Detecting Disk Usage Trends

#### Cons
ider a scenario where you are monitoring disk usage across multiple servers. Using rate() would help identify trends and anomalies that follow a consistent pattern (e.g., gradual increase or decrease in disk usage). On the other hand, using irate() might pick up on sudden spikes due to temporary high load conditions.

#### Closing Summary

Choosing between rate() and irate() depends on your specific use case. Use rate() when you need relative rates that provide a consistent view of metric changes over time. Use irate() when you want to capture the most recent instantaneous rate, which can be useful for detecting spikes or drops in real-time data.

#### Common Mistakes/Follow-Ups
Using irate() for long-term trends: This can lead to misleading results because it only considers the last sample point.
Over-relying on rate() without considering real-time changes: For metrics that experience sudden changes, rate() might not capture these events accurately.

**Comparison:**

| Time        | Value     | Counter   |
|-------------|-----------|-----------|
| 2023-10-01T00:00Z | 100      | 500       |
| 2023-10-01T00:05Z | 110      | 550       |

**Code:**

```
PROMQL
Copy
# Example of rate()
rate(counter[5m])

# Example of irate()
irate(counter[5m])
```

**Example input / output:**

```text
# Using rate() on the provided data
rate([
  { timestamp: "2023-10-01T00:00Z", value: 100, counter: 500 },
  { timestamp: "2023-10-01T00:05Z", value: 110, counter: 550 }
]) = (110 - 100) / 5m = 2.0

# Using irate() on the same data
irate([
  { timestamp: "2023-10-01T00:00Z", value: 100, counter: 500 },
  { timestamp: "2023-10-01T00:05Z", value: 110, counter: 550 }
]) = (110 - 100) / 0.0833h ≈ 6
```

**Data:**

```json
{
  "data": [
    {
      "timestamp": "2023-10-01T00:00Z",
      "value": 100,
      "counter": 500
    },
    {
      "timestamp": "2023-10-01T00:05Z",
      "value": 110,
      "counter": 550
    }
  ]
}
```

**Flow:**

1. Rate Calculation (rate())
2. Consistency over Time
3. Use for long-term trends
4. Irate Calculation (irate())
5. Capturing InstantaneousChanges
6. Use for real-time spikes ordrops

