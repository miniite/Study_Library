---
type: "practice"
difficulty: "Deep"
---

# How would you architect an ML pipeline so a bad feature deployment can be rolled back independently of the model version, and vice versa?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Securing ML Delivery](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Securing_ML_Delivery.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect an ML pipeline where feature deployments and model versions can be managed independently, we need to ensure that changes in features or models are tracked meticulously and can be rolled back without affecting each other. This involves creating a robust system for versioning both features and models, along with a reliable rollback mechanism.

#### Triaging the Question

The core of this question is about designing an ML pipeline that allows independent deployment and rollback of features and model versions. The interviewer is probing your understanding of cloud fundamentals, version control strategies, and rollback mechanisms in complex systems. Key assumptions include:

We need to support multiple feature sets and model versions.
Feature changes should not affect the model version being used by a deployed system, and vice versa.

#### First Principles Solution
Versioning: Implement a robust versioning strategy for both features and models.
Feature Store: Use a centralized feature store that supports feature tagging and versioning.
Pipeline Management: Design an automated pipeline management system to handle deployments independently.
Rollback Mechanism: Ensure there is a reliable way to roll back changes without disrupting other parts of the system.
Detailed Solution

Versioned Feature Store:

Use a feature store that supports versioning, such as AWS SageMaker Feature Store or Google Vertex AI Feature Store.
Each feature set can be tagged with a unique version identifier (e.g., v1, v2).
This allows you to switch between different versions of features seamlessly.

Model Versioning:

Use a model registry that supports multiple versions, such as AWS SageMaker Model Registry or Google Vertex AI Model Registry.
Each model version can be tagged with a unique identifier (e.g., v1, v2).
This ensures that you can easily roll back to previous models without affecting the feature store.

Pipeline Management:

Use CI/CD tools like Jenkins, GitLab CI, or GitHub Actions to manage pipeline deployments.
Implement a deployment strategy where features and models are deployed independently via separate pipelines.
For example, you can have a "feature deployment" pipeline that deploys new feature versions and a "model deployment" pipeline that updates the model version.

Rollback Mechanism:

Establish clear rollback procedures in your CI/CD pipeline.
Use automated tests (unit, integration, end-to-end) to ensure that both features and models can be rolled back safely.
Implement monitoring and alerting mechanisms to quickly detect issues during rollbacks.

#### Closing Summary

By implementing a versioned feature store and model registry, along with independent deployment pipelines and robust rollback procedures, you can ensure that changes in features or models are managed independently. This approach not only improves system reliability but also simplifies maintenance and debugging.

#### Common Mistakes/Follow-Ups
Common Mistake: Not understanding the importance of version control and its implications on downstream systems.

#### Follow-Up
Question: How would you integrate this with your existing ML pipeline? What are some potential challenges and how would you address them?

**Comparison:**

```
Feature Version	Model Version
v1	-
-	v2
Feature Version	Model Version	Deployment Status
v0	v1	Rolled back
```

**Code:**

```python
# Example configuration for AWS SageMaker Feature Store
feature_store = boto3.client('sagemaker-featurestore-runtime')
response = feature_store.create_feature_group(
    FeatureGroupName='customer_features',
    Features=[
        {'Name': 'age', 'FeatureType': 'Integer'},
        {'Name': 'income', 'FeatureType': 'Double'}
    ],
    EventTimeFeatureName='event_time'
)
```

**Example input / output:**

```text
Input:
- Deploy feature version v1 to production.
- Train and deploy model version v2.

Output:
- Production system uses features from v1.
- Model version v2 is active in the model registry, but not used by the production system yet.

Dry-run:
- Rollback feature version: Switch back to v0.
- Rollback model version: Switch back to v1.
```

**Data:**

```json
{
  "feature_group": {
    "name": "customer_features",
    "features": [
      {"name": "age", "type": "integer"},
      {"name": "income", "type": "double"}
    ],
    "event_time_feature_name": "event_time"
  },
  "model_version": {
    "version_number": 2,
    "model_name": "fraud_detection_model",
    "create_date": "2023-10-01"
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Feature Deployment
6. Is Feature Versioned?
7. Tag Feature with New Version
8. Use Existing Feature Store
9. Deploy to Production
10. Use Existing Feature Store
11. Model Deployment
12. Is Model Versioned?
13. Tag Model with New Version
14. Use Existing Model Registry
15. Deploy to Production
16. Use Existing Model Registry

