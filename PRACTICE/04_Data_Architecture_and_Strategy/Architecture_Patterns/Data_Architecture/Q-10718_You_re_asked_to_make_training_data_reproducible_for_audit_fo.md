---
type: "practice"
difficulty: "Deep"
---

# You're asked to make training data 'reproducible for audit' for a credit model. What in your lake/warehouse setup do you need to log and immutably keep?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Reproducible Training Data Versions](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Reproducible_Training_Data_Versions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure training data is reproducible for audits, it's crucial to maintain a detailed and immutable record of all inputs used in the model training process. This involves logging not just the final training dataset but also the provenance of each piece of data, metadata related to the data processing steps, and any transformations applied during preprocessing. In a robust data architecture setup, both a data lake and a data warehouse play critical roles.

Firstly, we need to log the raw data sources that feed into our model. This includes identifying the datasets, timestamps, and versioning schemes used. Secondly, detailed metadata about each step in the data pipeline is essential for tracing the lineage of the data throughout its lifecycle. Lastly, any transformations or models applied during preprocessing should be meticulously recorded.

To implement this, we would need to set up both a data lake and a data warehouse with specific configurations. The lake will house the raw data, while the warehouse will process and transform it into more structured formats for analysis.

**Comparison:**

```
Data Source	Type	Version
customer_transactions	csv	v1.2
Processing Step	Parameters
feature_engineer	-
data_cleanse	-
Transformation	Method
impute_missing_values	mean
scale_features	min_max
Model Version	Training Date	Input Features	Output Features
3	2023-10-05T14:00:00Z	customer_id, transaction_amount,	credit_risk_score
		transaction_frequency
Data Log Entry (Input)	Description
source_data	Raw data details
processing_steps	Steps involved in preprocessing
transformations_applied	Detailed steps and parameters applied
```

**Code:**

```
Example of metadata schema for logging data provenance in a data lake

metadata_schema = { "source": {"name": str, "type": str, "timestamp": datetime}, "data_version": int, "processing_steps": [str], # List of steps like 'preprocess', 'feature_engineer' "transformations_applied": [{"step": str, "params": dict}], # Detailed transformations }

CODE
Copy

```python
# Example schema for logging in a data warehouse
warehouse_schema = {
    "model_version": int,
    "training_date": datetime,
    "input_features": [str],  # List of features used
    "output_features": [str],
    "model_performance_metrics": dict,  # Metrics like accuracy, AUC, etc.
}
```

**Example input / output:**

```text
# Example input to output transformation for a data pipeline log entry
input_log_entry = {
    "source_data": {"name": "customer_transactions", "type": "csv", "timestamp": "2023-10-01T00:00:00Z", "version": "v1.2"},
    "processing_steps": ["feature_engineer", "data_cleanse"],
    "transformations_applied": [
        {"step": "impute_missing_values", "params": {"method": "mean"}},
        {"step": "scale_features", "params": {"method": "min_max"}}
    ]
}

# Output log entry after processing
output_log_entry = {
    "source_data": input_log_entry["source_data"],
    "processing_steps": ["feature_engineer", "data_cleanse"],
    "transformations_applied": [
        {"step": "impute_missing_values", "params": {"method": "mean"}},
        {"step": "scale_features", "params": {"method": "min_max"}}
    ],
    "model_version": 3,
    "training_date": "2023-10-05T14:00:00Z",
    "input_features": ["customer_id", "transaction_amount", "transaction_frequency"],
    "output_features": ["credit_risk_score"]
}
```

**Data:**

```json
{
  "source_data": {
    "name": "customer_transactions",
    "type": "csv",
    "timestamp": "2023-10-01T00:00:00Z",
    "version": "v1.2"
  },
  "processing_steps": ["feature_engineer", "data_cleanse"],
  "transformations_applied": [
    {"step": "impute_missing_values", "params": {"method": "mean"}},
    {"step": "scale_features", "params": {"method": "min_max"}}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Data Source
8. Is Data Lake?
9. Log Raw Data
10. Process Raw Data
11. Is Transformation Applied?
12. Log Transformations
13. Log Processed Data
14. Processed Data
15. Is Warehouse Used?
16. Store in Warehouse
17. Directly Use for Training

