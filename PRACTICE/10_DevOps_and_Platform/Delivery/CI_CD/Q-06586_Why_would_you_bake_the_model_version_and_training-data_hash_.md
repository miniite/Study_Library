---
type: "practice"
difficulty: "Deep"
---

# Why would you bake the model version and training-data hash into your detector's telemetry, and how does your pipeline use it to trace a detection regression back to a deploy?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Build and Release Metadata](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Build_and_Release_Metadata/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Baking the model version and training data hash into the detector’s telemetry serves as a critical mechanism for tracing and debugging any detection regressions that occur post-deployment. This technique allows us to pinpoint exactly which version of the model was used during inference, along with the specific dataset it was trained on. By embedding this metadata in real-time telemetry, we can correlate issues in live deployments back to their root causes in development or training.

#### Triaging the Question

The question is probing our understanding of CI/CD best practices and how they are applied in monitoring and troubleshooting machine learning models deployed in production. Key aspects include:

Model Versioning: Ensuring that we can track which model version was used during a specific inference.
Data Consistency: Verifying that the data used for training aligns with what's being fed into the model during deployment.
Step-by-Step Solution from First Principles
1. Model Version Baking

When deploying an ML model, it’s essential to have a clear lineage of which version was deployed at any given time. By baking this information into the telemetry data, we ensure that:

Traceability: Any anomalies in real-time predictions can be linked back to the specific model version.
Reproducibility: If issues arise, we can quickly refer to the training logs and artifacts from that particular model build.
2. Training Data Hashing

Embedding a hash of the training data ensures that:

Data Consistency: We verify that the dataset used during inference matches exactly what was used for training.
Regeneration Capability: If any discrepancies arise, we can re-generate the exact same dataset and train a model on it to compare with the deployed version.

#### Implementation

To implement this, consider the following steps:

Model Versioning:

During deployment, capture the exact model version (e.g., v2.34).
Embed this version in the telemetry data sent from each inference request.

Training Data Hashing:

Calculate a hash of the training dataset before feeding it into the model for training.
Store this hash alongside the trained model metadata.
Telemetry Example

Suppose we have a detection system that needs to be monitored continuously. Here’s how the telemetry might look:

JSON
{
  "model_version": "v2.34",
  "training_data_hash": "abcd1234ef567890",
  "inference_request_id": "req-123",
  "timestamp": "2023-10-01T12:00:00Z"
}
Tracing a Regression

When a regression is detected:

Fetch Telemetry Data: Retrieve the telemetry data from the time of the inference that led to the anomaly.
Compare Model Version: Ensure the model version in telemetry matches the deployed version.
Verify Training Data: Compare the training data hash with the known hashes from previous deployments or retrain using the same dataset.

#### Closing Summary

Baking model versions and training data hashes into telemetry provides a robust mechanism for tracing and debugging issues in real-time. It ensures that we can quickly identify the exact conditions under which an issue occurred, facilitating faster resolution of production problems.

#### Common Mistakes
and Follow-Ups
Ignoring Model Versioning: Failing to capture model versions can make it difficult to trace issues.
No Data Consistency Checks: Not verifying data hashes can lead to undetected discrepancies in the training dataset.
Lack of Telemetry: Absence of telemetry altogether means no data for post-deployment monitoring.

