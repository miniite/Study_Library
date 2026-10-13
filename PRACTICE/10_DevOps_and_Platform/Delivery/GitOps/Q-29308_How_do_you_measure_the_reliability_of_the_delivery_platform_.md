---
type: "practice"
difficulty: "Deep"
---

# How do you measure the reliability of the delivery platform as a product — adoption, reconcile latency percentiles, drift-MTTR — and report it to leadership as a scorecard?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Sync Notifications](<../../../../10_DevOps_and_Platform/Delivery/GitOps/GitOps_Fundamentals/Sync_Notifications.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To measure the reliability of a GitOps-based delivery platform, we need to focus on several key metrics that capture different aspects of its performance and stability. These include adoption, reconciliation latency percentiles, drift-MTTR (Mean Time To Revert), and overall system health scores. The goal is not just to gather these metrics but also to provide actionable insights to leadership in the form of a scorecard.

Firstly, let's break down what each of these metrics represents:

Adoption: This metric measures how widely used the GitOps platform is across different teams or environments.
Reconciliation Latency Percentiles: This reflects how quickly changes are applied to target environments after they're committed to the Git repository.
Drift-MTTR (Mean Time To Revert): This measures the time taken to revert a change that has caused issues in production.

To provide this information as a scorecard, we need to set up a comprehensive monitoring and reporting system. We'll start by defining how each metric is calculated and then discuss how they can be aggregated into a single, actionable score.

**Comparison:**

| Metric     | Thresholds              | Current Status  |
|------------|-------------------------|-----------------|
| Adoption   | >=80% for healthy status| Dev-Team 90%, Ops-Team 85% |
| Latency    | <100ms                  | 75ms           |
| MTTR       | <24h                    | N/A            |

**Code:**

```python
# Example Python code for calculating reconciliation latency percentiles
import pandas as pd

def calculate_reconciliation_latency_percentiles(latencies):
    latencies_df = pd.DataFrame({'latency': latencies})
    percentile_95 = latencies_df['latency'].quantile(0.95)
    return percentile_95

# Example data for reconciliation latency
reconciliation_latencies = [30, 45, 60, 75, 90, 120, 150, 180]
percentile_95 = calculate_reconciliation_latency_percentiles(reconciliation_latencies)
print(f"95th Percentile Reconciliation Latency: {percentile_95}")
```

**Example input / output:**

| Metric        | Data                 | Score (0-10) |
|---------------|----------------------|-------------|
| Adoption      | Dev-Team: 90%, Ops-Team: 85% | 8.5         |
| Reconciliation Latency Percentiles   | 95th percentile: 120s, Mean: 75s       | 6.8        |
| Drift-MTTR    | Recent Change Reverts: 30m & 12h | 4           |

**Data:**

```json
{
  "adoption": {
    "teams": [
      {"name": "Dev-Team", "gitOpsUsage": "90%", "environments": ["Prod", "Staging"]},
      {"name": "Ops-Team", "gitOpsUsage": "85%", "environments": ["Prod"]}
    ]
  },
  "reconciliationLatencyPercentiles": {
    "percentile_95": 120,
    "mean": 75
  },
  "driftMTTR": {
    "recentChangeReverts": [
      {"issue": "Database Schema Change", "timeToRevert": "30m"},
      {"issue": "UI Bug Fix", "timeToRevert": "12h"}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Adoption
8. Is above threshold?
9. Score +2
10. Provide training/awareness
11. Calculate Latency Percentiles
12. Latency < 100ms?
13. Score +2
14. Investigate and optimize
15. Drift-MTTR Calculation
16. MTTR < 24h?
17. Score +3
18. Implement automatedrollbacks/monitoring
19. Aggregate Scores
20. Report to Leadership withInsights

