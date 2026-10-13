---
type: "practice"
difficulty: "Expert"
---

# A bar-raiser probes your forecasting: your bottoms-up ML-compute forecast (driven by model roadmap, expected traffic, and training runs) and finance's top-down GPU-spend model diverge 30% every quarter. Design the reconciliation — which model owns which drivers and how you converge them.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cost Forecasting](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Cost_Forecasting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To reconcile the divergence between your bottoms-up ML-compute forecast (driven by model roadmap, expected traffic, and training runs) and finance's top-down GPU-spend model, we need to ensure that both models are aligned on key drivers and have a clear understanding of each other’s assumptions. The goal is to converge these forecasts so they reflect the same underlying data and business goals.

#### Triage

The question essentially asks you to design a process for aligning two different forecasting models: one driven by detailed, bottom-up modeling (ML-compute forecast) and another driven by broader, top-down financial analysis (GPU-spend model). The divergence of 30% every quarter indicates potential misalignment in assumptions or data inputs.

#### Assumptions
Data Accuracy: Both models are using accurate but potentially different datasets.
Temporal Consistency: Both forecasts need to be consistent over time.
Model Transparency: Both ML and financial teams have visibility into the underlying assumptions of each model.
Step-by-Step Solution
1. Identify Key Drivers

First, identify the key drivers in both models:

ML-Compute Forecast:

Model roadmap (number and type of models to be developed).
Expected traffic growth.
Training runs per model.

GPU-Spend Model:

Total GPU capacity.
Usage patterns across different teams/projects.
2. Mapping Drivers

Map the drivers from both forecasts to find common ground:

Model Roadmap: Ensure both models consider the same set of upcoming projects and their timelines.
Traffic Growth: Validate expected traffic growth with historical data or other predictive signals.
Training Runs: Account for the number and duration of training runs, especially in high-load periods.
3. Convergence Strategy

Propose a strategy to converge these drivers:

Regular Reviews: Conduct quarterly reviews where both teams discuss their forecasts and identify discrepancies.
Data Integration: Integrate data from both models into a common repository for transparency.
Automated Checks: Use automation tools to periodically validate assumptions across both models.
4. Implementation

Implement the convergence strategy with specific steps:

Set up a Common Dashboard:
Create a dashboard that combines key metrics from both forecasts.
Regular Meetings:
Schedule regular (e.g., monthly) meetings between ML and finance teams to review forecast discrepancies.
Automated Validation:
Develop scripts to automatically cross-verify data points from both models.
5. Monitoring and Feedback

Continuous monitoring and iterative improvement are essential:

Feedback Loops: Implement feedback loops where findings from one model can inform improvements in the other.
Performance Metrics: Define key performance metrics (KPMs) such as forecast accuracy, response time, and cost savings to track progress.

#### Closing Summary

By aligning the drivers of both models through regular reviews, data integration, and automated checks, we can ensure that both forecasts reflect a consistent understanding of the underlying business environment. This approach not only reduces divergence but also enhances overall forecasting reliability.

#### Common Mistakes/Follow-Ups
Ignoring Historical Data: Failing to validate assumptions with historical trends.
Lack of Transparency: Not involving key stakeholders from both ML and finance teams.
Over-reliance on Automation: Relying solely on automated tools without human oversight for critical decision-making.

**Comparison:**

```
project_id	model_traffic	gpu_usage
1	1000	900
2	500	400
Discrepancies Found
Project ID: 1
Model Traffic: 1000
GPU Usage: 900
Discrepancy: +10%
---------------------
Project ID: 2
Model Traffic: 500
GPU Usage: 400
Discrepancy: -20%
```

**Code:**

```python
# Example Python script for automated validation
import pandas as pd

def validate_forecasts(model_compute_df, gpu_spend_df):
    # Join datasets based on common key (e.g., project ID)
    combined_df = model_compute_df.merge(gpu_spend_df, on='project_id')

    # Check for discrepancies in key metrics
    discrepancies = combined_df[combined_df['model_traffic'] != combined_df['gpu_usage']]

    return discrepancies

# Sample data
model_compute_data = pd.DataFrame({
    'project_id': [1, 2],
    'model_traffic': [1000, 500]
})

gpu_spend_data = pd.DataFrame({
    'project_id': [1, 2],
    'gpu_usage': [900, 400]
})
discrepancies = validate_forecasts(model_compute_data, gpu_spend_data)
print(discrepancies)
```

**Example input / output:**

```text
+----------+------------+------------+
| project_id | model_traffic | gpu_usage  |
+----------+------------+------------+
| 1         | 1000        |    900      |
| 2         | 500         |    400      |
+----------+------------+------------+
```

**Data:**

```json
{
  "model_compute": {
    "project_id": [1, 2],
    "model_traffic": [1000, 500]
  },
  "gpu_spend": {
    "project_id": [1, 2],
    "gpu_usage": [900, 400]
  }
}
```

**Flow:**

1. No
2. Identify Key Drivers
3. Are drivers aligned?
4. Conduct Regular Reviews
5. Integrate Data
6. Develop Automated Checks
7. Monitor and Improve
8. Continuous ImprovementLoop

