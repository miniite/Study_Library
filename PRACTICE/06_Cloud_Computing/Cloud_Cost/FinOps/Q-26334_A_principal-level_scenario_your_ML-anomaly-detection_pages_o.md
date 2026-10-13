---
type: "practice"
difficulty: "Deep"
---

# A principal-level scenario: your ML-anomaly-detection pages on every legitimate large training run but misses a slow $150K/month leak from an inference fleet that never scales down off-peak. Redesign the detection so it understands ML-workload context — expected training spikes versus unexpected inference idle.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cost Anomaly Detection](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Cost_Anomaly_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary TL;DR

In this scenario, we need to redesign our FinOps and cloud cost governance system to better understand the context of machine learning (ML) workloads—specifically, distinguishing between expected training spikes and unexpected inference idle times that lead to significant off-peak costs. The goal is to ensure that anomalies are detected accurately while minimizing false positives.

#### Triaging the Question

The core question revolves around designing a cost governance system that can differentiate between expected fluctuations in ML workloads (like training runs) and abnormal behaviors indicative of inefficiencies, such as constant inference activity without proper scaling. This involves understanding the typical patterns of both training and inference to establish baselines for normal operation.

#### Assumptions
Historical Data Availability: We have access to historical data on resource usage, including training runs and inference activities.
Resource Types: The system will need to monitor CPU, GPU, memory, and network resources.
Cost Metrics: The cost of each type of resource is known and can be aggregated.

#### First-Principles Solution

Historical Baseline Establishment:

Collect and analyze historical data on both training runs and inference activities over a period that covers typical usage patterns (e.g., a month or more).
Segment the data by time periods to capture daily, weekly, and monthly trends.

Training Workload Identification:

Develop an algorithm to identify when training is happening by analyzing resource utilization spikes.
Use machine learning models (like anomaly detection) to predict typical training run patterns based on historical data.

Inference Activity Monitoring:

Identify the baseline for inference activity during off-peak hours.
Implement a threshold-based system to flag instances where inference usage significantly exceeds this baseline.

Behavioral Analysis:

Use behavioral analysis techniques to detect patterns that deviate from normal training and inference behaviors.
For example, if inference remains active even when it should be scaling down during off-peak hours, mark it as a potential anomaly.

Cost Anomaly Detection:

Integrate cost data with resource usage data to ensure that high costs correlate with significant resource consumption.
Flag instances where costs are incurred disproportionately without corresponding resource utilization (e.g., inference activity without scaling down).

Alerting and Reporting:

Set up alerts for detected anomalies so that stakeholders can investigate and take corrective actions.
Generate detailed reports to help teams understand the root causes of anomalies.

#### Closing Summary

To address the issue, we need a robust system that can dynamically adapt to different ML workloads by understanding their typical behaviors. This involves establishing historical baselines, using machine learning for anomaly detection, and integrating cost data for accurate anomaly identification. By implementing these steps, we can reduce false positives and ensure more effective cost governance.

#### Common Mistakes
and Follow-Ups
False Positives: Avoid overfitting to historical patterns by ensuring the system is flexible enough to adapt to new normal behaviors.
Resource Overprovisioning: Ensure that scaling policies are well-defined and enforced to prevent unnecessary costs.
Data Quality: The accuracy of the solution heavily depends on the quality and availability of historical data. Poor data can lead to incorrect baselines and anomalies.
Triaging Follow-Ups

What if there is no historical data?:

If no historical data exists, consider using external benchmarks or industry standards for initial baselines.

How will the system handle seasonal changes in ML workloads?:

Seasonal adjustments can be incorporated into the baseline establishment process to account for periodic variations.

What if there are multiple types of inference activities?:

Segregate different types of inference activities and establish separate baselines for each, or use clustering techniques to group similar behaviors together.

**Comparison:**

```
Metric	Baseline Value	Notes
Average CPU Usage	60%	During off-peak hours
GPU Memory Utilization	75MB	During peak inference periods
```

**Code:**

```python
# Example anomaly detection model using scikit-learn

from sklearn.ensemble import IsolationForest
import pandas as pd

def train_isolation_forest(training_data):
    # Assume training_data is a DataFrame with resource usage features
    clf = IsolationForest(contamination=0.1, random_state=42)
    clf.fit(training_data)
    return clf

def detect_anomalies(inference_data, trained_model):
    predictions = trained_model.predict(inference_data)
    anomalies = inference_data[predictions == -1]
    return anomalies
```

**Example input / output:**

| Input Data          | Output (Anomalies)                         |
|---------------------|-------------------------------------------|
| Training data       | No anomalies detected                     |
| Inference data      | Anomalies detected at [timestamp]         |

**Data:**

```json
{
  "training_data": {
    "timestamp": ["2023-10-01", "2023-10-02", ...],
    "cpu_usage": [85, 75, ...],
    "gpu_usage": [90, 85, ...]
  },
  "inference_data": {
    "timestamp": ["2023-10-01", "2023-10-02", ...],
    "cpu_usage": [50, 45, ...],
    "gpu_usage": [30, 25, ...]
  }
}
```

**Flow:**

1. Yes
2. No
3. Historical Data Collection
4. Is there enough historicaldata?
5. Establish Baselines
6. Use External Benchmarks
7. Develop Anomaly DetectionModel
8. Use External Benchmarks forInitial Baseline
9. Train Isolation Forest Model
10. Adapt to New NormalBehaviors
11. Detect Anomalies
12. Alert Stakeholders

