---
type: "practice"
difficulty: "Deep"
---

# Design an anomaly-detection layer over Vault audit logs to catch a service reading 10x its baseline or a human enumerating paths. What signals matter, and how do you keep false positives from causing alert fatigue?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Vault Outage Resilience](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Vault_Outage_Resilience.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an anomaly-detection layer over Vault audit logs, we need to identify patterns that indicate unusual activity, such as a service reading 10x its baseline or a human enumerating paths. The key signals include frequency of access, rate of requests, and path enumeration. We will use statistical methods and machine learning to detect anomalies while minimizing false positives.

#### Triaging the Question

The interviewer is probing your understanding of:

Audit Log Analysis: How to interpret Vault audit logs.
Statistical Methods: Techniques for detecting outliers in access patterns.
Machine Learning Models: Using models like anomaly detection or clustering.
False Positive Management: Ensuring that the system does not alert on benign activity.

#### Assumptions
We have access to Vault audit logs, which include timestamps, service names, paths accessed, and user actions.
The baseline for normal behavior is established over time.
We can use historical data to train our models.
We are using a cloud-based monitoring tool that supports real-time anomaly detection.
Step-by-Step Solution

Data Collection: Collect Vault audit logs in a centralized location such as AWS CloudWatch or Azure Monitor.

Baseline Establishment: Calculate the baseline access patterns for each service and user over time.

Feature Engineering:

Frequency of Access: Count requests per unit time (e.g., per minute).
Rate of Requests: Calculate the rate at which a service or human is accessing paths.
Path Enumeration: Track unique paths accessed by services or humans.

Statistical Anomaly Detection:

Use Z-score to identify outliers where access exceeds 10x the baseline.
Implement a sliding window approach to update baselines dynamically based on recent activity.

Machine Learning Models:

Train models using historical data to detect patterns of path enumeration and abnormal request rates.
Use unsupervised learning methods like Isolation Forest or DBSCAN for anomaly detection.

Real-Time Processing: Implement a real-time stream processing framework (e.g., Apache Kafka, AWS Kinesis) to process logs as they come in.

Alerting Mechanism: Define thresholds and generate alerts when anomalies are detected.

False Positive Reduction:

Use probabilistic methods to score the likelihood of an anomaly before alerting.
Implement a cooling-off period for recent alerts to prevent repeated notifications.

#### Closing Summary

By combining statistical methods and machine learning, we can effectively detect unusual activities in Vault audit logs without overwhelming the monitoring team with false positives. This approach ensures that critical security incidents are caught early while maintaining operational efficiency.

#### Common Mistakes/Follow-Ups
False Positives: Ensure the system is finely tuned to reduce alert fatigue.
Baseline Drift: Regularly update baseline models to adapt to changing access patterns.
Resource Usage: Monitor the impact of real-time processing on overall system performance.

**Comparison:**

```
Feature	Description
Access Count	Number of times a service accessed a path in the recent window.
Rate of Requests	Frequency of requests per unit time.
```

**Code:**

```python
# Example code for anomaly detection using Z-score
import pandas as pd
from scipy import stats

def detect_anomalies(data, threshold=3):
    z_scores = stats.zscore(data['access_rate'])
    return (abs(z_scores) > threshold).any()
```

**Example input / output:**

```text
Input:
Audit log data from Vault

Output:
Anomaly detection results and alerts based on Z-score analysis and real-time processing.
```

**Data:**

```json
{
  "service_name": "web_app",
  "path": "/secrets/**",
  "timestamp": "2023-10-01T14:00:00Z",
  "access_count": 50,
  "rate_of_requests": 0.8
}
```

**Flow:**

1. Log Collection
2. Moving Window Aggregation
3. Baseline Calculation
4. Real-Time Processing &Alerting
5. Anomaly Detection (Z-score)
6. Probabilistic Scoring
7. Cooling-Off Period

