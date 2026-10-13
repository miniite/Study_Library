---
type: "practice"
difficulty: "Intermediate"
---

# Why might you normalize or scale metrics before feeding them into an anomaly detection model?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Multivariate Anomaly Correlation](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Multivariate_Anomaly_Correlation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with time series data in anomaly detection, normalization and scaling are essential preprocessing steps that can significantly impact the performance of your models. This is because most anomaly detection algorithms assume a certain distribution or scale for the input data, and without proper normalization, these assumptions may not hold true. In this overview, we will discuss why normalization and scaling are necessary, the specific techniques involved, and how to apply them effectively in an AIOps environment.

Normalization and scaling help in making sure that all features contribute equally to the model training process by reducing the range of values. For instance, if you have metrics with vastly different scales, such as CPU usage (0-100%) and memory usage (0-2^32), the feature with a larger scale might dominate the learning process, leading to suboptimal performance.

**Code:**

```python
from sklearn.preprocessing import MinMaxScaler

# Example data
data = [[10], [20], [50], [100]]

# Initialize and fit scaler
scaler = MinMaxScaler()
scaled_data = scaler.fit_transform(data)

print("Original Data:", data)
print("Scaled Data:", scaled_data)
```

**Example input / output:**

```text
Original Data: [[10], [20], [50], [100]]
Scaled Data: [[0.  ] [0.25] [0.75] [1.  ]]
```

**Flow:**

1. No
2. Yes
3. Original Data
4. Is data normalized?
5. Apply normalization
6. Scaled Data

