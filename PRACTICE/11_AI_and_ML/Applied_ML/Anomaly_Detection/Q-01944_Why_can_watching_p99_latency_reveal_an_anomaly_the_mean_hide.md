---
type: "practice"
difficulty: "Intermediate"
---

# Why can watching p99 latency reveal an anomaly the mean hides, and how do you detect on percentiles at scale?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Smoothing and Sensitivity Trade-offs](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Smoothing_and_Sensitivity_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

P99 latency is crucial because it helps identify rare but severe outliers that can significantly impact service performance. By detecting anomalies in p99, we ensure that critical issues are not overlooked by focusing only on the mean latency. This solution involves using statistical methods and scalable infrastructure to monitor percentiles effectively.

#### Triaging the Question

The question asks why monitoring P99 (99th percentile) latency is important for anomaly detection and how to detect anomalies in percentiles at scale. The core of the question lies in understanding the limitations of mean latency and the benefits of using percentiles like P99, especially when dealing with time series data.

#### Assumptions
Data Collection: We are working with a large dataset of latency measurements.
Time Series Data: The latency data is collected over time, forming a time series.
Scalability Requirements: The solution needs to handle the scale of the data efficiently and effectively.
Step-by-Step Solution:
1. Understanding Mean vs. Percentiles

The mean latency provides an overall average, which can be misleading in cases where there are significant outliers. For instance, if most requests have a low latency but a few requests experience extremely high latencies (due to network issues or server overload), the mean might not accurately represent the typical user experience.

2. Importance of P99
Identifying Outliers: P99 captures the extreme tail end of the distribution, where most of the data lies within the first 99%. This means that any outliers (high latency) will be highlighted.
Real User Experience: For many services, especially those with high availability and performance requirements, user experience is critical. P99 helps ensure that even a small percentage of users do not experience unacceptable delays.
3. Detecting Anomalies

To detect anomalies in percentiles at scale:

Time Series Decomposition:
Separate the data into trend, seasonal, and residual components.
Statistical Methods:
Use statistical thresholds to define what constitutes an anomaly (e.g., deviations from expected values).
Machine Learning Approaches:
Train models on historical data to learn normal patterns.
Monitoring Infrastructure:
Implement robust monitoring systems that can handle large volumes of time series data.
4. Implementation Steps

Data Collection and Storage:

Collect latency data at regular intervals (e.g., every second or minute).
Store the data in a scalable database like Time Series databases (Prometheus, InfluxDB).

Anomaly Detection Algorithms:

Implement algorithms such as Z-score, Moving Average, or more advanced models like LSTM.

Real-time Monitoring:

Set up real-time alerts based on detected anomalies.
5. Scalability Considerations
Parallel Processing: Use distributed systems to process data in parallel.
Streaming Data Handling: Implement stream processing frameworks (Apache Kafka, Apache Flink) for real-time analysis.
Efficient Storage and Querying: Utilize database optimizations like indexes and query caching.

#### Closing Summary

Monitoring P99 latency is essential because it helps identify critical performance issues that the mean might miss. By combining statistical methods with scalable infrastructure, we can effectively detect anomalies in percentiles at scale. This approach ensures that user experience is prioritized and services remain highly available.

#### Common Mistakes/Follow-Ups
Over-reliance on Mean: Failing to consider outliers by only monitoring the mean.
Ignoring Scalability: Not designing a solution that can handle large volumes of data efficiently.
Inadequate Alerting: Setting up ineffective alert systems that fail to provide timely notifications.
Additional Considerations:
Impact Analysis: Understanding how anomalies affect different service components and users.
Cost vs. Benefit: Balancing the cost of advanced monitoring tools against the benefits they provide.

**Comparison:**

```
Component	Description
Trend	Long-term trend of the series
Seasonal	Periodic variations
Residual	Random noise
```

**Code:**

```python
from statsmodels.tsa.seasonal import seasonal_decompose

# Example code to decompose time series data
def decompose_time_series(data):
    result = seasonal_decompose(data)
    return result.trend, result.seasonal, result.resid

trend, seasonal, residual = decompose_time_series(latency_data)
```

**Data:**

```json
{
  "latency_data": [
    {"timestamp": "2023-10-01T12:00:00Z", "value": 50},
    {"timestamp": "2023-10-01T12:01:00Z", "value": 60},
    ...
    {"timestamp": "2023-10-01T14:00:00Z", "value": 1500}
  ]
}
```

**Flow:**

1. Data Collection
2. Time Series Decomposition
3. Trend Analysis
4. Seasonal Patterns
5. Residuals

