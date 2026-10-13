---
type: "practice"
difficulty: "Deep"
---

# How do you detect and alert on data drift in a feature table feeding a production ML model?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distribution Change Alerts](<../../../../02_Data_Engineering/Data_Quality/Data_Observability/Distribution_Change_Alerts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To detect and alert on data drift in a feature table feeding a production ML model, we need to establish a robust monitoring framework that continuously tracks changes in the distribution of features over time. This involves setting up statistical tests to measure the drift, defining thresholds for when an alert should be triggered, and implementing a notification mechanism to inform stakeholders.

#### Triaging the Question

The core challenge here is ensuring the ML model's performance remains stable by maintaining consistent feature distributions. The question probes our ability to set up comprehensive observability practices, handle false positives and negatives effectively, and ensure timely alerts for actionable insights. Assumptions include having access to historical data, the availability of statistical libraries, and a clear understanding of the business context.

#### First-Principles Solution
Data Collection: Collect feature data from production environments.
Baseline Establishment: Use historical data to establish baselines for each feature.
Drift Detection: Implement drift detection techniques using statistical tests like Kolmogorov-Smirnov (KS) test or Earth Mover's Distance (EMD).
Threshold Definition: Define thresholds based on business requirements and statistical significance levels.
Notification Mechanism: Set up alerts to notify stakeholders when drift exceeds predefined thresholds.
Step-by-Step Solution

Data Collection:

Extract feature data from the production environment, storing it in a monitoring database or time-series system like Prometheus or InfluxDB.

Baseline Establishment:

Use historical data to calculate statistical summaries (mean, variance, percentiles) for each feature.
Store these summaries as baselines.

Drift Detection:

For new incoming data, apply drift detection techniques:
PYTHON
from scipy.stats import ks_2samp

def detect_drift(old_data, new_data):
    stat, p_value = ks_2samp(old_data, new_data)
    return p_value < 0.05  # Example threshold for significance

Threshold Definition:

Define thresholds based on the criticality of features and business requirements.
For example, if a feature's drift significantly impacts model performance, set stricter thresholds.

Notification Mechanism:

Set up alerts using tools like Slack, email, or SMS when drift exceeds thresholds.

#### Example
alert setup in Python:
PYTHON
import requests

def send_alert(feature_name, threshold):
    if p_value < threshold:
        response = requests.post(
            "https://api.slack.com/webhooks",
            json={"text": f"Drift detected for {feature_name}!"}
        )
        print(f"Alert sent: {response.status_code}")

#### Closing Summary

Implementing a robust data drift detection system requires a multi-step approach, from collecting and storing feature data to setting up statistical tests and alert mechanisms. Ensuring timely alerts is crucial for maintaining model performance in production environments.

#### Common Mistakes
and Follow-Ups
False Positives: Overly aggressive thresholds can lead to frequent false positives.
Alert Fatigue: Constant alerts might make stakeholders ignore important signals.
Inadequate Baselines: Using outdated or insufficient historical data can mislead drift detection.
Lag in Data Collection: Delayed data collection can result in delayed detections.

#### Follow-ups
could involve discussing advanced techniques like incremental learning, retraining models, and integrating with CI/CD pipelines for automated model updates.

**Comparison:**

| Feature Name | Old Data Mean | New Data Mean |
|--------------|---------------|---------------|
| feature_a     | 5.0           | 5.2           |
| feature_b     | 7.8           | 7.6           |

**Code:**

```python
import numpy as np
from scipy.stats import ks_2samp

def calculate_drift(old_data, new_data):
    stat, p_value = ks_2samp(old_data, new_data)
    return p_value

def setup_monitoring_pipeline(feature_table, historical_features):
    drift_threshold = 0.05
    for feature_name in feature_table.columns:
        old_data = historical_features[feature_name]
        new_data = feature_table[feature_name]
        if calculate_drift(old_data, new_data) < drift_threshold:
            send_alert(feature_name, drift_threshold)

def send_alert(feature_name, threshold):
    response = requests.post(
        "https://api.slack.com/webhooks",
        json={"text": f"Drift detected for {feature_name}!"}
    )
    print(f"Alert sent: {response.status_code}")

# Example usage
historical_features = pd.read_csv("path/to/historical_data.csv")
feature_table = pd.read_csv("path/to/production_data.csv")
setup_monitoring_pipeline(feature_table, historical_features)
```

**Example input / output:**

```text
Input:
Feature Table (New Data):
timestamp | feature_a | feature_b
2023-10-01| 5.2       | 7.6
2023-10-02| 4.8       | 7.9

Historical Features:
timestamp | feature_a | feature_b
2023-09-01| 5.0       | 7.8
2023-09-02| 4.9       | 7.5

Output (Detection):
Drift detected for feature_a!
```

**Data:**

```json
{
  "feature_table": {
    "timestamp": ["2023-10-01", "2023-10-02", ...],
    "feature_a": [5.2, 4.8, ...],
    "feature_b": [7.6, 7.9, ...]
  },
  "historical_features": {
    "timestamp": ["2023-09-01", "2023-09-02", ...],
    "feature_a": [5.0, 4.9, ...],
    "feature_b": [7.8, 7.5, ...]
  }
}
```

**Flow:**

1. Yes
2. No
3. Data Collection
4. Baseline Establishment
5. Drift Detection (KS Test)
6. Drift Exceeded Threshold?
7. Send Alert
8. Monitor Continuously

