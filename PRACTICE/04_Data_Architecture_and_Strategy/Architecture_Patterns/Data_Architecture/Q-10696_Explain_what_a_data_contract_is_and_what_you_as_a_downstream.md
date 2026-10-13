---
type: "practice"
difficulty: "Intermediate"
---

# Explain what a data contract is and what you, as a downstream ML consumer, would insist it guarantees about a feature source.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Central vs Domain Feature Modeling](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Central_vs_Domain_Feature_Modeling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A data contract in the context of data architecture refers to a set of defined rules and agreements that dictate how data is structured, stored, and accessed by downstream consumers like ML models. As an ML engineer with intermediate experience, understanding what your feature source guarantees via a robust data contract is critical for ensuring model reliability and maintainability.

#### Triage

The question asks you to explain the concept of a data contract and specify what guarantees you would insist on as a downstream ML consumer from a feature source. The interviewer is probing your knowledge of data integrity, consistency, and interoperability, especially in complex data pipelines involving data lakes and warehouses. You will need to consider aspects such as schema validation, metadata, data freshness, and the semantic meaning of features.

#### First-Principles Solution
Schema Definition: Ensure that the feature source clearly defines its schema, including data types, formats, and relationships between fields.
Data Quality: Verify that the data is clean and adheres to a consistent quality standard, free from anomalies or inconsistencies.
Freshness: Ensure that the data is up-to-date and reflects recent events or changes.
Semantic Clarity: The feature source should provide clear documentation of what each field represents in terms of its business context and meaning.
Versioning: Implement a version control system for schema changes to avoid breaking downstream consumers.
Detailed Explanation

Schema Definition: A well-defined schema is crucial as it ensures that the data can be easily understood and processed by ML models. For instance, if you have numerical features like age, they should consistently use integer or float types.

PLAINTEXT
# Example Schema: customer_features.csv
age:int, gender:category, income:float, ...

Data Quality: Ensure that the data is clean and does not contain missing values, outliers, or inconsistencies. For example, age should be logically within a reasonable range (0-150).

PLAINTEXT
# Example Data Cleanliness Check
if age < 0 or age > 150:
    raise ValueError("Invalid age value")

Freshness: The data should be refreshed regularly to reflect recent events. This might involve setting up real-time ingestion pipelines for streaming data or batch processing schedules.

PLAINTEXT
# Example Data Freshness Check
import datetime

if (datetime.now() - last_update_time).total_seconds() > freshness_threshold:
    raise ValueError("Data is stale")

Semantic Clarity: Documentation should explain the business logic behind each field. For example, income might be derived from several financial transactions.

PLAINTEXT
# Example Feature Documentation
income: The total annual income calculated based on transaction data over the past year.

Versioning: As schemas evolve, maintaining version control ensures that changes do not disrupt existing consumers. Version numbers can be embedded in feature source metadata.

PLAINTEXT
# Example Schema Version Control
import pandas as pd

schema_version = "v2"

def validate_schema(df):
    if df.schema != f"schema={schema_version}":
        raise ValueError("Schema mismatch")

#### Closing Summary
and Common Mistakes/Follow-Ups

A robust data contract ensures that the feature source is reliable, consistent, and well-documented. As an ML engineer, you must insist on these guarantees to build trustworthy models. Common mistakes include ignoring schema validation, failing to handle data quality issues, or not providing sufficient semantic clarity.

#### Follow-up
questions might explore how you would implement these checks in practice, such as using ETL tools like Apache Beam or Flink for data transformation and validation pipelines. Another follow-up could be about how to handle versioning during model deployment and production monitoring.

**Comparison:**

| Column | Type    | Description                       |
|--------|---------|-----------------------------------|
| age    | int     | Customer's age                    |
| gender | category| Male or Female                     |
| income | float   | Annual customer income            |

**Code:**

```python
# Example Schema Definition Check
def validate_schema(df):
    expected_schema = "age:int, gender:category, income:float"
    actual_schema = df.dtypes.to_string()
    if expected_schema != actual_schema:
        raise ValueError("Schema mismatch")

# Example Data Quality Check
def check_data_quality(df):
    for column in ["age", "income"]:
        if (df[column] < 0).any():
            raise ValueError(f"Negative value found in {column}")
```

**Example input / output:**

```text
# Example Input Data
age, gender, income
25, male, 50000.0

# Example Output Check (assuming data quality check fails)
ValueError: Negative value found in age
```

**Data:**

```json
{
  "customer_features": {
    "columns": [
      {"name": "age", "type": "int"},
      {"name": "gender", "type": "category"},
      {"name": "income", "type": "float"}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Data Contract
6. Validate Schema
7. Proceed with Data Processing
8. Alert Developer/Team
9. Check Data Quality
10. Pass?

