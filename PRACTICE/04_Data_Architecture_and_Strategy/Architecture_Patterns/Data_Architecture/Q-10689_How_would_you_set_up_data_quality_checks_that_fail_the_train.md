---
type: "practice"
difficulty: "Deep"
---

# How would you set up data quality checks that fail the training job rather than silently training on bad data? Give concrete check types.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Reproducible Training Data Versions](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Reproducible_Training_Data_Versions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure robust and reliable machine learning (ML) models, it is crucial to implement stringent data quality checks that prevent the model from training on bad or incorrect data. This approach ensures that the trained model reflects high-quality features, leading to better performance and reliability. For a Data Engineer at an ML engineering level, setting up these checks involves understanding the data lifecycle and integrating various validation mechanisms.

The core idea is to create a comprehensive data quality architecture that integrates with the end-to-end data pipeline, ensuring that bad data is identified early in the process and stops the training job before any harm can be done. This setup typically involves a combination of static schema checks, dynamic consistency checks, and real-time anomaly detection. Here’s how you would go about it:

#### Triage

The question asks for setting up data quality checks that fail the training job rather than silently allowing bad data to proceed through the pipeline. The key points are:

Static Schema Checks: Ensure data is in the correct format.
Dynamic Consistency Checks: Verify that the data adheres to expected patterns and relationships.
Real-Time Anomaly Detection: Detect unusual or unexpected patterns before training starts.

#### First Principles Solution

Data Ingestion Layer: Implement data validation at the source (e.g., ETL processes, API ingestions).

Data Warehouse/Lake: Store raw and processed data in a structured manner.

Quality Checks:

Schema Validation: Ensure all incoming data matches expected schemas (e.g., using Apache Beam, Apache Flink for real-time checks).

#### Cons
istency Checks: Validate that the data conforms to business rules (e.g., range checks, unique constraints).
Anomaly Detection: Use statistical methods or machine learning models to detect anomalies in the data.

Integration with Training Pipeline:

If any of the above checks fail, halt the training job and alert stakeholders.
Provide detailed logs and error messages for debugging purposes.
Step-by-Step Implementation

Define the Checks:

Schema Validation: Ensure all columns have expected data types and formats.
PYTHON
schema = {
    'column_name': {'type': 'int'},
    'another_column': {'type': 'str'}
}

#### Cons
istency Checks: Verify that certain columns are within expected ranges or meet specific business rules.
PYTHON
def validate_data(row):
    if row['value'] < 0:
        return False
    if not row['category'].isnumeric():
        return False
    return True
Anomaly Detection: Use statistical models to identify outliers in the data.
PYTHON
from sklearn.ensemble import IsolationForest

# Fit an isolation forest model on historical data
model = IsolationForest(contamination=0.05)
model.fit(historical_data)

# Predict anomalies in new data
anomalies = model.predict(new_data) == -1

Integrate Checks with Training Pipeline:

PYTHON
def run_data_quality_checks(data, schema):
    violations = []
    for row in data:
        if not all(row[col] is None or isinstance(row[col], schema[col]['type']) for col in schema):
            violations.append('Invalid column type')
        # Add more checks as needed

    return violations

def preprocess_and_train(data):
    violations = run_data_quality_checks(data, schema)
    if violations:
        print("Data quality issues detected:", violations)
        raise ValueError("Training job failed due to data quality issues")

    processed_data = transform_data(data)  # Example processing step
    model.fit(processed_data)

Alerting and Logging:

Use logging frameworks (e.g., Python’s logging) to log detailed error messages.
Integrate with alerting systems (e.g., Slack, email) to notify stakeholders when checks fail.

#### Closing Summary

In summary, setting up data quality checks that fail the training job involves a multi-layered approach using schema validation, consistency checks, and real-time anomaly detection. These checks should be integrated into the end-to-end data pipeline to ensure only high-quality data is used for model training. This setup not only improves model reliability but also helps in maintaining compliance with business rules.

#### Common Mistakes/Follow-Ups
Mistake: Silently failing or bypassing critical checks can lead to suboptimal models.

#### Follow-Up
Ensure all checks are comprehensive and cannot be easily circumvented.
Mistake: Not integrating logs and alerts properly, leading to delayed issue resolution.

#### Follow-Up
Implement robust logging and alerting mechanisms.

**Comparison:**

```
Violation	Description
Invalid column type	Data in a column does not match the expected data type.
Outlier detected	The data point is an anomaly and may require further investigation.
```

**Code:**

```python
# Example Schema Validation Code
def validate_data(row):
    schema = {
        'age': {'type': int},
        'salary': {'type': float}
    }
    violations = []
    for col, specs in schema.items():
        if not isinstance(row[col], specs['type']):
            violations.append(f'Invalid column type: {col}')
    return violations

# Example Anomaly Detection Code
from sklearn.ensemble import IsolationForest
import numpy as np

historical_data = np.random.rand(100, 3)
model = IsolationForest(contamination=0.05)
model.fit(historical_data)

new_data = np.random.rand(20, 3)
anomalies = model.predict(new_data) == -1
```

**Example input / output:**

```text
data = [
    {"age": 29, "salary": 60000.5},
    {"age": -1, "salary": 70000}
]

violations = validate_data(data[1])
print(violations)  # Output: ['Invalid column type: age']
```

**Data:**

```json
{
    "age": 29,
    "salary": 60000.5,
    "position": "Engineer"
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Data Ingestion
6. Is Schema Valid?
7. Process Data
8. Log Violation
9. Run Consistency Checks
10. Are There Anomalies?
11. Log Anomaly
12. Preprocess and Train Model
13. Alert Stakeholders
14. Alert Stakeholders

