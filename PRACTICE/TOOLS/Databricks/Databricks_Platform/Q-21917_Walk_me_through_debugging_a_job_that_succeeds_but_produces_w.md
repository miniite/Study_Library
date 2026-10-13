---
type: "practice"
difficulty: "Deep"
---

# Walk me through debugging a job that succeeds but produces wrong numbers — how do you isolate a logic bug from a data bug in a medallion pipeline?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Delta Live Tables and Serving](<../../../../TOOLS/Databricks/Databricks_Platform/Delta_Live_Tables_and_Serving.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a Databricks job that succeeds but produces incorrect numbers, it's crucial to distinguish between a logic bug and a data bug in your medallion pipeline. This requires a systematic approach to isolate the source of the issue. The key is to methodically compare the expected vs actual results at each step of the pipeline while considering both logical correctness and data consistency.

Firstly, let’s restate what we’re trying to achieve: debugging means identifying whether the problem lies in how your logic processes data or if there are inconsistencies in the input data itself. To do this, we need to carefully analyze the code, validate intermediate states, and ensure that data transformations meet our expectations.

#### Triage

The question is probing into your ability to systematically debug complex pipelines by breaking down the issue into manageable components. We assume you have a Databricks job processing data in a medallion architecture with stages like raw -> clean -> aggregated.

#### First-Principles Solution

Initial Setup and Environment Check:

Verify that your cluster environment is correctly configured.
Confirm that the data version or snapshot being used corresponds to what’s expected during the job run.

Log Analysis and Error Messages:

Review Databricks logs for any error messages or warnings.
Look for anomalies in timestamps, row counts, or specific columns that might indicate an issue.

Data Lineage Tracing:

Use the Databricks Data Lineage feature to trace back from the final output to identify which intermediate steps produced different results than expected.
Check data transformations at each stage of the pipeline: raw -> clean -> aggregated.

Incremental Debugging:

Run incremental checks by processing smaller subsets of data or specific batches to pinpoint where discrepancies start occurring.
Validate logic correctness at critical junctures in your code, such as data cleaning and aggregation functions.

Data Consistency Check:

Compare the input data against known good sources (e.g., test data sets) for any anomalies.
Ensure that all transformations respect schema integrity and data types.

Code Review and Unit Testing:

Conduct a thorough review of your codebase, focusing on logic-heavy sections.
Implement unit tests to validate individual components work as expected independently.

Performance Profiling:

Profile the job’s performance metrics (e.g., memory usage, execution time) to ensure that no resource constraints are causing unexpected behavior.

#### Closing Summary
and Common Mistakes/Follow-Ups

By methodically tracing data lineage, validating intermediate states, and thoroughly reviewing logic and data, you can effectively isolate whether a bug is due to incorrect transformations or flawed input. Remember, the devil is often in the details—pay close attention to edge cases, schema mismatches, and logical errors.

Common pitfalls include jumping straight into code changes without understanding the full context of the issue, overlooking detailed log analysis, and not rigorously validating assumptions about data inputs.

Common Follow-Ups
How do you handle multiple concurrent Databricks jobs processing the same data?
What tools or practices have you used to optimize Databricks job performance for large-scale transformations?

**Comparison:**

```
Stage	Expected Output	Actual Output
Raw	All rows present, valid values	Rows with 'invalid_value' in column_name removed, some expected rows are missing.
Cleaned	Valid columns present, no invalid values	Missing rows with 'invalid_value' in column_name, some expected rows are still missing.
Aggregated	Correct sums grouped by column1	Incorrect sums or no data at all.
```

**Code:**

```python
# Example code snippet for incremental debugging
from pyspark.sql import SparkSession

spark = SparkSession.builder.getOrCreate()

def debug_pipeline(input_data, output_path):
    # Step 1: Read raw data
    raw_df = spark.read.format("csv").option("header", "true").load(input_data)

    # Step 2: Clean and transform the data
    clean_df = raw_df.filter(raw_df.column_name != 'invalid_value').select('column1', 'column2')

    # Step 3: Aggregate cleaned data
    aggregated_df = clean_df.groupBy('column1').agg({'column2': 'sum'})

    # Output the result for verification
    aggregated_df.write.mode("overwrite").parquet(output_path)
```

**Example input / output:**

```text
Input:
- Raw CSV data with some invalid values and missing columns

Expected Output (at each stage):
1. Cleaned Data: Missing rows with 'invalid_value' in column_name removed, all columns present.
2. Aggregated Data: Sum of column2 grouped by column1.

Actual Output:
- Cleaned Data: Some expected rows are missing.
- Aggregated Data: Incorrect sums or no data at all.
```

**Data:**

```json
{
  "raw_data": {
    "format": "csv",
    "path": "/mnt/raw_data"
  },
  "cleaned_data": {
    "path": "/mnt/clean_data"
  },
  "aggregated_data": {
    "path": "/mnt/aggregated_data"
  }
}
```

**Flow:**

1. Read Raw Data
2. Clean and Transform
3. Aggregate Data
4. Write to Parquet
5. Verify Output

