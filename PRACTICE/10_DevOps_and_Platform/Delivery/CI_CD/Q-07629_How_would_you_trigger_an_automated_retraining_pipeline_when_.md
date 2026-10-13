---
type: "practice"
difficulty: "Deep"
---

# How would you trigger an automated retraining pipeline when monitoring detects data drift, and what guardrails stop it from shipping a worse model?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ ML Model Delivery Pipelines](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/ML_Model_Delivery_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Automated retraining pipelines are crucial for maintaining model performance in the face of changing data distributions. The key is to set up a robust monitoring system that detects data drift and triggers retraining, while ensuring that only improved models are deployed. This involves implementing guardrails such as A/B testing and rigorous validation checks.

#### Triaging the Question

The core question here revolves around setting up an automated pipeline for model retraining when monitored data indicates changes in distribution (data drift). The goal is to ensure that the new model performs better than or at least as well as the current one. This requires a deep understanding of CI/CD practices, A/B testing strategies, and rigorous validation processes.

Step-by-Step Solution
1. Monitoring for Data Drift
Concept: Implement a system to continuously monitor incoming data against historical baselines.
Techniques:
Use statistical metrics like Kullback-Leibler Divergence (KL divergence) or Jensen-Shannon Divergence to detect changes in distributions.
Set up thresholds for when drift is considered significant enough to trigger a retraining pipeline.
2. Triggering the Retraining Pipeline
Step 1: Define the monitoring triggers based on statistical significance levels (e.g., p-value threshold).
Step 2: When a drift event is detected, send an alert to the CI/CD system.
Step 3: The CI/CD system then automatically initiates the retraining pipeline.
3. Retraining and Validation
Step 1: Train a new model using the updated data.
Step 2: Perform rigorous validation to ensure the new model performs better or at least as well as the current one:
Cross-validation: Use k-fold cross-validation to assess performance.
A/B Testing: Deploy both models in parallel, monitor their performance on a subset of traffic, and compare results.
4. Guardrails Against Degrading Models
A/B Testing: Ensure that the new model is validated against the existing one before full deployment:
Allocate a small percentage of traffic to the new model.
Monitor key metrics (e.g., accuracy, loss) and revert if performance degrades.
Performance Metrics: Define clear success criteria based on business objectives. For instance, if precision decreases by more than 10%, the retraining should be halted.
5. Deployment
Once validated, deploy the new model to full traffic:
Gradually increase the traffic allocation from the A/B test.
Monitor continuously for any unexpected issues and rollback mechanisms in place.

#### Closing Summary

In summary, setting up a robust CI/CD pipeline that triggers automated retraining based on data drift involves continuous monitoring, rigorous validation, and guardrails against degrading models. This ensures model performance remains optimal over time while minimizing risks associated with deploying potentially worse models.

#### Common Mistakes/Follow-Ups
Mistake 1: Failing to properly monitor for data drift can lead to delayed retraining or unnecessary retraining.
Mistake 2: Insufficient A/B testing can result in poorly performing models being deployed, causing customer dissatisfaction and trust issues.
Mistake 3: Lack of guardrails against degrading models means the system might ship a worse model without proper validation.
Triaging Follow-Ups
How do you implement the monitoring system?
What specific statistical tests would you use for drift detection?
Can you walk through an example scenario where A/B testing fails and how you handle it?

**Comparison:**

```
Feature	Old Distribution	New Distribution
A	40%	50%
B	30%	20%
C	30%	30%
```

**Code:**

```python
# Example of a simple drift detection function using KL divergence
from scipy.stats import entropy

def detect_data_drift(old_data, new_data):
    old_distribution = old_data['feature'].value_counts(normalize=True)
    new_distribution = new_data['feature'].value_counts(normalize=True)

    kl_divergence = entropy(old_distribution, new_distribution)
    if kl_divergence > DRIFT_THRESHOLD:
        return True
    return False

# Example of triggering a retraining pipeline
def trigger_retraining_pipeline():
    # Assume 'data_drift_detected' is a boolean variable indicating drift
    if data_drift_detected:
        print("Retraining pipeline triggered!")
```

**Example input / output:**

```text
Input:
old_data = {"feature": ["A", "B", "C"], "value_counts": [0.4, 0.3, 0.3]}
new_data = {"feature": ["A", "B", "D"], "value_counts": [0.5, 0.2, 0.3]}

Output:
Retraining pipeline triggered!
```

**Data:**

```json
{
  "old_data": {
    "feature": ["A", "B", "C"],
    "value_counts": [0.4, 0.3, 0.3]
  },
  "new_data": {
    "feature": ["A", "B", "D"],
    "value_counts": [0.5, 0.2, 0.3]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Detect Data Drift
6. Is Drift Significant?
7. Trigger Retraining Pipeline
8. Continue Monitoring
9. Perform Model Validation (A/BTesting)
10. Performance Better or Equal?
11. Deploy New Model
12. Revert to Old Model,Investigate Issues

