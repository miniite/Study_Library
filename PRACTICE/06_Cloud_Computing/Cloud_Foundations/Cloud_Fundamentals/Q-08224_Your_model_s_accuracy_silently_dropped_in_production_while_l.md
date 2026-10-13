---
type: "practice"
difficulty: "Deep"
---

# Your model's accuracy silently dropped in production while latency looked fine. How would you use cloud monitoring to catch quality regressions, not just system health?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Tail Latency Investigation](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Tail_Latency_Investigation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a sudden drop in model accuracy without an observable increase in latency, leveraging cloud monitoring tools is crucial. This involves setting up comprehensive observability practices that can catch quality regressions as early as possible. The key idea here is to use multiple types of metrics and logs across different layers of your system.

#### Triaging the Question

The question is probing how you would set up a robust monitoring pipeline in the cloud to detect changes in model accuracy, which might not be reflected by traditional latency metrics. This involves understanding the architecture of the system, identifying critical points where such regressions could occur, and setting up appropriate alerts and dashboards.

#### First-Principles Solution
Model Versioning: Implement a versioned model deployment strategy to keep track of different versions and their performance.
A/B Testing: Use A/B testing or canary releases to compare the old and new models in production without full-scale impact.
Batch Jobs: Run periodic batch jobs using historical data to simulate real-world scenarios and compare the outputs against known good results from previous model versions.
Real-time Metrics: Monitor real-time metrics such as prediction accuracy, error rates, and false positives/negatives on a per-prediction basis.
Alerting Systems: Set up alerting systems that notify you when certain thresholds of accuracy are breached.
Detailed Steps

Model Versioning

Ensure each deployment updates the model version in your logging and monitoring system.
Use metadata tagging to identify which models were deployed at what time.

A/B Testing

Set up a split traffic approach where a small portion of the incoming requests go through both old and new models.
Collect metrics on these requests to compare accuracy, latency, and failure rates between versions.

Batch Jobs

Schedule periodic jobs that process historical data with both the old and new models.
Compare the results against expected outcomes or known good data sets.
Use statistical tests (e.g., t-tests) to determine if the differences are significant.

Real-time Metrics

Integrate your model predictions into a real-time monitoring system like Prometheus or Datadog.
Define custom metrics for prediction accuracy, error rates, and other relevant KPIs.
Set up dashboards that display these metrics in real time.

Alerting Systems

Configure alert rules that trigger notifications when the accuracy threshold is breached.
Use a combination of hard-coded thresholds (e.g., 90% vs. 85%) and adaptive thresholds based on historical performance.

#### Closing Summary

By setting up these comprehensive monitoring practices, you can detect quality regressions early in production, ensuring that your system continues to meet the required standards without relying solely on latency metrics.

#### Common Mistakes/Follow-ups
Not Considering Model Versioning: Failing to track model versions could lead to confusion and difficulty in diagnosing issues.
Ignoring Real-time Monitoring: Relying only on historical data for comparisons might miss immediate quality drops.
Overlooking A/B Testing: Not using A/B testing can result in significant production impact before discovering a problematic model.

Would you like to dive deeper into any of these points, or do you have specific cloud providers in mind?

**Comparison:**

```
Model Version	Accuracy
v1	95.0
v2	87.0
```

**Code:**

```python
# Example Python code for setting up a simple A/B test

import random

def predict(input_data):
    # Model prediction logic here
    pass

def main():
    old_model_version = "v1"
    new_model_version = "v2"

    traffic_split = 0.5  # 50% of requests go through the new model

    for i in range(1000):  # Simulate 1000 predictions
        if random.random() < traffic_split:
            prediction, version = predict(input_data=i), new_model_version
        else:
            prediction, version = predict(input_data=i), old_model_version

        # Log or monitor the outcome
        log_prediction(prediction=prediction, model_version=version)

if __name__ == "__main__":
    main()
```

**Example input / output:**

```text
Input:
- `predict(input_data=i)`

Output:
- Logs or metrics indicating the prediction outcome and model version used.
```

**Data:**

```json
{
  "model_versions": {
    "v1": {"accuracy": 95.0},
    "v2": {"accuracy": 87.0}
  },
  "alert_thresholds": {
    "accuracy": 90.0,
    "latency": 100.0
  }
}
```

**Flow:**

1. Model Versioning
2. A/B Testing
3. Batch Jobs
4. Real-time Metrics
5. Alerting Systems
6. Alerting Systems

