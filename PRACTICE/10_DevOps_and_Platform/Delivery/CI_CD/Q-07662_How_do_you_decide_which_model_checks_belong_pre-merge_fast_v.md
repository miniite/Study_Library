---
type: "practice"
difficulty: "Deep"
---

# How do you decide which model checks belong pre-merge (fast) versus post-merge or pre-prod (slow, expensive training/eval)?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Pull Request Pipelines](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Pull_Request_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding where to place model checks in a CI/CD pipeline, the key is balancing speed and accuracy of validation with cost efficiency. Pre-merge checks are fast but limited in their ability to catch issues that only arise after integration; post-merge or pre-prod checks, while more thorough, can be resource-intensive and delay the merge process.

The goal is to identify critical points where model performance could degrade due to integration issues, ensuring these are caught early. Here's a step-by-step approach from first principles:

Identify Critical Components: Determine which parts of your model or workflow are most susceptible to errors that can only be detected after changes have been merged into the main codebase.
Estimate Cost and Impact: Evaluate the cost (in terms of time, resources) versus the impact on the project if a critical error is missed at each stage.
Risk Assessment: Prioritize checks based on the risk of failure or severity of consequences.

Let's break down these steps in detail:

Identify Critical Components

Training Data Changes: If your model heavily relies on training data that may change with new merges, pre-merge checks are crucial.
Model Architecture Modifications: Any significant changes to the model architecture should ideally be validated before merging to ensure no breaking changes were introduced.
Hyperparameter Tuning: Frequent changes in hyperparameters might require retraining models to maintain accuracy. These could be checked post-merge, but frequent checks can help catch issues sooner.

Estimate Cost and Impact

Pre-Merge Checks:
Training Time: Shorter training times are ideal here since the merge process must proceed quickly.
Evaluation Speed: Fast evaluation tools like unit tests or small-scale validation datasets work well.
Post-Merge/Pre-Prod Checks:
Resource Intensive: These checks often involve full-scale retraining and comprehensive evaluation, which can be time-consuming.
Impact on Schedule: Delays in merging can impact the overall project timeline.

Risk Assessment

High Risk Components: Features that have a high risk of breaking or significantly degrading performance should be prioritized for thorough checks.
Low Risk Components: Lower-risk changes, like minor bug fixes, can often be handled with less rigorous checks.
Triaging the Model Checks

Pre-Merge:

Training Data Validation: Use a small subset of training data to quickly validate that no issues arise from recent merges.
Unit Tests and Assertions: Write tests for critical components of your model that can run in minutes.

Post-Merge/Pre-Prod:

Full Retraining and Evaluation: Deploy the updated model in a staging environment where it is fully retrained and evaluated against comprehensive datasets to ensure no performance degradation.
Performance Metrics: Monitor key metrics like accuracy, loss, etc., to catch any significant issues.

#### Closing Summary

In summary, the decision on where to place model checks should balance the need for thorough validation with the constraints of speed and resource usage. Critical components and high-risk areas require more thorough post-merge or pre-prod checks due to their potential impact, while lower-risk changes can be validated through simpler, faster pre-merge checks.

#### Common Mistakes/Follow-Ups

#### Common mistakes
include:

Overlooking critical components that could fail only after integration.
Underestimating the cost of delaying merges and not catching issues early enough.

#### Follow-ups
might involve discussing specific tools and techniques used for validation at different stages or exploring how to further optimize resource usage without compromising on accuracy.

**Comparison:**

| Stage | Check Type | Frequency | Cost & Time | Risk Impact | |-------------|--------------------|---------------|----------------| | Pre-Merge | Unit Tests, Assertions | Daily/After Each Commit | Fast | Low to Medium | | Post-Merge | Full Retraining and Evaluation | Weekly/Semi-Weekly | Slow, Expensive | High |

**Code:**

```python
# Example of a pre-merge check function
def validate_model_pre_merge(model, train_data_subset):
    # Run quick tests on training data subset
    model.fit(train_data_subset)
    validation_metrics = evaluate_model(model, val_data)
    return validation_metrics

# Example of a post-merge check process
def validate_model_post_merge(model, full_train_data, full_val_data):
    # Full retraining and evaluation in staging environment
    model.fit(full_train_data)
    final_evaluation_results = evaluate_model(model, full_val_data)
    return final_evaluation_results
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Validate model pre-merge with subset of training data | Validation metrics |
| Validate model post-merge in staging environment | Final evaluation results |

**Data:**

```json
{
  "train_data_subset": {
    "features": ["f1", "f2"],
    "labels": [0, 1]
  },
  "full_train_data": {
    "features": [...],
    "labels": [...]
  },
  "full_val_data": {
    "features": [...],
    "labels": [...]
  }
}
```

**Flow:**

1. Pre-Merge Check
2. Validate Model with Subset ofTraining Data
3. Check Validation Metrics
4. Post-Merge Check
5. Deploy Staging Environment
6. Full Retraining and Evaluation
7. Evaluate Final ModelPerformance

