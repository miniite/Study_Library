---
type: "practice"
difficulty: "Deep"
---

# How would you embed data-quality checks (null rates, referential integrity, freshness) into a Spark pipeline so bad batches quarantine instead of publish?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ End to End Exactly Once](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/End_to_End_Exactly_Once.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To embed robust data quality checks in a Spark pipeline and ensure that only high-quality data is published, we need to design a system where null rates, referential integrity, and freshness are validated at each stage of processing. When any check fails, the problematic batch should be quarantined rather than published. This involves multiple steps: defining the quality checks, integrating them into the Spark pipeline, handling failures gracefully, and managing quarantined data.

#### Triage

The question is asking how to embed comprehensive data quality validation mechanisms within a Spark pipeline to ensure that only valid batches are processed further or published. The interviewer is probing your understanding of:

Data Quality Checks: Null rates, referential integrity, and freshness.
Pipeline Design: How these checks can be integrated without impacting performance.
Error Handling: Mechanisms for handling failed validations gracefully.
Batch Processing: Quarantining bad batches rather than publishing them.

#### Assumptions
We are dealing with a large-scale data pipeline using Apache Spark.
The data is stored in Kafka, and the processing logic is implemented in a Spark application.
We need to monitor null rates, ensure referential integrity between tables, and check for freshness (i.e., ensuring that the data is not too old).
If any of these checks fail, the problematic batch should be quarantined.
Step-by-Step Solution
1. Define Data Quality Checks

We will define three primary quality checks:

Null Rate Check: Ensure no column exceeds a certain null rate threshold.
Referential Integrity Check: Ensure that foreign keys in one table match entries in the referenced table.
Freshness Check: Verify that the data is not older than a specified maximum age.
2. Integrate Checks into Spark Pipeline

We will integrate these checks within the Spark pipeline using custom Spark transformations and actions.

PYTHON
from pyspark.sql import SparkSession, functions as F

# Initialize Spark session
spark = SparkSession.builder.appName("Data Quality Check").getOrCreate()

# Define a function to calculate null rate
def null_rate(column):
    return (F.count(F.when(F.col(column).isNull(), 1)).cast("float") / F.count("*")).alias(f"null_rate_{column}")

# Example schema for demonstration purposes
schema = ["id", "name", "email"]

# Read data from Kafka topic
df = spark.readStream.format("kafka").option("topic", "raw_data_topic").load()

# Apply null rate check
null_checks = df.select(*[null_rate(c) for c in schema])

# Apply referential integrity check (assuming 'id' is a foreign key)
ref_integrity_check = df.join(other=reference_table, on="id", how="left_anti")

# Apply freshness check
freshness_check = df.filter(F.col("timestamp") >= F.current_timestamp() - F.expr("interval 1 day"))

# Combine all checks
quality_checks = null_checks.join(ref_integrity_check, on="id", how="inner").join(freshness_check, on="id", how="inner")

# Quarantine bad batches (save to a different topic or mark them)
quarantine_df = quality_checks.filter((F.col("null_rate_id") > 0.1) | F.col("ref_integrity_check").isNull() | (F.col("timestamp") < F.current_timestamp() - F.expr("interval 1 day")))

# Publish good batches
good_batches = quality_checks.filter((F.col("null_rate_id") <= 0.1) & F.col("ref_integrity_check").isNotNull() & (F.col("timestamp") >= F.current_timestamp() - F.expr("interval 1 day")))
3. Handling Failures Gracefully

We need to ensure that the Spark application can handle failures gracefully and that bad batches are quarantined. This involves implementing a robust error handling mechanism.

PYTHON
# Save good batches to another topic or marked as processed
good_batches.writeStream.format("kafka").option("topic", "clean_data_topic").start()

# Save bad batches to quarantine topic for further inspection or remediation
quarantine_df.writeStream.format("kafka").option("topic", "quarantine_topic").start()
4. Monitoring and Maintenance

To ensure the quality checks are effective, we need to monitor their performance and maintain them over time.

#### Closing Summary

By embedding comprehensive data quality checks within our Spark pipeline, we can ensure that only high-quality batches are processed further or published. This involves defining null rate, referential integrity, and freshness checks, integrating these into the Spark pipeline, handling failures gracefully, and managing quarantined data. Effective monitoring is crucial to maintaining system health and ensuring robust data processing.

#### Common Mistakes/Follow-Ups
Ignoring Null Rates: Failing to account for high null rates can lead to incomplete or inaccurate data.
Failing Referential Integrity Checks: Ensuring foreign keys match referenced records is critical to data consistency.
Inadequate Freshness Check: Not checking the age of data can result in using outdated information, leading to poor decision-making.

For further discussion:

How do you optimize these checks for performance?
What measures would you take to ensure that the quarantine mechanism doesn’t become a bottleneck?
How do you handle edge cases where data might be temporarily unavailable?

**Comparison:**

```
Check Type	Description
Null Rate	Ensures no column exceeds a certain null rate threshold.
Referential Integrity	Ensures foreign keys match referenced records.
Freshness	Verifies data is not older than a specified maximum age.
```

**Code:**

```python
from pyspark.sql import SparkSession, functions as F

# Initialize Spark session
spark = SparkSession.builder.appName("Data Quality Check").getOrCreate()

# Define a function to calculate null rate
def null_rate(column):
    return (F.count(F.when(F.col(column).isNull(), 1)).cast("float") / F.count("*")).alias(f"null_rate_{column}")

# Example schema for demonstration purposes
schema = ["id", "name", "email"]

# Read data from Kafka topic
df = spark.readStream.format("kafka").option("topic", "raw_data_topic").load()

# Apply null rate check
null_checks = df.select(*[null_rate(c) for c in schema])

# Apply referential integrity check (assuming 'id' is a foreign key)
ref_integrity_check = df.join(other=reference_table, on="id", how="left_anti")

# Apply freshness check
freshness_check = df.filter(F.col("timestamp") >= F.current_timestamp() - F.expr("interval 1 day"))

# Combine all checks
quality_checks = null_checks.join(ref_integrity_check, on="id", how="inner").join(freshness_check, on="id", how="inner")

# Quarantine bad batches (save to a different topic or mark them)
quarantine_df = quality_checks.filter((F.col("null_rate_id") > 0.1) | F.col("ref_integrity_check").isNull() | (F.col("timestamp") < F.current_timestamp() - F.expr("interval 1 day")))

# Publish good batches
good_batches = quality_checks.filter((F.col("null_rate_id") <= 0.1) & F.col("ref_integrity_check").isNotNull() & (F.col("timestamp") >= F.current_timestamp() - F.expr("interval 1 day")))
```

**Example input / output:**

```text
Input:
- Raw data from Kafka topic: {"id": 1, "name": "John Doe", "email": null}

Output:
- Quarantined batch (null rate check failed for 'email'):
{"id": 1, "name": "John Doe"}

- Published batch (passed all checks):
{"id": 2, "name": "Jane Smith", "email": "jane@example.com"}
```

**Data:**

```json
{
  "schema": ["id", "name", "email"],
  "reference_table": {
    "columns": ["id", "ref_id"]
  }
}
```

**Flow:**

1. Read from Kafka
2. Null Rate Check
3. Quarantine Batch
4. Publish Batch
5. Log Quatination
6. Log Publication
7. Referential Integrity Check
8. Quarantine Batch
9. Freshness Check
10. Qurantine Batch
11. Log Quarantine
12. Log Quarantine

