---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between per-host and per-service granularity, and how does your choice change the preprocessing you do?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Cost of Data Cleaning](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Cost_of_Data_Cleaning.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with AIOps data preprocessing and cleaning, understanding the difference between per-host and per-service granularity is crucial. Per-host granularity means focusing on individual hosts or machines, while per-service granularity involves looking at services running across multiple hosts. The choice of granularity significantly impacts how you preprocess and clean your data.

The core idea here is that the nature of the data and the use case determine which granularity to choose. For instance, if you're monitoring network traffic, per-host granularity might be more appropriate because each host will have unique traffic patterns. Conversely, if you’re tracking service performance, per-service granularity could be better as it aggregates data across all instances of a service.

To summarize:

Per-host focuses on individual machines.
Per-service looks at services regardless of the hosts they run on.

Let's break this down step by step from first principles:

#### Triaging the Question

The interviewer is probing your understanding of data granularity and its implications for preprocessing. You need to explain how different granularities affect the cleaning process, including handling anomalies, normalizing data, and ensuring consistency across data points.

#### Assumptions
We are working with time-series data from monitoring systems.
The goal is to clean and preprocess data before feeding it into AIOps tools for anomaly detection or other analysis.

#### First-principles Solution
Per-host Granularity:
Data Collection: Each host collects metrics independently, leading to a variety of unique patterns across different hosts.
Cleaning Steps:
Normalization: Standardize units and scales (e.g., converting CPU usage from percent to a fixed scale).
Handling Missing Data: Impute or interpolate missing values based on historical data.
Outlier Detection: Use statistical methods like Z-score to identify and handle outliers specific to each host.
Per-service Granularity:
Data Collection: Metrics are aggregated across all instances of the same service, leading to a more uniform pattern compared to per-host metrics.
Cleaning Steps:
Normalization: Standardize units but consider scaling based on the average behavior of the service.
Handling Missing Data: Use methods that account for the overall traffic patterns of the service.
Outlier Detection: Detect anomalies in a broader context, considering the aggregate data and historical trends.

#### Closing Summary

The choice between per-host and per-service granularity depends heavily on the specific use case. Per-host granularity is suitable when you need to analyze individual machine performance, while per-service granularity is better for service-level monitoring where aggregation across hosts is more meaningful.

#### Common Mistakes/Follow-ups
Misunderstanding Granularity: Failing to recognize the difference between host-specific and service-wide metrics.
Ignoring Historical Context: Overlooking the importance of historical data in identifying anomalies, which can vary significantly by granularity.
Not Considering Aggregation Methods: Ignoring the need for different aggregation methods (e.g., averaging vs. summing) based on granularity.
Triaging Follow-ups
What are some real-world examples where per-host or per-service granularity might be more appropriate?
How would you handle data skewness in a mixed-granularity dataset?
Can you explain the trade-offs between these two approaches in terms of accuracy and performance?

**Comparison:**

| Granularity | Host1 (normalized) | Host2 (normalized) | ServiceA/B/C (normalized) |
|-------------|--------------------|-------------------|--------------------------|
| CPU Normalized | -0.6745, 1.3416, -0.6745 | -0.297, -1.342, -1.613 | 0.297, -1.342, -1.613 |

**Code:**

```python
# Example code snippet for normalizing CPU usage (per-host)
import pandas as pd

def normalize_cpu_usage(df, granularity='host'):
    if granularity == 'host':
        # Normalize based on host-specific patterns
        df['cpu_normalized'] = df.groupby('hostname')['cpu_usage'].transform(lambda x: (x - x.mean()) / x.std())
    elif granularity == 'service':
        # Normalize based on service-wide patterns
        df['cpu_normalized'] = df.groupby(['service_name', pd.Grouper(key='timestamp', freq='H')])['cpu_usage'].transform(lambda x: (x - x.mean()) / x.std())
    return df

# Example data
data = {
    'hostname': ['host1', 'host2', 'host3'],
    'timestamp': [1, 2, 3],
    'service_name': ['serviceA', 'serviceB', 'serviceC'],
    'cpu_usage': [80, 90, 75]
}

df = pd.DataFrame(data)
normalized_df_host = normalize_cpu_usage(df, granularity='host')
print(normalized_df_host)

# Output:
#   hostname  timestamp service_name  cpu_usage  cpu_normalized
# 0     host1          1    serviceA        80           -0.6745
# 1     host2          2    serviceB        90            1.3416
# 2     host3          3    serviceC        75           -0.6745
```

**Example input / output:**

```text
Input:
hostname  timestamp  service_name  cpu_usage
host1         1       serviceA          80
host2         2       serviceB          90
host3         3       serviceC          75

Output (per-host):
hostname  timestamp  service_name  cpu_usage  cpu_normalized
host1         1       serviceA          80           -0.6745
host2         2       serviceB          90            1.3416
host3         3       serviceC          75           -0.6745

Output (per-service):
hostname  timestamp  service_name  cpu_usage  cpu_normalized
host1         1       serviceA          80              0.297
host2         2       serviceB          90             -1.342
host3         3       serviceC          75             -1.613
```

**Data:**

```json
{
    "data": {
        "hostname": ["host1", "host2", "host3"],
        "timestamp": [1, 2, 3],
        "service_name": ["serviceA", "serviceB", "serviceC"],
        "cpu_usage": [80, 90, 75]
    }
}
```

**Flow:**

1. Data Collection
2. Granularity Choice
3. Per-Host Processing
4. Per-Service Processing
5. Host-Specific Normalization
6. Handling Missing Data
7. Detecting Outliers
8. Service-Level Normalization
9. Historical ContextConsideration
10. Aggregation Methods

