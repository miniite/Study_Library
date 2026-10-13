---
type: "practice"
difficulty: "Deep"
---

# An interviewer probe: you said you'd auto-tag PII. What happens when your classifier tags a hashed/tokenized column as PII and blocks a legitimate pipeline? How does your design avoid that failure mode?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cataloging External Data](<../../../../05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Cataloging_External_Data.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In our data catalog, ensuring that personal identifying information (PII) is properly tagged and managed is critical to maintaining compliance and operational efficiency. When we automatically tag columns using a machine learning classifier, one potential failure mode is misclassifying hashed or tokenized PII as actual PII, which could disrupt legitimate pipelines. To address this issue, our design must balance robustness with precision.

To summarize the core solution: we need a multi-layered approach that combines automated tagging with manual verification and contextual awareness to minimize false positives while maintaining operational integrity.

First, let's triage the question:

The interviewer is probing how you handle potential misclassifications in your data catalog.
They are likely concerned about the reliability of your classifier and its ability to handle edge cases like hashed or tokenized PII.
We need to consider assumptions such as the nature of our data, the frequency of hashing/tokenization, and the scale of the system.

Now, let's reason through this from first principles:

1. Automated Tagging with Machine Learning

We use a machine learning model trained on labeled datasets that include various types of PII (e.g., names, addresses, email IDs). This model is designed to identify patterns indicative of PII and flag potentially sensitive columns for further review.

Data Ingestion
Preprocessing
Machine Learning ModelTraining
Model Evaluation & Tuning
Model Deployment
Tagging New Data
2. Contextual Awareness

To reduce the risk of false positives, our system incorporates contextual awareness. This means that we consider not only the content of a column but also its role within the dataset and the broader context of the data pipeline.

For example:

Column Role: If a hashed value is part of a log or audit trail, it might be considered less sensitive.
Data Schema: Understanding the structure of the database helps determine which columns are more likely to contain PII.
Pipeline Metadata: Knowing the intended use and transformation steps of a pipeline can help in deciding whether a column should be tagged as PII.
3. Manual Verification

Even with advanced machine learning, manual verification is essential for high-stakes data management. We implement a workflow where flagged columns are reviewed by domain experts who understand the nuances of the data and can make informed decisions on their sensitivity levels.

Tagging System
Model Prediction
Contextual Evaluation
Manual Review
Final Tagging &Documentation
4. Layered Approach

To further mitigate risks, we use a layered approach:

Primary Tagging: Initial tagging by the machine learning model.
Secondary Filtering: Additional filters that can be manually configured based on known patterns or business rules.
Third-Level Review: A final review stage where experts confirm the tags.
5. Feedback Loop

To continuously improve our classifier, we establish a feedback loop:

Users and domain experts provide feedback on tagged and untagged data.
This feedback is used to retrain and refine the machine learning model over time.
User Feedback
Retraining Model
Improved Tagging Accuracy

#### Closing Summary

Our approach involves a combination of automated tagging, contextual awareness, manual verification, layered filtering, and continuous feedback. This multi-faceted design ensures that we minimize false positives while maintaining the integrity and operational efficiency of our data pipelines.

#### Common Mistakes

Failing to incorporate contextual awareness.
Over-relying on machine learning without human oversight.
Not establishing a robust feedback mechanism for model improvement.

**Comparison:**

| Pipeline Name       | Column   | Original Value  | Initial Tag | Context Check (Sensitive) | Manual Review (Sensitive) | Final Tag |
|---------------------|----------|-----------------|-------------|---------------------------|----------------------------|----------|
| audit_log_pipeline  | hashed_email | ABCD1234@hash.com | PII         | No                        | No                         | Not PII   |
| customer_data_pipeline | address    | 1234 Main St, Anytown, USA | PII       | Yes                       | Yes                        | PII      |

**Code:**

```python
# Example code snippet for initial tagging
def tag_column(column_data):
    # Preprocess data if necessary
    processed_data = preprocess_data(column_data)

    # Predict PII likelihood using machine learning model
    prediction = ml_model.predict(processed_data)

    return prediction

# Example of context-aware filtering function
def is_contextual_sensitivity_high(context, column_value):
    # Check if the column value fits a high-sensitivity pattern based on context
    if context == "log":
        return False  # Logs are typically not considered sensitive

    # Additional checks can be added here

    return True

# Example of manual review function
def manual_review_column(tagged_column, domain_expert):
    # Provide the column data and expert for review
    reviewed_tag = domain_expert.review_column(tagged_column)

    return reviewed_tag
```

**Example input / output:**

| Input | Output |
|-------|--------|
| `hashed_email` tagged as PII in `audit_log_pipeline` context<br>Contextual evaluation: `false`<br>Manually reviewed by expert: `not sensitive` | Tagging updated to non-PII |

**Data:**

```json
{
  "columns": [
    {
      "name": "hashed_email",
      "original_value": "ABCD1234@hash.com",
      "tagged_value": "PII"
    },
    {
      "name": "address",
      "original_value": "1234 Main St, Anytown, USA",
      "tagged_value": "PII"
    }
  ],
  "contextual_data": [
    {
      "pipeline_name": "audit_log_pipeline",
      "column": "hashed_email",
      "is_sensitive": false
    },
    {
      "pipeline_name": "customer_data_pipeline",
      "column": "address",
      "is_sensitive": true
    }
  ]
}
```

**Flow:**

1. Data Ingestion
2. Preprocessing
3. Machine Learning ModelPrediction
4. Contextual Evaluation
5. Manual Review
6. Final Tagging &Documentation

