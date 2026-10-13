---
type: "practice"
difficulty: "Deep"
---

# You're asked to consume a 'customer' data product owned by another team for your churn model. What do you check about its data contract before you build on it?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distinguishing Drift from Pipeline Breaks](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Distinguishing_Drift_from_Pipeline_Breaks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that the customer data from another team can be effectively consumed for our churn model, we need to thoroughly validate and understand the data's structure, quality, consistency, and provenance. This involves several key steps:

Understand the Data Contract: The first step is to clearly define what the "data contract" entails. This includes understanding the schema of the data (e.g., column names, types), any constraints or rules governing the data, and how it should be interpreted.

Data Quality Check: We need to ensure that the data meets basic quality criteria such as completeness, accuracy, and consistency.

Provenance and Metadata: Understanding where the data comes from and its lineage is crucial for trust and traceability. This includes knowing the source systems, any transformations applied, and how the data was collected.

Now let's break down these steps in more detail:

#### Triage
What We're Probing

The interviewer is probing your ability to understand complex dependencies between data products, ensure high-quality data, and manage data lineage for robust machine learning models. This also tests your attention to detail and your ability to work collaboratively across teams.

#### First-Principles Solution
Step 1: Understand the Data Contract
Schema Validation: Check if the schema (column names, types) matches our expectations.

#### Cons
traints and Rules: Ensure any defined constraints or rules are followed. For example, if customer IDs should be unique, verify that this is enforced.
Understand Data Contract
Schema Validation

#### Cons
traints and Rules
Step 2: Data Quality Check
Completeness: Verify that all required fields are present.
Accuracy: Ensure data entries make sense (e.g., valid dates, no negative values for quantities).

#### Cons
istency: Cross-check the data with known historical records or other trusted sources.
B
Completeness Check
Accuracy Check

#### Cons
istency Check
Step 3: Provenance and Metadata
Source Identification: Determine which systems the data originates from.
Transformation Details: Understand any transformations or cleaning steps applied to the raw data.
Documentation and Versioning: Ensure there is proper documentation of changes, and that metadata (e.g., date of collection) is accurately recorded.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, we need to ensure the customer data is well-defined, reliable, and traceable before integrating it into our churn model. This involves validating the schema, checking for quality issues, and understanding the provenance of the data.

#### Common mistakes
include:

Overlooking basic schema validation.
Ignoring data quality checks, leading to poor model performance.
Failing to document or track changes in the data pipeline.

#### Follow-ups
might involve discussing how to automate these checks, handling data inconsistencies, and ensuring ongoing monitoring of the data source.

**Comparison:**

```
Schema Validation	Result
customer_id	Passed
purchase_history	Failed (Missing value)
last_purchase_date	Passed
```

**Code:**

```python
# Example schema validation function
def validate_schema(data):
    required_columns = ['customer_id', 'purchase_history', 'last_purchase_date']
    missing_cols = [col for col in required_columns if col not in data.columns]
    if missing_cols:
        return f"Missing columns: {', '.join(missing_cols)}"
    else:
        return "Schema validation passed"

# Example quality check function
def check_data_quality(data):
    issues = []
    # Check completeness
    if data['purchase_history'].isna().sum() > 0:
        issues.append("Incomplete purchase history")
    # Check accuracy (example)
    if (data['last_purchase_date'] < pd.to_datetime('2021-01-01')).any():
        issues.append("Invalid last purchase date")

    return "Data quality check passed" if not issues else f"Found {len(issues)} issues: {', '.join(issues)}"
```

**Example input / output:**

```text
Input:
{
  "data": {
    "customer_id": [1, 2, 3],
    "purchase_history": [50, null, 75],
    "last_purchase_date": ["2021-06-01", "2021-09-15", "2022-08-31"]
  }
}

Output:
{
  "schema_validation_result": "Schema validation passed",
  "data_quality_check_result": "Incomplete purchase history"
}
```

**Data:**

```json
{
  "data": {
    "customer_id": [1, 2, 3],
    "purchase_history": [50, null, 75],
    "last_purchase_date": ["2021-06-01", "2021-09-15", "2022-08-31"]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Is schema valid?
7. Continue to Data QualityCheck
8. Fix schema issues and re-check
9. Are there data quality issues?
10. Resolve issues, log formonitoring
11. Data ready for modelintegration

