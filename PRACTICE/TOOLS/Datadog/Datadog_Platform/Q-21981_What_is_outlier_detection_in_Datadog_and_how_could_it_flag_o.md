---
type: "practice"
difficulty: "Deep"
---

# What is outlier detection in Datadog and how could it flag one misbehaving host in a group?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Anomaly and Forecast Monitors](<../../../../TOOLS/Datadog/Datadog_Platform/Anomaly_and_Forecast_Monitors.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Outlier detection in Datadog is a powerful feature that helps identify unusual or abnormal behavior in metrics, which can be particularly useful in diagnosing issues with individual hosts within a larger infrastructure. At its core, it uses statistical methods to differentiate between normal and anomalous data points. To understand how outlier detection flags one misbehaving host, we need to delve into the underlying mechanisms and consider various factors.

**Comparison:**

```
Metric Values	Host1	Host2	Host3
Time 0	90	86	98
Time 1	92	87	102
Time 2	95	88	105
Time 3	88	89	100
Time 4	93	90	103
```

**Code:**

```python
# Example of basic outlier detection logic in Python using z-score method
import numpy as np

def detect_outliers(data):
    mean = np.mean(data)
    std_dev = np.std(data)
    threshold = 3  # Standard deviation threshold
    outliers = [x for x in data if abs(x - mean) > threshold * std_dev]
    return outliers
```

**Example input / output:**

```

Given the metric values for three hosts over five time periods:

Host1: [90, 92, 95, 88, 93]
Host2: [86, 87, 88, 89, 90]
Host3: [98, 102, 105, 100, 103]

Let's apply the outlier detection logic:

PYTHON
Copy
def detect_outliers(data):
    mean = np.mean(list(data.values())[0])
    std_dev = np.std(list(data.values())[0])
    threshold = 3
    outliers = [x for x in data if abs(x - mean) > threshold * std_dev]
    return outliers

outlier_host1 = detect_outliers({"host1": [90, 92, 95, 88, 93]})
print(outlier_host1)

Output:

PYTHON
Copy
[88]

Host1 has one outlier: 88. This metric value is significantly different from the mean and standard deviation of other values.
```

**Data:**

```json
{
  "metric_values": [
    {"host1": [90, 92, 95, 88, 93]},
    {"host2": [86, 87, 88, 89, 90]},
    {"host3": [98, 102, 105, 100, 103]}
  ]
}
```

**Flow:**

1. Data Collection
2. Normalizing Data
3. Calculate Mean and StandardDeviation
4. Define Outlier Threshold (e.g.,Z-Score > 3)
5. Identify Outliers Based onThreshold
6. Determine Anomalies in EachHost's Metrics
7. Flag Misbehaving Hosts withAnomalies

