---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd downsample and roll up metrics inside the lake so that long-horizon trend anomaly detection stays fast without losing signal.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Hot vs Cold Telemetry](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Hot_vs_Cold_Telemetry.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure long-horizon trend anomaly detection remains fast while retaining important signal data, we need to carefully downsample and roll up metrics stored in our Observability Data Lake. This process involves several key steps that balance computational efficiency with the ability to detect meaningful anomalies over extended time periods.

Firstly, let's define what downsampling and rolling up mean in this context: downsampling reduces the data volume by decreasing the temporal resolution (e.g., from minutes to hours), while rolling up aggregates data points into higher-level summaries. The goal is to maintain the integrity of trends without overwhelming the system with too much raw data.

#### Triaging the Question

The question aims to explore how we can efficiently manage large volumes of time-series data for long-term trend analysis, ensuring both performance and accuracy. We need to make assumptions about the nature of the metrics (e.g., granularity, types) and the specific anomaly detection requirements.

#### Assumptions
Metrics are collected at high frequency (e.g., every minute).
We want to analyze trends over long horizons (months or years).
Anomaly detection is critical but must be computationally efficient.
The system should handle large-scale data effectively with minimal latency.

#### First-Principles Solution

High-Frequency Data Collection: Collect metrics every minute, ensuring comprehensive coverage.

Downsampling Strategy:

Aggregation Functions: Use appropriate aggregation functions (e.g., mean, median, max) based on the nature of the metric.
Temporal Resolution: Downsample to hourly or daily data points for storage in the lake.

Rolling Up Metrics:

Daily Aggregates: Create rolling aggregates per day (e.g., aggregate hourly metrics into daily summaries).
Weekly/Monthly Aggregates: Further roll up data into weekly and monthly aggregations to reduce the dataset size.

Data Storage Strategy:

Time-Indexed Partitions: Organize data by time, partitioning based on year, month, or week for efficient querying.
Parquet/Arrow Formats: Use columnar storage formats like Parquet or Arrow for fast data access and compression.

Query Optimization:

Indexing: Implement indexes on timestamp fields to speed up query performance.
Caching: Cache frequently accessed summaries to reduce computation load.

Alerting and Reporting:

Real-Time Alerts: Set up alerts based on high-resolution data for immediate issues.
Periodic Reports: Generate periodic reports using lower-resolution aggregated data.

#### Closing Summary

By carefully downsampling and rolling up metrics, we can significantly reduce the volume of data stored in our Observability Data Lake while retaining essential information for long-term trend analysis. This approach ensures that anomaly detection remains fast and effective without overwhelming the system with raw high-frequency data.

#### Common Mistakes/Follow-Ups
Overdownsampling: Reducing temporal resolution too much can lead to loss of critical signals.
Improper Aggregation Functions: Using incorrect aggregation functions might distort trends or fail to capture anomalies accurately.
Insufficient Caching: Not caching frequently accessed summaries can degrade performance.
Lack of Indexing: Poor indexing can severely impact query times.
Triaging Further
Metrics Type: Different types of metrics (e.g., count, rate) may require different aggregation strategies.
Storage Costs: Higher resolution data and frequent roll-ups can increase storage costs; balance is key.
Performance vs. Accuracy Trade-offs: Understand the specific requirements for performance versus accuracy in real-world scenarios.

**Comparison:**

```
Time	Metric Value	Aggregation Type	Description
Hourly	12	Mean	Downsampled value for each hour
Daily	120	Mean, Standard Deviation	Aggregated values over a day
```

**Code:**

```python
# Example code for downsampling and rolling up metrics

def downsample_and_roll_up(metric_data, resolution='1H'):
    # Downsample data to desired resolution
    downsampled = metric_data.resample(rule=resolution).mean()

    # Roll up daily aggregates
    daily_aggregates = downsampled.groupby(pd.Grouper(freq='D')).agg(['mean', 'std'])

    return downsampled, daily_aggregates

# Example usage
import pandas as pd

metric_data = pd.DataFrame({
    'timestamp': pd.date_range(start='2023-01-01', end='2023-01-31 23:59', freq='T'),
    'value': [i for i in range(24*31)]
})

downsampled, daily_aggregates = downsample_and_roll_up(metric_data)
```

**Example input / output:**

| Timestamp | Downsampled Value (1H) | Daily Mean Aggregate |
|-----------|-----------------------|---------------------|
| 2023-01-01 | 12                     | 12                  |
| ...       | ...                   | ...                 |
| 2023-01-31 | 718                    | 718                 |

**Data:**

```json
{
    "metric_data": {
        "timestamp": ["2023-01-01 00:00", "2023-01-01 00:01", ..., "2023-01-31 23:59"],
        "value": [0, 1, 2, ..., 720]
    }
}
```

**Flow:**

1. Yes
2. No
3. Collect Metrics
4. High-Resolution Data?
5. Downsample to Hourly
6. D
7. Roll Up Daily Aggregates
8. Store in Data Lake (Partitioned)
9. Roll Up Weekly/MonthlyAggregates
10. Query Optimized Storage

