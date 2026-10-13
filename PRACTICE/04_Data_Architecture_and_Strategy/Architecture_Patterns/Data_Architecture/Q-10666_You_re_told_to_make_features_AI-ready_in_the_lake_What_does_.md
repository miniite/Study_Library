---
type: "practice"
difficulty: "Intermediate"
---

# You're told to make features 'AI-ready' in the lake. What does that mean practically for how you store, version, and document feature tables?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Reproducible Training Data Versions](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Reproducible_Training_Data_Versions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure features are "AI-ready" in a data lake, we need to focus on three core aspects: storing the data appropriately, maintaining proper versioning, and thorough documentation. Essentially, making features AI-ready means setting up your feature tables with optimal structures that can be seamlessly integrated into machine learning pipelines.

#### Triaging the Question

The key here is understanding what "AI-ready" implies in terms of data management practices. The interviewer is probing on how we handle data lifecycle management—storage, versioning, and documentation—to support robust and scalable AI/ML models. We need to assume a typical scenario where raw data from various sources is ingested into the lake, and features are derived for model training.

#### First Principles Solution

Storing Data Appropriately:

Use a structured format like Parquet or ORC that supports efficient querying.
Store in a distributed file system (e.g., HDFS) to leverage parallel processing capabilities.
Employ a schema-on-read approach, where the schema is defined at query time for flexibility.

Maintaining Versioning:

Implement a version control strategy to track changes in feature definitions and data transformations over time.
Use metadata management systems like Apache Atlas or AWS Glue Catalog to maintain lineage information.

Thorough Documentation:

Document the feature extraction process, including any ETL steps, transformations, and cleaning techniques.
Maintain a changelog that records updates to features, dependencies, and any business rules applied during processing.
Practical Implementation

To illustrate this practically:

Storing Data Appropriately:
Raw Data Ingestion
Process & Transform
Feature Engineering
Data Quality Checks
Parquet/ORC Files
Monitor & Alert
Storage Tier
Maintaining Versioning:
Feature Definition
Version Control
Documentation & Metadata
Maintain Lineage
Change Logs
Thorough Documentation:
MARKDOWN
| Feature Name   | Data Source       | Transformation Pipeline | Description                            |
|----------------|------------------|------------------------|----------------------------------------|
| CustomerAge    | CRM              | Age calculation        | Calculates customer age from DOB        |
| PurchaseAmount | Sales Database   | Clean & Normalize      | Cleans and normalizes purchase amounts  |

#### Closing Summary

In summary, making features "AI-ready" in a data lake involves setting up a robust storage strategy, maintaining version control for feature definitions, and documenting the entire process. This ensures that features can be reliably used for training ML models while providing transparency into their origin and evolution.

#### Common Mistakes/Follow-Ups
Mistake: Overlooking the need for metadata management which leads to difficulty in tracing data lineage.

#### Follow-Up
Discussing how to integrate version control with feature store tools like Databricks Feature Store or MLflow.

**Code:**

```python
# Example ETL Code snippet (simplified)
import pandas as pd

def process_data(input_path, output_path):
    raw_data = pd.read_csv(input_path)
    # Transformations
    transformed_data = raw_data[['customer_id', 'purchase_date']]
    transformed_data['age'] = calculate_age(transformed_data['dob'])
    # Save in Parquet format
    transformed_data.to_parquet(output_path)

def calculate_age(dob):
    today = pd.Timestamp.now()
    return (today.year - dob.dt.year) - ((today.month, today.day) < (dob.dt.month, dob.dt.day))
```

**Example input / output:**

| Feature Name | Data Source       | Transformation Pipeline           | Description                            |
|--------------|------------------|-----------------------------------|----------------------------------------|
| CustomerAge  | CRM              | Age calculation                    | Calculates customer age from DOB        |
| PurchaseAmount | Sales Database   | Clean & Normalize                  | Cleans and normalizes purchase amounts  |

**Data:**

```json
{
  "features": [
    {
      "name": "CustomerAge",
      "data_source": "CRM",
      "transformation_pipeline": ["Age calculation"],
      "description": "Calculates customer age from DOB"
    },
    {
      "name": "PurchaseAmount",
      "data_source": "Sales Database",
      "transformation_pipeline": ["Clean & Normalize"],
      "description": "Cleans and normalizes purchase amounts"
    }
  ]
}
```

