---
type: "practice"
difficulty: "Intermediate"
---

# Your training pipeline reads Parquet from the lake but a colleague reads the same table via the warehouse and gets different row counts. How would you debug this discrepancy between the lake and the warehouse copy?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ End to End No Data Loss](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/End_to_End_No_Data_Loss.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When faced with discrepancies in data counts between the raw data stored in a data lake and a warehouse, it's essential to methodically investigate potential causes. In this scenario, we need to ensure that the Parquet files read from the lake are identical to those read by your colleague’s process through the warehouse. Let’s break down the issue step-by-step.

Firstly, the key assumptions are:

Both systems (lake and warehouse) should reflect the same dataset.
The processes for reading data should be robust and consistent.

The question is probing into understanding how data inconsistencies can arise in a data pipeline, and what steps you would take to debug such an issue. We need to consider potential sources of discrepancies, including data processing pipelines, storage formats, and even metadata discrepancies.

#### Triage

The core problem here is that the row counts differ between two identical datasets stored differently (lake vs. warehouse). This could be due to several reasons:

Data transformation or cleaning steps performed during the process.
Metadata differences affecting the row count.
Storage format inconsistencies leading to different interpretations of data.
Step-by-Step Debugging from First Principles

Verify Parquet File Integrity:

Ensure that the Parquet files in both the lake and warehouse are identical by comparing file hashes or metadata directly on disk.

Check Data Transformation Steps:

Revisit any ETL (Extract, Transform, Load) jobs used to move data from the lake to the warehouse.
Look for potential transformations that might alter row counts, such as filtering, deduplication, or aggregation steps.

Examine Warehouse Schema Differences:

Compare the schema of the table in both systems to ensure they are exactly identical.
Check if any default values, indexes, or constraints were applied differently during the load process.

Review Data Loading Process:

Inspect how data is loaded into the warehouse from the lake. Are there any batch jobs or continuous ingestion processes?
Ensure that all rows in the Parquet files are being correctly identified and processed by both systems.

Logging and Auditing:

Implement logging to capture detailed information about each step of the ETL process.
Use audit trails to track changes and identify any potential discrepancies early in the pipeline.

Data Sample Verification:

Manually inspect a small sample of data from both systems side-by-side to look for any obvious differences or anomalies.
Use tools like SQL queries to fetch and compare specific records.

Metadata Consistency Check:

Verify that metadata such as timestamps, partitioning keys, and other critical fields are consistent across both environments.

#### Closing Summary

By systematically verifying each step of the data processing pipeline, we can identify where the discrepancy in row counts is occurring. The primary areas to focus on are the integrity of Parquet files, consistency of ETL processes, schema differences, and logging for detailed tracking. Implementing these steps will help ensure that both systems reflect the same dataset accurately.

#### Common Mistakes/Follow-Ups
Overlooking intermediate transformations or cleaning steps.
Assuming data is identical without proper verification.
Not thoroughly comparing metadata across different environments.
Failing to implement comprehensive logging and auditing mechanisms early in the pipeline.

**Comparison:**

```
File Path	Lake Row Count	Warehouse Row Count
/path/to/lake/file.parquet	10000	9876
Schema Differences
- Column A: NULLs treated differently <br> - Default value for column B not consistent
```

**Code:**

```python
# Example script for file integrity check using Python (Pandas)
import pandas as pd

def verify_parquet_integrity(file1, file2):
    # Load Parquet files
    df1 = pd.read_parquet(file1)
    df2 = pd.read_parquet(file2)

    # Verify metadata and row counts
    if df1.shape != df2.shape:
        print("Row count discrepancy: ", df1.shape, " vs. ", df2.shape)
    else:
        print("File integrity check passed.")

# Example usage
verify_parquet_integrity("/path/to/lake/file.parquet", "/path/to/warehouse/file.parquet")
```

**Example input / output:**

```text
Input:
- Parquet files from the lake and warehouse

Output:
- Log of verification results (file integrity, row count, schema comparison)
- Identified discrepancies or metadata differences
```

**Data:**

```json
{
  "lake_file_path": "/path/to/lake/file.parquet",
  "warehouse_file_path": "/path/to/warehouse/file.parquet"
}
```

**Flow:**

1. No
2. Yes
3. No
4. Verify Parquet File Integrity
5. Are Files Identical?
6. Check Data TransformationSteps
7. Review Warehouse SchemaDifferences
8. Examine Data Loading Process
9. Implement Logging andAuditing
10. Verify Metadata ConsistencyCheck
11. Data Sample Verification
12. Debugging Complete?

