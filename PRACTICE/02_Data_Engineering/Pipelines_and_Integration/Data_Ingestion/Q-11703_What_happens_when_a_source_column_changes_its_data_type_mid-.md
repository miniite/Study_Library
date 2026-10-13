---
type: "practice"
difficulty: "Deep"
---

# What happens when a source column changes its data type mid-stream (e.g., int to string)?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Managed Ingestion and Re-syncs](<../../../../02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Managed_Ingestion_and_Re-syncs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with data ingestion tools like Airbyte or Fivetran, it's crucial to understand how they handle schema changes, especially when a source column's data type changes mid-stream (e.g., from an integer to a string). This situation can lead to unexpected behavior and potential data integrity issues. Let’s break down what happens and explore the full solution.

#### Triage
What the Question Is Really Asking

The question is primarily about understanding how data ingestion tools like Airbyte or Fivetran handle schema changes, specifically when a column's data type changes unexpectedly during the streaming process. The interviewer might be probing your knowledge of:

How these tools manage schema evolution.
Potential issues that could arise from such changes.
Strategies to mitigate these issues.

#### Assumptions
and Context
We are using Airbyte or Fivetran for real-time data ingestion.
A source column, initially an integer (int), has its data type changed to a string during the streaming process.
This change is unexpected and not accounted for in the current schema setup of the ingestion tool.
Step-by-Step Solution

Initial Setup: Assume you are using Airbyte or Fivetran to ingest data from a source system (like a database) into a target system (like a Data Warehouse).

Schema Evolution:

When setting up an Airbyte connection, you define the schema of your sources and targets.
Initially, let's assume the age column is defined as int in both the source and target schemas.

Unexpected Change:

Suppose at some point, due to a backend update or a data migration, the age column starts receiving string values instead of integers.

Airbyte/Fivetran Behavior:

Airbyte/Fivetran will attempt to ingest the new string values into your target database.
If no schema changes are detected and no error handling is in place, this can lead to errors such as invalid data type or even partial ingestion failures.

Impact on Data Integrity:

Inaccurate data: The integer values might get truncated or misinterpreted as strings, leading to incorrect calculations.
Schema Mismatch Errors: If the target schema doesn't support string values for the age column, you may encounter data integrity issues.
Data Loss: Some ingestion tools might fail silently and skip records with invalid data types.

Mitigation Strategies:

Schema Auto-Discovery: Configure Airbyte to automatically detect changes in the source schema.
Type Casting/Conversion: Implement type casting or conversion logic in your data pipeline (e.g., using SQL transformations) to handle unexpected data types.
Error Handling and Logging: Set up robust error handling mechanisms and logging to catch and log such issues for later analysis.

Best Practices:

Regularly review source systems for schema changes.
Use version control in your ingestion pipelines to track schema changes.
Implement a data validation layer to ensure data consistency before storing it in the target system.

#### Closing Summary

Handling unexpected schema changes, such as an integer column turning into a string, requires careful planning and robust error handling. Tools like Airbyte or Fivetran provide mechanisms to detect and manage these changes, but it’s essential to have a comprehensive strategy in place to ensure data integrity and minimize disruptions.

#### Common Mistakes
and Follow-Ups
Mistake 1: Ignoring schema changes during the development phase.
Mistake 2: Failing to implement error handling and logging mechanisms.

#### Follow-Up
Discussing how you would integrate schema change detection with your CI/CD pipeline or leveraging Airbyte’s advanced features for automated schema management.

**Comparison:**

```
Schema Changes	Impact	Mitigation Strategy
Int -> String	Data Inaccuracy	Type Casting/Logging
		Error Handling
```

**Code:**

```sql
-- Example of a type conversion in SQL
SELECT
  CASE
    WHEN column_name::text ~ '^[0-9]+$' THEN CAST(column_name AS INT)
    ELSE NULL -- Handle invalid values by setting them to NULL or another default value
  END as corrected_age
FROM source_table;
```

**Example input / output:**

```text
Input:
- Source Table: age (initially int, then string)
- Target Table: age (expected to be int)

Output:
- Records with valid integer values are correctly ingested.
- Records with invalid string values result in NULL or error handling as per defined logic.
```

**Data:**

```json
{
  "source_schema": {
    "age": "int"
  },
  "target_schema": {
    "age": "int"
  }
}
```

**Flow:**

1. Source System
2. Fivetran/Airbyte
3. Target Warehouse
4. Schema Check & Validation
5. Error Handling
6. Unexpected Data Type
7. Type Conversion Logic
8. Validated and IngestedRecords

