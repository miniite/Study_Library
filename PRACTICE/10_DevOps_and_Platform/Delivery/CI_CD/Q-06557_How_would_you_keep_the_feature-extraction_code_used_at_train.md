---
type: "practice"
difficulty: "Deep"
---

# How would you keep the feature-extraction code used at training time and at inference time in lockstep through CI so a deploy can't create training-serving skew in your anomaly model?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Performance Regression Gates](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Performance_Regression_Gates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that the feature-extraction code remains consistent between training and inference, we need to establish a robust Continuous Integration (CI) process that keeps both environments aligned. The key idea is to maintain a single source of truth for the feature-extraction logic, ensuring it undergoes the same CI/CD pipeline used in production. This prevents any discrepancies that could lead to training-serving skew, where models perform differently due to differences in data preprocessing.

In this scenario, we are dealing with an anomaly detection model, which relies heavily on consistent and reliable feature extraction. Any inconsistency can significantly impact the accuracy of the model's predictions during both training and inference phases.

#### Triage

The question is essentially asking how to maintain consistency between two critical stages: training (where the model is developed and trained) and inference (where the model makes real-time predictions). The core challenge here is ensuring that any changes or updates made in one environment do not inadvertently introduce discrepancies into the other. This involves setting up a CI/CD pipeline to enforce consistency.

#### Assumptions
Single Codebase: There is a single codebase for both training and inference.
Feature Extraction Functionality: The feature-extraction logic is modular, allowing easy integration into different parts of the application.
Shared Environment: Both environments (training and inference) share the same CI/CD pipeline.

#### First-Principles Solution

Modularize Feature Extraction:

Identify the feature extraction code as a separate module or library that can be easily imported by both training and inference components.

CI Pipeline Setup:

Configure your CI tool (e.g., Jenkins, GitHub Actions) to run automated tests for this feature-extraction module across both environments.
Implement checks such as static analysis, linting, and unit testing to ensure the code is clean and free of errors.

Version Control:

Use version control systems like Git to manage different versions of your codebase.
Tag releases that are deemed stable for training and inference separately but use the same base version.

CI Triggered by Changes:

Ensure the CI pipeline is triggered whenever changes are pushed to the feature-extraction module’s repository.

Automated Deployment:

Deploy any changes through a CI/CD pipeline that ensures both environments (training and inference) get updated simultaneously if needed.

Continuous Monitoring:

Implement real-time monitoring of model performance in production to catch any issues early.
Set up alerting mechanisms for significant deviations between training and inference metrics.

#### Closing Summary

By following a strict CI/CD process, we can maintain the consistency of feature extraction code across both training and inference environments. This ensures that models trained on one environment perform consistently when deployed in production, avoiding any skew due to data preprocessing discrepancies.

#### Common Mistakes/Follow-Ups
Ignoring Changes: Failing to update the inference pipeline with changes made during development.
Manual Deployment: Manually deploying code without automated testing can lead to inconsistencies.
Lack of Monitoring: Not setting up monitoring and alerting for model performance can result in unnoticed issues.

**Code:**

```python
# Example feature extraction module
import numpy as np

def extract_features(data):
    # Feature extraction logic here
    return features
```

**Example input / output:**

```
Input: A sample input dataset data = [1, 2, 3]
Output: Features extracted by the feature-extraction module
PYTHON
Copy
features = extract_features([1, 2, 3])
print(features)  # Example output: [0.5, 1.0, 1.5]
```

**Flow:**

1. CI Triggered
2. Static Analysis
3. Linting
4. Unit Testing
5. Automated Deployment
6. Real-Time Monitoring
7. Alerts for Deviations

