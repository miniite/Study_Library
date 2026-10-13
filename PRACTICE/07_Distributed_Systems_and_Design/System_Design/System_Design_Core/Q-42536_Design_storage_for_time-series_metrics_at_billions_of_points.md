---
type: "practice"
difficulty: "Deep"
---

# Design storage for time-series metrics at billions of points/day so recent data is fast and old data is cheap. Discuss downsampling, rollups, retention tiers, and why a columnar layout helps range queries.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Clickstream Counting Pipeline](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Clickstream_Counting_Pipeline/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design storage for time-series metrics that handle billions of points per day while ensuring recent data is fast and old data is cheap, we need to consider several key strategies: downsampling, rollups, retention tiers, and choosing an efficient data layout like columnar. Let's break down each component from first principles.

#### Triage

The core of the question revolves around optimizing storage for time-series data while managing costs efficiently over different retention periods. The interviewer is probing your understanding of trade-offs between performance, cost, and data accessibility. We will assume a typical scenario where real-time access to recent data is critical, but historical data can be periodically accessed at lower resolution.

#### First-Principles Solution
Downsampling: Reduce the granularity of older data points while maintaining enough detail for analysis.
Rollups: Aggregate time-series metrics into higher-level summaries over longer periods.
Retention Tiers: Store different levels of data with varying access frequencies and costs.
Columnar Layout: Optimize storage and query performance by storing columns together.
Detailed Explanation
Downsampling

Downsampling involves reducing the resolution (e.g., from 1-minute to 5-minute intervals) of older data points. This is crucial because recent data often requires higher granularity for analysis, while older data can be coarsened without significantly impacting its usefulness. For example, we might keep hourly metrics for data up to a week old and downsample to daily averages beyond that.

Rollups

Rollup operations aggregate time-series data into summaries over longer periods. For instance, summing up metric values every 5 minutes and storing the result can reduce storage costs significantly while still providing useful aggregated views of the data. This is particularly effective for older data where detailed granularity isn't necessary.

Retention Tiers

Implementing retention tiers means differentiating between how we store recent and old data based on access frequency and cost. Recent data, which may be accessed frequently, can be stored in a more expensive but faster tier (e.g., SSDs), while older data is moved to cheaper storage like HDDs or even cloud-based object storage with lower retrieval costs.

Columnar Layout

A columnar layout optimizes query performance by storing and retrieving data based on columns rather than rows. This is particularly beneficial for range queries, which are common in time-series analysis. In a columnar database, all values of the same attribute (column) are stored together, allowing efficient access to specific ranges of data.

Why Columnar Layout Helps Range Queries

Columnar storage improves query performance by reducing the amount of unnecessary data that needs to be scanned and processed. For example, if you need to retrieve a range of temperature readings for a given sensor over several hours, a row-oriented database would scan an entire row (which might contain many irrelevant columns), whereas a columnar layout can directly access the relevant column.

#### Closing Summary

By implementing downsampling, rollups, and retention tiers, we ensure that recent data is stored with high granularity to meet real-time analysis needs, while older data is managed more cost-effectively. A columnar layout further optimizes storage and query performance by reducing the amount of unnecessary data processing during range queries.

#### Common Mistakes/Follow-Ups
Ignoring Cost Implications: Failing to properly implement retention tiers can lead to excessive costs.
Over-Downsampling/Under-Rollup: Balancing granularity with cost is crucial; over-dimensioning downsampling or rollups can waste storage, while under-dimensioning them could impact query performance.
Focusing Only on Recent Data: Neglecting the need for historical data analysis can lead to incomplete insights and missed opportunities.

**Comparison:**

```
Timestamp	Value
2023-10-01T09:05:00Z	76.1
Timestamp	Value
2023-09-01T09:00:00Z	74.8
```

**Code:**

```python
# Example Python code snippet for downsampling and rollup operations

import pandas as pd

def downsample(df, time_column='timestamp', value_column='value', freq='5T'):
    """Downsamples DataFrame by averaging values over specified frequency."""
    df.set_index(time_column, inplace=True)
    df_resampled = df.resample(freq).mean()
    return df_resampled.reset_index()

def rollup(df, time_column='timestamp', value_column='value', agg_func='sum', freq='D'):
    """Rolls up DataFrame by aggregating values over specified frequency."""
    df.set_index(time_column, inplace=True)
    df_rollup = df.resample(freq).agg(agg_func)
    return df_rollup.reset_index()
```

**Example input / output:**

```text
# Example input and output for downsampling and rollup operations

import pandas as pd

recent_metrics = [
    {"timestamp": "2023-10-01T09:00:00Z", "value": 75.4},
    {"timestamp": "2023-10-01T09:05:00Z", "value": 76.8}
]

historical_metrics = [
    {"timestamp": "2023-09-01T09:00:00Z", "value": 74.1},
    {"timestamp": "2023-09-01T09:05:00Z", "value": 73.6}
]

# Convert to DataFrame
recent_df = pd.DataFrame(recent_metrics)
historical_df = pd.DataFrame(historical_metrics)

# Downsampling recent metrics
downsampled_recent = downsample(recent_df, time_column='timestamp', value_column='value', freq='5T')
print(downsampled_recent)

# Rollup historical metrics
rollup_historical = rollup(historical_df, time_column='timestamp', value_column='value', agg_func='sum', freq='D')
print(rollup_historical)
```

**Data:**

```json
{
  "recent_metrics": [
    {"timestamp": "2023-10-01T09:00:00Z", "value": 75.4},
    {"timestamp": "2023-10-01T09:05:00Z", "value": 76.8},
    ...
  ],
  "historical_metrics": [
    {"timestamp": "2023-09-01T09:00:00Z", "value": 74.1},
    {"timestamp": "2023-09-01T09:05:00Z", "value": 73.6}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Data Source
4. Pre-processing
5. Is Data Recent?
6. Downsample
7. Rollup
8. Retention Tier 1 - SSD
9. Retention Tier 2 - HDD/CloudStorage

