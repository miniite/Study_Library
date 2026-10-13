---
type: "practice"
difficulty: "Intermediate"
---

# Explain what lineage is and give a concrete example where lineage saved you when a downstream model started misbehaving.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distinguishing Drift from Pipeline Breaks](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Distinguishing_Drift_from_Pipeline_Breaks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Lineage in data architecture refers to tracking the provenance of data through its transformations, from source to final output. This is crucial for maintaining trust and understanding how different parts of your system interact with each other. For example, when a model started misbehaving, we were able to trace back issues to an earlier step using lineage information.

Let's say you have a machine learning pipeline where data goes through multiple stages: ingestion from various sources, preprocessing, feature engineering, and finally training the model. Lineage helps track which source file contributed to each transformed dataset, what transformations were applied, and by whom and when. This traceability can be invaluable in debugging and maintaining complex models.

#### Triage

The question is probing your understanding of data lineage concepts and how they can help solve issues in a real-world scenario. The interviewer is likely looking for assumptions about the importance of robust data tracking and your ability to articulate how it works in practice.

#### First-Principles Solution

Assume we have a ML pipeline with multiple stages: ingestion from databases and APIs, preprocessing (data cleaning, feature extraction), and model training. Suppose a model's performance suddenly degraded without any changes to the codebase or data inputs.

Identify the Issue: The first step is to identify that there has been an issue in the model’s performance.
Check Logs and Metrics: Review logs for recent changes or anomalies. Check if new datasets were loaded, transformations applied, or models retrained.
Lineage Analysis: Use lineage tools to trace back each dataset and transformation involved in the current pipeline run. Specifically, check:
Which raw data sources were used?
What preprocessing steps were executed on these sources?
Were there any anomalies or unexpected behaviors noted during these transformations?

In our case, the model performance drop was traced back to a specific feature extraction step where data from an external API had been altered due to a recent change in API schema without updating our transformation logic. By understanding this lineage, we could revert the change and retrain the model with the correct features.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Lineage is essential for debugging and maintaining trust in your models. Without it, identifying such issues can become very difficult. In my experience, robust data lineage helped us quickly diagnose a problem that would have taken much longer to resolve otherwise.

#### Common mistakes
include neglecting to implement or maintain proper lineage tracking systems, which can lead to wasted time when issues arise. Follow-ups might include discussing how to set up a lineage system and best practices for maintaining it over time.

**Comparison:**

```
Transformation Step	Action	Description
1	clean_null_values	Removes null or NaN values from the dataset.
2	normalize_data	Scales numerical features to a standard range (e.g., [0, 1]).

This ensures each step of data transformation is recorded for future reference.
```

**Code:**

```python
# Example code snippet for lineage tracking in a data pipeline

def load_data(source):
    # Code to load raw data from source
    return data

def preprocess(data, transformations):
    # Apply preprocessing steps based on defined transformations
    processed_data = apply_transformations(data, transformations)
    return processed_data

def train_model(data):
    # Train the model using the preprocessed data
    model = fit_model(data)
    return model

# Example usage with lineage tracking
source_info = {
    "name": "External API",
    "version": "v2"
}

transformations = [
    {"step": 1, "action": "clean_null_values"},
    {"step": 2, "action": "normalize_data"}
]

raw_data = load_data(source=source_info)
processed_data = preprocess(data=raw_data, transformations=transformations)
model = train_model(processed_data)

# Store lineage information
lineage_info = {
    "source_info": source_info,
    "transformations_applied": transformations,
    "model_performance_metrics": model.evaluate()
}

save_lineage(lineage_info)
```

**Example input / output:**

```text
Input:
- Raw data from External API v2
- Predefined transformations

Output:
- Processed Data with cleaned and normalized values
- Trained Model with performance metrics
```

**Data:**

```json
{
  "source_info": {
    "name": "External API",
    "version": "v2"
  },
  "transformations_applied": [
    {"step": 1, "action": "clean_null_values"},
    {"step": 2, "action": "normalize_data"}
  ],
  "model_performance_metrics": {
    "accuracy": 0.85,
    "loss": 0.34
  }
}
```

**Flow:**

1. v2
2. Load Raw Data
3. Check Source Version
4. Transformations Applied
5. Process Data
6. Train Model
7. Evaluate Model Performance
8. Save Lineage Info

