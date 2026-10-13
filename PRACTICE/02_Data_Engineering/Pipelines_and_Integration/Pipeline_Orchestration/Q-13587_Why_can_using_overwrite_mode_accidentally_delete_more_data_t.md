---
type: "practice"
difficulty: "Deep"
---

# Why can using overwrite mode accidentally delete more data than you expect?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ GDPR Deletion in Feature Pipelines](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/GDPR_Deletion_in_Feature_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with data pipelines, especially those involving write operations like "overwrite," it's crucial to understand how they work and their potential pitfalls. Overwrite mode is designed to replace existing data in a destination with new data from a source. However, in practice, this can lead to unintended data loss if certain assumptions are not properly validated or managed. This answer delves into the core issue, explores why it happens, and provides best practices to avoid such issues.

To begin, let's consider an example scenario where we have a dataset that is processed through multiple stages before being written back to its original table using Airflow with Spark in overwrite mode. The problem arises when there are transient or intermittent changes in the data sources or during the processing pipeline itself.

#### Triage
What’s Being Probed

The question aims to understand the nuances of write operations, particularly how overwrite modes work and why they can accidentally delete more data than intended. It also probes your ability to identify edge cases and handle them effectively.

#### Assumptions
and Context
Data Source: The source dataset is dynamic or changes frequently.
Pipeline Complexity: The pipeline involves multiple stages of transformation, filtering, or joining with other datasets.
Execution Environment: Airflow orchestrates the execution, possibly with retries and parallel tasks.
Step-by-Step Solution
1. Understanding Overwrite Mode

Overwrite mode in write operations means that any existing data at the destination is completely replaced by the new data being written. This can be implemented using SQL REPLACE or INSERT ... ON DUPLICATE KEY UPDATE statements, or directly with Spark's DataFrame.write.overwrite() method.

2. Edge Case Scenario

#### Cons
ider a dataset of user transactions that updates frequently due to real-time payments. If we have an Airflow DAG configured to read from the source and write back using overwrite mode:

SQL
INSERT INTO transactions (user_id, transaction_amount)
VALUES (?, ?)
ON DUPLICATE KEY UPDATE transaction_amount = VALUES(transaction_amount);

If there is a minor delay or network issue between the source reading and writing stages, it can cause data to be written out of sequence. For instance:

Task A reads a snapshot of transaction1 at time T0.
Task B reads another snapshot of transaction2 at time T1 > T0.
Task C writes transaction1 back, but due to the delay, it overwrites an outdated version of transaction2.
3. Example Scenario

Suppose we have a dataset with two transactions: user_id = 1, transaction_amount = $50 and user_id = 2, transaction_amount = $75. In a single execution:

Task A reads the source with user_id = 1.
Task B reads the source with user_id = 2.
Both tasks are written back to the destination table using overwrite mode.

If there’s a network delay between reading and writing, it can result in the following sequence of events:

Task A writes $50 for user 1.
Task B writes $75 for user 2, overwriting the previous write operation on $50.
4. Prevention Strategies

To avoid accidental data deletion, consider the following strategies:

Idempotency: Ensure that each write is idempotent so that re-execution does not change the outcome.
Snapshot Isolation: Use read snapshots to ensure that you are always writing based on a consistent state of the source dataset.
Timestamps and Versioning: Maintain timestamps or version numbers in your datasets to detect and handle out-of-order writes.
Atomic Transactions: If possible, use atomic transactions to ensure that all operations within a pipeline stage are treated as a single unit.

#### Closing Summary

In summary, overwrite modes can lead to unintended data deletion when there are delays or intermittent issues between reading and writing stages. Best practices include ensuring idempotency, using snapshot isolation techniques, maintaining timestamps, and leveraging atomic transactions where applicable.

#### Common Mistakes/Follow-Ups
Failing to Consider Transient Issues: Not accounting for temporary delays in network or processing.
Overlooking Data Consistency: Assuming that sequential writes are always guaranteed without proper mechanisms like transactional support.
Not Implementing Idempotency Checks: Failing to ensure that write operations do not cause data loss if re-executed.

**Comparison:**

```
Time	User ID	Transaction Amount
T0	1	50.0
	2	75.0
T1 (Delayed)	2	75.0
```

**Code:**

```python
# Example Spark code snippet for a simple transformation and overwrite write operation
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("DataPipeline").getOrCreate()

# Read from source dataset
source_df = spark.read.csv("path/to/source")

# Perform transformations (e.g., filtering, joining)
transformed_df = source_df.filter(source_df.amount > 100)

# Write back to the destination using overwrite mode
transformed_df.write.mode('overwrite').parquet("path/to/destination")
```

**Example input / output:**

```text
- **Input**:
  - Source dataset snapshot at time T0: `{ user_id = 1, transaction_amount = $50 }` and `{ user_id = 2, transaction_amount = $75 }`.
- **Output**:
  - After execution with a delay in network or processing, the final state might be `{ user_id = 2, transaction_amount = $75 }`, accidentally deleting the `$50` transaction.
```

**Data:**

```json
{
  "user_id": [1, 2],
  "transaction_amount": [50.0, 75.0]
}
```

**Flow:**

1. Yes
2. No
3. Read Source
4. Is Delayed?
5. Write Transaction 2Overwriting 1
6. Write Transaction 1 and 2 inOrder
7. Transaction 1 Lost
8. Data Written Successfully

