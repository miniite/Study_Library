---
type: "practice"
difficulty: "Expert"
---

# How would you wire your CD pipeline to auto-rollback an LLM deploy when live faithfulness or user-thumbs-down rate spikes, and how do you avoid rolling back on normal metric noise?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Rollback Strategy](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Rollback_Strategy/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When setting up a Continuous Delivery (CD) pipeline for deploying Large Language Models (LLMs), auto-rollback is a critical feature to ensure that any issues, such as drops in live faithfulness or user feedback, are addressed promptly without impacting end-users too much. The challenge lies in distinguishing between genuine issues and normal metric fluctuations ("metric noise"). Here’s how you can approach this problem:

#### Triage
Understanding the Problem

The primary goal is to automatically roll back a deploy if there is an unexpected drop in faithfulness or user satisfaction (e.g., negative feedback). However, we must also avoid rolling back due to temporary spikes that are within normal variance. This requires robust anomaly detection and validation mechanisms.

#### Assumptions
Faithfulness Metric: A metric indicating how closely the LLM's output aligns with expected results.
User Feedback: Real-time feedback from end-users, typically as a thumbs-down score or equivalent.
Baseline Metrics: Historical data to establish normal behavior and detect anomalies.

#### First-Principles Solution

Define Anomaly Detection Thresholds:

Establish thresholds for faithfulness and user feedback that trigger an alert if breached. These thresholds should be based on historical data and statistical analysis (e.g., mean + 2 standard deviations).

Real-Time Monitoring:

Set up real-time monitoring of the LLM’s performance using a CD pipeline tool like Jenkins, GitLab CI/CD, or GitHub Actions.

Alerting System:

Implement an alerting system that sends notifications when thresholds are breached. This could be via Slack, email, or other communication tools.

Validation Mechanisms:

Before rolling back, validate the anomaly using additional checks:
Check for temporary spikes in traffic or usage patterns that might explain the drop.
Compare current metrics against a sliding window of historical data to ensure the drop is significant and not just noise.

Rollback Strategy:

If validated as an issue, trigger a rollback to the previous successful deployment version. Ensure this rollback is quick and does minimal disruption to users.

Post-Rollback Monitoring:

Monitor the post-rollback state closely to ensure that metrics stabilize and no further issues arise.

#### Closing Summary

By setting up a robust anomaly detection system, validating anomalies before rolling back, and ensuring a controlled rollback process, we can effectively manage unexpected drops in LLM performance while minimizing disruption. This involves a combination of statistical analysis, real-time monitoring, and validation mechanisms to ensure that only significant issues trigger a rollback.

#### Common Mistakes
and Follow-Ups:
Common Mistake: Rolling back too quickly without validating the issue.

#### Follow-Up
Implement validation checks before initiating a rollback.
Common Mistake: Not establishing clear thresholds for anomaly detection.

#### Follow-Up
Use historical data to define meaningful thresholds that minimize false positives and negatives.
Common Mistake: Failing to monitor post-rollback metrics.

#### Follow-Up
Continuously monitor key performance indicators after a rollback to ensure stability.

**Comparison:**

```
Metric	Threshold (Mean + 2SD)	Current Value
Faithfulness	0.95	0.92
User Feedback	10	12
```

**Code:**

```python
# Example Python code for anomaly detection threshold check
def detect_anomaly(faithfulness, user_feedback, thresholds):
    if faithfulness < thresholds['faithfulness_threshold'] or \
       user_feedback > thresholds['user_feedback_threshold']:
        return True
    return False

# Example configuration dictionary
thresholds = {
    'faithfulness_threshold': 0.95,
    'user_feedback_threshold': 10
}
```

**Example input / output:**

```text
Input Metrics:
{
  "current_metrics": {
    "faithfulness": 0.92,
    "user_feedback": 12,
    "timestamp": "2023-10-05T14:00:00Z"
  },
  "thresholds": {
    "faithfulness_threshold": 0.95,
    "user_feedback_threshold": 10
  }
}

Output Decision:
Rollback triggered due to anomaly in faithfulness and user feedback.
```

**Data:**

```json
{
  "current_metrics": {
    "faithfulness": 0.92,
    "user_feedback": 12,
    "timestamp": "2023-10-05T14:00:00Z"
  },
  "thresholds": {
    "faithfulness_threshold": 0.95,
    "user_feedback_threshold": 10
  }
}
```

**Flow:**

1. Yes
2. No
3. Real-Time Data Collection
4. Is Anomaly Detected?
5. Rollback Validation Checks
6. Rollback to Previous Version
7. Metric Monitoring Continues

