---
type: "practice"
difficulty: "Deep"
---

# What is RobustScaler and when is it preferable to StandardScaler?

**Difficulty:** Deep · **Levels:** Fresher · [↑ Seasonal Baselines](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Seasonal_Baselines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

RobustScaler and StandardScaler are both preprocessing techniques used in machine learning pipelines, but they serve different purposes. In a nutshell, RobustScaler is preferable when dealing with datasets that contain outliers or extreme values, as it scales features using the interquartile range (IQR) rather than the mean and standard deviation.

The core idea here is to understand the differences between these two scalers and their respective use cases. Let's break down each concept from first principles and explore why RobustScaler can be a better choice in certain scenarios.

**Comparison:**

| Original Data | StandardScaler | RobustScaler |
|---------------|----------------|--------------|
| [10, 3]        | [-0.95, -0.68] | [-0.475, -0.295] |
| [4, 8]         | [-0.68, 1.03]  | [-0.76, 0.43]   |
| [9, 5]         | [-0.5, 0.53]   | [-0.58, 0.28]   |
| [167, 2]       | [1.79, -0.31]  | [4.18, -1.69]   |
| [100, 1]       | [1.66, -0.75]  | [2.67, -1.19]   |

**Code:**

```python
from sklearn.preprocessing import StandardScaler, RobustScaler
import numpy as np

# Sample data with outliers
data = np.array([[10, 3], [4, 8], [9, 5], [167, 2], [100, 1]])
```

**Example input / output:**

```text
Given the sample data:
- Original Data: \([10, 3], [4, 8], [9, 5], [167, 2], [100, 1]\)
We can see that there is an outlier at `[167, 2]`.

Let's apply both scalers to this data and observe the differences.
```

**Data:**

```json
{
    "data": [
        [10, 3],
        [4, 8],
        [9, 5],
        [167, 2],
        [100, 1]
    ]
}
```

**Flow:**

1. Data with Outliers
2. Apply StandardScaler
3. Features scaled based onmean and std deviation
4. Outliers get extreme values
5. Data with Outliers
6. Apply RobustScaler
7. Features scaled based on IQR
8. Outliers have less impact

