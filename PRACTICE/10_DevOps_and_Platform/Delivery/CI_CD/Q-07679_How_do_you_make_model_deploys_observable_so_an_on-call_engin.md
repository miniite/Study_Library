---
type: "practice"
difficulty: "Deep"
---

# How do you make model deploys observable so an on-call engineer can trace a prediction-quality incident back to the exact model version and training run?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Build and Release Metadata](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Build_and_Release_Metadata/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that on-call engineers can trace prediction quality incidents directly back to specific model versions and training runs, we need to build a robust observability framework in our CI/CD pipeline. The core idea is to create detailed metadata around each model version and ensure this metadata is logged and stored alongside predictions for easy reference.

Firstly, let's break down the question: The interviewer is probing how we can implement logging and metadata management effectively within the CI/CD process. This includes identifying what data needs to be captured, where it should be stored, and how it can be queried efficiently. We'll need to consider the scale of our deployments and ensure that logging mechanisms are scalable and don't impact model performance.

#### Triaging the Question

We need to create a system where:

Each deployed model version has unique metadata.
Metadata includes details about the training run, such as hyperparameters, feature importance, and validation metrics.
Predictions include references back to their corresponding metadata.
The logging mechanism is scalable and doesn't impact production performance.

#### First-Principles Solution

Metadata Capture:

During each model training, capture essential metadata like:
JSON
{
  "model_version": "v1",
  "hyperparameters": {"learning_rate": 0.005, "batch_size": 32},
  "training_metrics": {"accuracy": 0.87, "loss": 0.24}
}
Store this metadata in a database or a logging service like Elasticsearch.

Model Versioning:

Use a versioning system like Git for model artifacts and configurations.
Implement a continuous integration (CI) pipeline that automatically tags each commit with relevant metadata.

Prediction Logging:

Each prediction request includes a unique identifier or context ID.
Store the corresponding metadata along with the prediction result in a database or logging service.
JSON
{
  "prediction_id": "p-12345",
  "metadata_reference": "v1",
  "features": {"temperature": 20, "humidity": 70},
  "label": "rainy"
}

Observability Tools:

Use tools like Prometheus and Grafana for real-time monitoring.
Implement logging services that allow querying metadata by prediction ID or model version.

Scalability Considerations:

Ensure the logging system can handle high volumes of requests without performance degradation.
Implement indexing strategies to optimize query performance.

Security and Compliance:

Encrypt sensitive data in storage and logs.
Follow relevant compliance standards like GDPR or HIPAA if applicable.

#### Closing Summary

By implementing a comprehensive CI/CD pipeline that captures metadata during training, ensures model versioning, logs predictions with references back to their metadata, and uses observability tools for real-time monitoring, we can create an environment where on-call engineers can quickly trace prediction-quality incidents back to the exact model version and training run. This solution not only enhances debugging capabilities but also improves overall system reliability and maintainability.

#### Common Mistakes/Follow-Ups
Not capturing enough metadata: Ensure all relevant information is logged, including hyperparameters and validation metrics.
Inefficient logging mechanisms: Optimize logging to avoid performance bottlenecks in production environments.
Lack of security measures: Implement proper encryption and compliance checks to protect sensitive data.
Scalability issues: Design the system to handle large volumes of logs and metadata without degradation.

**Comparison:**

```
Model Version	Hyperparameters	Training Metrics
v1	learning_rate=0.005	accuracy=0.87, loss=0.24
	batch_size=32
```

**Code:**

```python
# Example code for logging prediction with reference metadata
import json

def log_prediction(prediction_id, model_version, features, label):
    metadata_reference = f"v{model_version}"
    log_entry = {
        "prediction_id": prediction_id,
        "metadata_reference": metadata_reference,
        "features": features,
        "label": label
    }
    # Store in a database or logging service
    print(json.dumps(log_entry))
```

**Example input / output:**

```text
Input:
- Prediction request with features: { "temperature": 20, "humidity": 70 }
- Model version: v1

Output:
{
  "prediction_id": "p-12345",
  "metadata_reference": "v1",
  "features": {"temperature": 20, "humidity": 70},
  "label": "rainy"
}
```

**Data:**

```json
{
  "model_version": "v1",
  "hyperparameters": {"learning_rate": 0.005, "batch_size": 32},
  "training_metrics": {"accuracy": 0.87, "loss": 0.24}
}
```

**Flow:**

1. CI Pipeline
2. Training Model
3. Capture Metadata
4. Store in Database
5. Prediction Request
6. Retrieve Metadata
7. Log Prediction with Reference
8. On-Call Engineer TracesIncident

