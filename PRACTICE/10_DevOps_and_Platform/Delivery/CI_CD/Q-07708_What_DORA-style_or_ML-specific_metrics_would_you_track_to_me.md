---
type: "practice"
difficulty: "Deep"
---

# What DORA-style or ML-specific metrics would you track to measure the health of your model delivery pipeline (time-to-production, retrain frequency, rollback rate)?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ DORA Metrics](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Deployment_Observability/DORA_Metrics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To measure the health of an ML model delivery pipeline, we need to focus on both the efficiency and robustness of our Continuous Integration/Continuous Deployment (CI/CD) processes. DORA metrics are a great starting point, but for ML-specific pipelines, additional nuances come into play. Here’s what you would track:

Time-to-Production (TTP): This measures how quickly changes can be delivered to production. For an ML model pipeline, this could include the time from data ingestion to deployment.
Rollback Rate: This metric indicates how often deployments fail and require rollback. In an ML context, it might reflect the rate of unsuccessful redeploys due to model failure or performance issues.
Defect Density per Change (DDC): This measures the number of defects in code changes per release. For ML, this could be adjusted to consider the complexity of the model and data.

To further tailor these metrics for an ML pipeline:

Model Retrain Frequency: How often do you need to retrain models? Frequent retraining can indicate issues like concept drift or model degradation.
Model Performance Degradation Rate: Track how quickly a deployed model’s performance drops over time. This could be measured by comparing current performance with historical benchmarks.

**Code:**

```python
# Example code to calculate Model Retrain Frequency and DDC
from datetime import datetime

class PipelineHealthMetrics:
    def __init__(self):
        self.retrain_dates = []
        self.defects = []

    def record_retraining(self, date: datetime):
        self.retrain_dates.append(date)

    def record_defect(self, complexity: int):
        self.defects.append(complexity)

    @property
    def retrain_frequency(self) -> float:
        if not self.retrain_dates:
            return 0
        latest_retrain = max(self.retrain_dates)
        earliest_retrain = min(self.retrain_dates)
        return len(self.retrain_dates) / (latest_retrain - earliest_retrain).days

    @property
    def defect_density_per_change(self) -> float:
        if not self.defects:
            return 0
        total_changes = len(self.retrain_dates)
        return sum(self.defects) / total_changes
```

**Example input / output:**

```text
Input:
{
  "retrain_dates": ["2023-10-01", "2023-10-15", "2023-10-30"],
  "defects": [4, 6, 8]
}

Output:
Retrain Frequency: 0.14 (once every 7 days)
Defect Density per Change: 6
```

**Data:**

```json
{
  "retrain_dates": [
    "2023-10-01",
    "2023-10-15",
    "2023-10-30"
  ],
  "defects": [4, 6, 8]
}
```

