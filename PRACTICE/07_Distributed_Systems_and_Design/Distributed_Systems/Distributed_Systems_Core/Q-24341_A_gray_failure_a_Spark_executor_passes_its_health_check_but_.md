---
type: "practice"
difficulty: "Deep"
---

# A gray failure: a Spark executor passes its health check but silently drops or mis-writes 2% of records. Explain how you'd catch this with row-count reconciliation and differential checks rather than trusting the executor's self-report.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Optimistic Concurrency and Clocks](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Optimistic_Concurrency_and_Clocks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In distributed systems, a common challenge is ensuring data integrity when multiple executors process records concurrently. A gray failure scenario where an executor passes health checks but silently miswrites or drops 2% of records can be particularly insidious. To catch such issues, we need to leverage row-count reconciliation and differential checks. This approach ensures that no data loss or corruption goes unnoticed by verifying the integrity of processed records.

To understand the problem fully, let's break it down step-by-step:

Understanding the Gray Failure: In a distributed system like Apache Spark, an executor may pass health checks due to temporary network issues, resource contention, or other transient faults. However, if these issues lead to silent data loss or corruption, we need robust mechanisms to detect such anomalies.
Row-Count Reconciliation: This involves periodically comparing the row counts of input and output datasets to identify discrepancies. If a significant number of rows are missing or altered, it signals potential issues with the executors' processing logic.
Differential Checks: Differential checks involve calculating differences between two sets of data (e.g., before and after processing) to detect any changes that should not have occurred.
Step-by-Step Solution
1. Define Data Integrity Checks

First, we need to define what constitutes a valid record count and ensure that all records are processed correctly. This involves:

PYTHON
# Example pseudocode for row-count reconciliation
def reconcile_row_counts(input_data_path: str, output_data_path: str) -> int:
    input_df = spark.read.parquet(input_data_path)
    output_df = spark.read.parquet(output_data_path)

    input_count = input_df.count()
    output_count = output_df.count()

    discrepancy = abs(input_count - output_count)

    return discrepancy
2. Implement Differential Checks

Differential checks involve comparing the processed data with a known good state or a previous version of the dataset:

PYTHON
# Example pseudocode for differential checks
def differential_check(current_data: DataFrame, reference_data: DataFrame) -> bool:
    # Calculate differences and check if any changes are unexpected
    difference = current_data.join(reference_data, on="key", how="outer") \
                            .where(col("value").isNotNull() & col("value").notEqual(col("reference_value")))

    return difference.count() > 0
3. Periodic Monitoring and Alerts

To ensure continuous monitoring of data integrity:

PYTHON
# Example pseudocode for periodic monitoring
def monitor_data_integrity():
    input_path = "path/to/input"
    output_path = "path/to/output"
    reference_path = "path/to/reference"

    while True:
        # Reconcile row counts
        discrepancy = reconcile_row_counts(input_path, output_path)
        if discrepancy > 0:
            print(f"Row count discrepancy detected: {discrepancy}")

        # Perform differential checks
        current_df = spark.read.parquet(output_path)
        reference_df = spark.read.parquet(reference_path)
        is_diff_present = differential_check(current_df, reference_df)

        if is_diff_present:
            print("Differential check failed - potential data corruption")

        time.sleep(3600)  # Check every hour

#### Closing Summary
and Common Mistakes

By implementing row-count reconciliation and differential checks, we can effectively detect gray failure scenarios where executors pass health checks but silently miswrite or drop records. This approach ensures that the system remains resilient to transient faults and maintains data integrity.

#### Common mistakes
include:

Failing to regularly reconcile row counts
Not implementing differential checks for nuanced discrepancies
Overlooking the need for periodic monitoring and alerts

These practices can help prevent silent failures from causing significant data loss or corruption in distributed systems.

**Comparison:**

```
Step	Action
1	Reconcile row counts
2	Perform differential checks
3	Implement periodic monitoring
```

**Code:**

```python
# Example pseudocode for row-count reconciliation
def reconcile_row_counts(input_data_path, output_data_path):
    input_df = spark.read.parquet(input_data_path)
    output_df = spark.read.parquet(output_data_path)

    input_count = input_df.count()
    output_count = output_df.count()

    discrepancy = abs(input_count - output_count)

    return discrepancy

# Example pseudocode for differential checks
def differential_check(current_data, reference_data):
    difference = current_data.join(reference_data, on="key", how="outer") \
                            .where(col("value").isNotNull() & col("value").notEqual(col("reference_value")))

    return difference.count() > 0

# Example pseudocode for periodic monitoring
def monitor_data_integrity():
    input_path = "path/to/input"
    output_path = "path/to/output"
    reference_path = "path/to/reference"

    while True:
        discrepancy = reconcile_row_counts(input_path, output_path)
        if discrepancy > 0:
            print(f"Row count discrepancy detected: {discrepancy}")

        current_df = spark.read.parquet(output_path)
        reference_df = spark.read.parquet(reference_path)
        is_diff_present = differential_check(current_df, reference_df)

        if is_diff_present:
            print("Differential check failed - potential data corruption")

        time.sleep(3600)  # Check every hour
```

**Example input / output:**

```text
Discrepancy detected: 1234 (row count)
Differential check failed - potential data corruption
```

**Data:**

```json
{
  "input_path": "path/to/input",
  "output_path": "path/to/output",
  "reference_path": "path/to/reference"
}
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. Start Monitoring
6. Row Count Discrepancy?
7. Differential Check
8. Data Integrity BreachDetected?
9. Audit and Alert
10. Continue Monitoring
11. Alert on Discrepancy

