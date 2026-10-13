---
type: "practice"
difficulty: "Expert"
---

# When a regulator asks you to prove exactly which data trained a specific model version deployed 8 months ago, what governance infrastructure must already be in place?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Governing Synthetic Training Data](<../../../../05_Data_Governance_and_Security/Governance/Data_Strategy/Governing_Synthetic_Training_Data.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of proving which data was used to train a specific model version that has been deployed 8 months ago, an advanced ML engineering team needs robust Data Strategy and Governance (DS&G) infrastructure in place. This involves several key components: data lineage, metadata management, data cataloging, and audit trails.

#### Triage
The question is probing into the depth of your organization's DS&G practices to ensure traceability and accountability. The interviewer wants to understand how you manage data throughout its lifecycle, from ingestion to model training, deployment, and beyond. Assumptions include that we are dealing with a production environment where multiple versions of models are deployed over time.

#### First-Principles Solution

Data Lineage: Implement a robust data lineage system that tracks the journey of each dataset through all stages—from raw input to feature engineering, model training, and deployment.
Metadata Management: Maintain detailed metadata about datasets, including provenance (source of data), transformations applied, timestamps, and any quality assessments.
Data Cataloging: Create a comprehensive catalog that documents all relevant datasets, their attributes, and the models they support. This includes documenting which versions of the model were trained on each dataset.
Audit Trails: Ensure there are clear logs and audit trails for all data operations, including model training runs. These logs should capture parameters used during training, timestamps, and any other relevant context.

To illustrate, consider a scenario where we need to prove which version of a customer churn prediction model (v12) was trained on the 5GB CSV file churn_data_2023-07.csv from August 2023. The governance infrastructure would show:

The raw data file churn_data_2023-07.csv was ingested and transformed using a specific pipeline.
This transformation produced features that were then used to train model version v12.
Model training logs (audit trails) would confirm the exact time, parameters, and input data versions.

#### Closing Summary
+ Common Mistakes/Follow-Ups: In summary, a well-implemented DS&G strategy ensures traceability, enabling us to prove with confidence which datasets were used for any given model version. Without such infrastructure, it would be nearly impossible to accurately answer the regulator's question.

#### Common mistakes
include:

Lack of consistent data lineage or incomplete metadata.
Poorly maintained audit trails that do not capture critical information about training runs.
Insufficient documentation in the data catalog, making it hard to correlate datasets with specific model versions.

#### Follow-ups
might involve discussing how different ML frameworks (like TensorFlow, PyTorch) integrate with these governance practices and what additional tools or services are available for enhanced traceability.

**Comparison:**

| Dataset | Feature Engineering Steps | Training Pipeline Version | Model Version | Timestamp |
|---------|---------------------------|--------------------------|---------------|-----------|
| churn_data_2023-07.csv | impute_missing_values, encode_categoricals | v1.0 | v12 | 2023-08-15T14:30:00Z |

**Code:**

```python
# Example of a simple metadata entry in JSON format
metadata = {
    "dataset_name": "churn_data_2023-07.csv",
    "data_source": "Customer Survey Data 2023",
    "feature_engineering_steps": ["impute_missing_values", "encode_categoricals"],
    "training_pipeline_version": "v1.0",
    "model_version": "v12",
    "timestamp": "2023-08-15T14:30:00Z"
}
```

**Example input / output:**

```text
Input:
{
  "dataset_name": "churn_data_2023-07.csv",
  "training_pipeline_version": "v1.0",
  "model_version": "v12",
  "timestamp": "2023-08-15T14:30:00Z"
}

Output:
{
  "dataset_name": "churn_data_2023-07.csv",
  "data_source": "Customer Survey Data 2023",
  "feature_engineering_steps": ["impute_missing_values", "encode_categoricals"],
  "training_pipeline_version": "v1.0",
  "model_version": "v12",
  "timestamp": "2023-08-15T14:30:00Z"
}
```

**Data:**

```json
{
  "dataset": {
    "name": "churn_data_2023-07.csv",
    "source": "Customer Survey Data 2023",
    "features": ["age", "gender", "subscription_length"],
    "transformations": [
      { "step": "impute_missing_values", "parameters": {"strategy": "mean"} },
      { "step": "encode_categoricals", "parameters": {} }
    ]
  },
  "model_training": {
    "pipeline_version": "v1.0",
    "model_version": "v12",
    "timestamp": "2023-08-15T14:30:00Z"
  }
}
```

**Flow:**

1. Data Ingestion
2. Transformation
3. Feature Engineering
4. Model Training
5. Model Deployment
6. Audit Trail
7. Regulatory Proof Request
8. Tracing Data Lineage

