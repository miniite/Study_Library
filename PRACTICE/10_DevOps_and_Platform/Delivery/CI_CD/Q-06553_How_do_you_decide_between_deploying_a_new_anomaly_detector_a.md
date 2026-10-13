---
type: "practice"
difficulty: "Deep"
---

# How do you decide between deploying a new anomaly detector as a canary gated on false-positive rate versus a shadow deploy gated by comparison against the incumbent?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Canary and Progressive Delivery](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Rollback_Strategy/Canary_and_Progressive_Delivery.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding whether to deploy a new anomaly detector in a canary deployment gated by false-positive rate or in a shadow deployment gated by comparison against the incumbent, we need to balance risk tolerance, performance, and the impact on the system. The key is to understand the trade-offs involved in each approach.

#### Triage

The question is asking us to compare two different methods of deploying an updated anomaly detector:

Canary Deployment (False-Positive Rate Gating): This method allows deployment of a new version while closely monitoring its performance to ensure it does not introduce unacceptable levels of false positives.
Shadow Deployment (Incumbent Comparison Gating): This method involves running the new and old versions in parallel, comparing their outputs, and ensuring they produce consistent results before fully switching over.

#### Assumptions
The incumbent anomaly detector is currently in production and performing adequately.
We have a version of the new detector that is ready for deployment.
There are performance metrics available to measure both detectors.

#### First Principles Solution

Risk Assessment:

False Positive Rate: Canaries are particularly useful when reducing false positives is critical, as they allow us to monitor and limit the impact before full rollout.
Accuracy Comparison: Shadow deployments ensure that the new version performs equivalently or better than the incumbent. This method reduces risk by verifying performance metrics before making a switch.

Performance Metrics:

False Positive Rate (FPR) of the new detector.
Accuracy, Precision, Recall, and F1 Score for both detectors.
Response Time and Latency.

Deployment Strategy:

For canary deployment, we might start with a small percentage of traffic to monitor false positives.
In shadow deployment, we would run the new and old versions in parallel, comparing metrics over time.
Detailed Solution
Canary Deployment (False-Positive Rate Gating)
Initial Setup:
Deploy the new anomaly detector as part of a canary release with 5% traffic.
Monitoring:
Continuously monitor the false positive rate and any other relevant metrics (e.g., response time).
Thresholds:
Define acceptable thresholds for false positives. If the FPR exceeds this threshold, roll back to the incumbent immediately.
Scaling Up:
Gradually increase traffic until full deployment if the new detector meets all criteria.
Shadow Deployment (Incumbent Comparison Gating)
Initial Setup:
Deploy both detectors in parallel on a subset of the production environment.
Monitoring:
Compare metrics such as false positive rate, accuracy, and response time for both versions.
Thresholds:
Define thresholds based on acceptable performance differences. If the new detector’s metrics fall below these thresholds, it can be promoted to full deployment.
Rollout Strategy:
Gradually increase traffic until the new version fully replaces the old one if all comparisons are favorable.

#### Closing Summary

Both approaches have their merits:

Canary Deployment is useful when reducing false positives is a high priority and risk needs to be minimized quickly.
Shadow Deployment ensures comprehensive performance validation before full deployment, reducing the risk of significant operational disruptions.

#### Common Mistakes/Follow-Ups
Ignoring Performance Metrics: Ensuring that all relevant metrics are monitored during both canary and shadow deployments.
Insufficient Monitoring Periods: Not allowing enough time for thorough comparison or monitoring of false positives.
Rushed Rollout Decisions: Making a decision to fully deploy without sufficient data from the monitoring phases.

**Comparison:**

```
Metric	Canary Metrics	Incumbent Metrics
False Positives	0.03	0.04
Response Time	50ms	60ms
```

**Code:**

```python
def monitor_canary(false_positives, threshold):
    if false_positives > threshold:
        return "Rollback"
    else:
        return "Continue Deployment"

def compare_versions(new_metrics, incumbent_metrics, thresholds):
    for metric in thresholds.keys():
        if new_metrics[metric] < thresholds[metric]:
            return "Reject New Version"
    return "Promote to Full Production"
```

**Example input / output:**

```text
Input:
- Canary metrics for the new anomaly detector
- Incumbent metrics from production
- Threshold values

Output:
- Decision to continue or rollback canary deployment (False Positives)
- Decision to promote or reject new version in shadow deployment based on performance comparison
```

**Data:**

```json
{
  "canary_metrics": {
    "false_positives": 0.03,
    "response_time": 50ms
  },
  "incumbent_metrics": {
    "false_positives": 0.04,
    "response_time": 60ms
  },
  "thresholds": {
    "false_positives": 0.1,
    "response_time": 70ms
  }
}
```

**Flow:**

1. No
2. Yes
3. Yes
4. No
5. Initial Canary Setup
6. Monitor False Positives
7. False Positives > Threshold?
8. Continue Deployment
9. Rollback and Fix Issues
10. Shadow Deploy New Version
11. Compare Metrics withIncumbent
12. New Version meetsthresholds?
13. Promote to Full Production
14. Reject New Version, KeepIncumbent

