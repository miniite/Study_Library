---
type: "practice"
difficulty: "Intermediate"
---

# What is the interquartile range (IQR) method for finding outliers, and how would you apply it in Python to a list of latency values?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Simple Anomaly Detection](<../../../../TOOLS/Python/Python_Language/Simple_Anomaly_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The interquartile range (IQR) method is a statistical technique used to identify outliers in a dataset by leveraging the median and quartiles. Essentially, IQR defines an interval that contains the middle 50% of data points, and any value outside this interval can be considered an outlier. In AIOps, detecting anomalies like latency spikes is crucial for maintaining system health.

To apply the IQR method in Python to a list of latency values, we need to:

Calculate the first quartile (Q1) and third quartile (Q3).
Compute the interquartile range as ( \text{IQR} = Q3 - Q1 ).
Define the lower and upper bounds for outliers: typically, any value below ( Q1 - 1.5 \times \text{IQR} ) or above ( Q3 + 1.5 \times \text{IQR} ) is considered an outlier.

Let's walk through each step in detail.

**Comparison:**

| Latency | Quartile |
|---------|----------|
| 20      | Q1       |
| 30      | Q1       |
| 45      | Median   |
| 60      | Q3       |
| 70      | Q3       |
| 85      | Q3       |
| 100     |          |
| 300     | Outlier  |

**Code:**

```python
import numpy as np

def calculate_iqr_outliers(latencies):
    # Step 1: Calculate the first and third quartiles
    Q1 = np.percentile(latencies, 25)
    Q3 = np.percentile(latencies, 75)

    # Step 2: Compute the IQR
    iqr = Q3 - Q1

    # Step 3: Define lower and upper bounds for outliers
    lower_bound = Q1 - 1.5 * iqr
    upper_bound = Q3 + 1.5 * iqr

    # Step 4: Identify outliers
    outliers = [latency for latency in latencies if (latency < lower_bound or latency > upper_bound)]

    return outliers, lower_bound, upper_bound

# Example usage:
latencies = [20, 30, 45, 60, 70, 85, 100, 300]
outliers, lb, ub = calculate_iqr_outliers(latencies)
print(f"Outliers: {outliers}")
print(f"Lower Bound: {lb}, Upper Bound: {ub}")
```

**Example input / output:**

```text
Given the example latencies `[20, 30, 45, 60, 70, 85, 100, 300]`:
- The lower bound is \( Q1 - 1.5 \times \text{IQR} = 60 - 1.5 \times 20 = 30 \)
- The upper bound is \( Q3 + 1.5 \times \text{IQR} = 85 + 1.5 \times 20 = 115 \)
- Therefore, the outliers are `[300]`.
```

**Data:**

```json
{
    "latencies": [20, 30, 45, 60, 70, 85, 100, 300]
}
```

**Flow:**

1. Input Latencies
2. Sort Latencies
3. Calculate Q1 and Q3
4. IQR = Q3 - Q1
5. Lower Bound = Q1 - 1.5 * IQR
6. Upper Bound = Q3 + 1.5 * IQR
7. Identify Outliers: x < LowerBound or x > Upper Bound
8. Output Outliers and Bounds

