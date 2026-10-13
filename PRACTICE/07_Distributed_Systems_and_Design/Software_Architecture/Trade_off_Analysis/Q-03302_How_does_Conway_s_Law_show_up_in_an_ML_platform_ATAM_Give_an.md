---
type: "practice"
difficulty: "Deep"
---

# How does Conway's Law show up in an ML platform ATAM? Give an example where a scenario revealed that a feature-pipeline boundary (and its train-serve seam) existed only because of the org chart split between the data-eng and ML teams, and how you documented that as a risk theme.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Turning ATAM Findings into Principles](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Turning_ATAM_Findings_into_Principles.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Conway's Law asserts that "organizations which design systems ... are constrained to reflect the communication structures of those organizations." In an ML platform ATAM, this means the architecture will mirror how teams within the organization are structured and communicate with each other. When conducting a thorough ATAM for our ML platform, we can identify these reflections, particularly at feature-pipeline boundaries where training-serving seams emerge due to organizational constraints.

Let's consider a practical example from one of my projects. The ML platform was being built by two main teams: the Data Engineering (DE) team and the Machine Learning (ML) team, each reporting directly to different executives. Initially, the architecture assumed seamless integration between these two teams. However, as we delved deeper into ATAM, several issues surfaced that highlighted a feature-pipeline boundary and its train-serve seam.

The DE team was responsible for data collection, preprocessing, storage, and ETL processes, while the ML team focused on model development, training, evaluation, and deployment. Initially, these teams communicated and collaborated closely through regular meetings and shared tools. However, as we mapped out the architecture, it became clear that a feature-pipeline boundary existed where the preprocessed data was handed off to the ML models for training.

This boundary manifested in several ways:

Data Schema Mismatch: The DE team used a specific schema for storing raw data, while the ML models required different preprocessing steps and features. This mismatch led to increased complexity in integrating these pipelines.
Feature Selection Discrepancies: The ML team often needed more or less granular features than what was initially provided by the DE team due to evolving model requirements. This resulted in delays and additional workarounds, such as custom transformations.
Training-Serving Mismatch: There were subtle differences in how the models were trained compared to how they served live predictions. These differences required careful documentation and validation processes.

These issues revealed that the feature-pipeline boundary and its train-serve seam existed not just because of technical requirements but also due to the org chart split between the DE and ML teams. We documented this as a risk theme in our ATAM report, noting:

Inter-Team Dependencies: Increased complexity in maintaining and scaling the system due to inter-team dependencies.
Training-Serving Drift Risk: Potential issues with model performance if the training-serving environments diverge further over time.

To mitigate these risks, we proposed several strategies:

Integrated Data Pipeline: Implementing a unified data pipeline that could handle both raw and preprocessed data, reducing the need for manual transformations.
Feature Store Implementation: Creating a centralized feature store where features can be versioned, and their usage tracked across both training and serving environments.
Continuous Integration/Continuous Deployment (CI/CD) Practices: Ensuring that both teams follow consistent CI/CD practices to minimize drift between the development and production environments.

By documenting these findings and proposing mitigations, we were able to align both teams more closely, reducing future integration issues and enhancing the overall robustness of our ML platform.

**Comparison:**

```
Feature	Preprocessing Logic
user_id	No change
session_id	No change
event_type	One-hot encoding
timestamp	Time normalization
Data Source & Schema	Description
Raw Data Storage	Stores raw data from different sources.
Feature Store	Manages versioned preprocessed features for training and serving.
```

**Code:**

```
MERMAID
Copy
graph TD;
    DE[Data Engineering Team] -->|Data Collection & Preprocessing| DataStorage[(Raw Data Storage)];
    DataStorage -->|Feature Transformation| FeatureStore[(Preprocessed Features)];
    FeatureStore -->|ML Model Training| MLModels[(Trained Models)];
    MLModels -->|Serving Predictions| ServingSystem[(Live Prediction Service)];
    DE -- Dependencies -> ML;
```

**Example input / output:**

```

Input:

Raw Data: Collected from various sources (e.g., clickstream data, user logs)
Feature Transformation Logic: Custom Python scripts for transforming raw data into features used by ML models

Output:

Preprocessed Features: Stored in the feature store with versioning enabled
Trained Model: Deployed to serve live predictions with consistent training-serving environments

Example Request-Response Pair:

HTTP
Copy
POST /transform HTTP/1.1
Content-Type: application/json

{
  "data": [
    {"timestamp": "2023-06-01T08:00:00Z", "user_id": "u123", "session_id": "s456", "event_type": "click"},
    {"timestamp": "2023-06-01T09:00:00Z", "user_id": "u789", "session_id": "s012", "event_type": "view"}
  ]
}

HTTP/1.1 200 OK
Content-Type: application/json

{
  "transformed_features": [
    {"timestamp": "2023-06-01T08:00:00Z", "user_engagement_score": 0.7, "session_duration": 30},
    {"timestamp": "2023-06-01T09:00:00Z", "user_engagement_score": 0.5, "session_duration": 45}
  ]
}
```

**Data:**

```json
{
  "data_storage": {
    "raw_data_schema": ["timestamp", "user_id", "session_id", "event_type"],
    "preprocessed_features": ["user_engagement_score", "session_duration"]
  },
  "feature_store": {
    "versioned_features": true,
    "tracking_mechanism": "GitOps"
  }
}
```

**Flow:**

1. Data Collection
2. Preprocessing
3. Model Training
4. Serving Predictions
5. DE
6. Raw Data Collection
7. Feature Transformation
8. (Trained Models)
9. (Live Prediction Service)

