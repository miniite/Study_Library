---
type: "practice"
difficulty: "Deep"
---

# How would you decide between per-region and per-global anomaly models for a metric with regional traffic patterns?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Adaptive Thresholds](<../../../../11_AI_and_ML/Applied_ML/Anomaly_Detection/Adaptive_Thresholds.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide between using a per-region or a per-global anomaly detection model for a metric that exhibits regional traffic patterns, we need to carefully consider several factors, including the nature of the data, the granularity required by business stakeholders, and the computational resources available. This decision involves both technical and strategic considerations.

First, let's define what "per-region" and "per-global" models mean in this context:

Per-region model: The anomaly detection is performed independently for each region.
Per-global model: The anomaly detection is performed across all regions as a single entity.

In the case of regional traffic patterns, we are dealing with data that has significant spatial variability. This means the metric's behavior can differ significantly between regions, making a per-region approach more suitable if local conditions and anomalies need to be understood separately.

#### Triage
Understanding the Question

The interviewer is probing your ability to:

Understand the nature of the data (time series with regional patterns).
Evaluate trade-offs in model granularity (per-region vs. per-global).

#### Cons
ider computational efficiency and resource constraints.
Align the solution with business requirements.

#### First-Principles Solution
Data Characteristics: Regional traffic metrics often show distinct patterns due to local events, regional usage habits, or external factors affecting different areas.
Business Requirements: If stakeholders need insights specific to each region (e.g., for targeted marketing campaigns or localized maintenance), a per-region model is more appropriate.
Computational Trade-offs:
Per-region models: More complex to set up and maintain but can offer finer-grained insights.
Per-global model: Simpler and potentially more computationally efficient, but may not capture regional nuances effectively.
Step-by-Step Solution
Data Exploration:
Analyze the time series data for each region to understand baseline patterns, seasonality, and any known external factors affecting traffic.
Define Anomaly Detection Rules: Establish rules that are appropriate for both per-region and per-global models. These might include statistical thresholds (Z-score, IQR), machine learning-based methods (Isolation Forest, Autoencoders), or rule-based systems.
Model Training:
For a per-region model, train individual models on data from each region using the chosen anomaly detection rules.
For a per-global model, train one unified model across all regions but adjust it to account for regional differences (e.g., by incorporating additional features like time of day or day of week).
Evaluation:
Use historical data to evaluate both models' performance. Metrics could include precision, recall, F1 score, and business impact.
Deployment and Monitoring: Deploy the model in a production environment and continuously monitor its performance. Adjust thresholds as necessary based on new data.

#### Closing Summary

In summary, for regional traffic patterns, a per-region anomaly detection model is generally more appropriate because it allows for localized insights that are crucial for decision-making at different geographic levels. However, the choice should be validated through careful analysis of business requirements and computational constraints.

#### Common Mistakes/Follow-Ups
Failing to consider local patterns: Overlooking regional differences can lead to less accurate anomaly detection.
Ignoring computational efficiency: A per-region model might require more resources than a global one, which could impact scalability.
Misaligning with business needs: Ensuring the chosen model meets stakeholder requirements is crucial for success.

**Comparison:**

| Region | Traffic Data (Day 1) | Z-Score |
|--------|---------------------|---------|
| NYC    | 20                  | -0.67   |
| LA     | 30                  | -0.54   |
| CHI    | 15                  | -0.67   |
| Region | Traffic Data (Day 2) | Z-Score |
|--------|---------------------|---------|
| NYC    | 35                  | 1.33    |
| LA     | 45                  | 0.89    |
| CHI    | 25                  | 0.34    |

**Code:**

```python
# Example of defining a Z-score threshold for anomaly detection
def z_score_threshold(data, threshold=3):
    return np.abs((data - data.mean()) / data.std()) > threshold

# Example training a per-region model using scikit-learn's IsolationForest
from sklearn.ensemble import IsolationForest
import numpy as np

def train_isolation_forest_per_region(region_data):
    model = IsolationForest(contamination=0.01)
    model.fit(region_data)
    return model
```

**Example input / output:**

```text
Input:
- NYC traffic data: [20, 35, 40, 25, 30]
- LA traffic data: [30, 45, 40, 35, 40]
- CHI traffic data: [15, 25, 30, 20, 22]

Output:
- NYC Z-score threshold model
- LA Z-score threshold model
- CHI Z-score threshold model

Dry-run:
For NYC:
- Day 1: Z-score = -0.67
- Day 2: Z-score = 1.33 (Anomaly detected)

For LA:
- Day 1: Z-score = -0.54
- Day 2: Z-score = 0.89

For CHI:
- Day 1: Z-score = -0.67
- Day 2: Z-score = 0.34
```

**Data:**

```json
{
  "region_traffic": {
    "NYC": [20, 35, 40, 25, 30],
    "LA": [30, 45, 50, 35, 40],
    "CHI": [15, 25, 30, 20, 22]
  }
}
```

**Flow:**

1. Define Data Characteristics
2. Explore Data
3. Identify Regional Patterns
4. Model Choice?
5. Per-Region
6. Train Per-Region Models
7. Global
8. Train Global Model
9. Evaluate Performance
10. Evaluate Performance
11. Maintain and Monitor
12. Maintain and Monitor

