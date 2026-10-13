---
type: "practice"
difficulty: "Expert"
---

# When you migrate a legacy three-hop Hadoop pipeline into a medallion lakehouse, how do you sequence it so you never have a big-bang cutover and can always roll back?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Controlling Gold Sprawl](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Medallion_Architecture/Controlling_Gold_Sprawl.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

To migrate a legacy three-hop Hadoop pipeline into a Medallion Lakehouse without a big-bang cutover, you need to carefully sequence the migration by leveraging incremental data processing and versioning mechanisms. This approach ensures that each layer (Bronze, Silver, Gold) can be independently validated and rolled back if necessary.

#### Triaging the Question

The core of this question is about ensuring seamless, controlled migration from a legacy Hadoop pipeline to a Medallion Lakehouse architecture while maintaining rollback capabilities. The interviewer is probing your understanding of data lineage, incremental processing, version control, and the principles of robust migration strategies.

#### Assumptions
The current three-hop Hadoop pipeline processes raw data (Bronze), cleans and enriches it (Silver), and then transforms it for analysis (Gold).
The Medallion Lakehouse will maintain these layers while introducing modern storage and processing technologies.
Rollback mechanisms are essential to ensure data integrity during the migration.

#### First Principles Solution

Data Layering in Medallion Architecture:

Bronze Layer: Raw, unprocessed data from various sources.
Silver Layer: Cleaned and transformed Bronze data with basic lineage tracking.
Gold Layer: Analyzed data ready for business insights.

Incremental Data Processing:

Use incremental or delta processing techniques to minimize the initial load time during migration.
Implement a pipeline that can process new incoming data while maintaining existing data in its current state until full validation is achieved.

Version Control and Validation:

For each layer, maintain multiple versions of data for rollback purposes.
Validate Silver and Gold layers independently using checksums or hashes before fully replacing the old Hadoop pipeline.

Parallel Execution Strategy:

Execute the migration in parallel batches: start by moving Bronze data incrementally to ensure foundational integrity.
Gradually migrate Silver and then Gold layers, verifying each step through automated validation processes.

Rollback Mechanisms:

Ensure that rollback capabilities are defined for each layer. For instance, if a transformation rule fails during the migration of the Gold layer, revert to a previous valid state using version-controlled data.
Implement change management practices to track and log all changes made during the migration.

#### Closing Summary
+ Common Mistakes/Follow-Ups:
Key Takeaways: Incremental processing, versioning, validation, parallel execution, and robust rollback mechanisms are critical for successful legacy pipeline migrations in a Medallion Lakehouse.
Common Pitfalls: Failing to maintain adequate logging and lineage tracking can lead to data discrepancies. Overlooking comprehensive testing before full cutover can result in significant disruptions.
Common Follow-Ups:
How do you ensure data consistency during the migration?
What tools or technologies would you use for version control in a Medallion Lakehouse?
Can you walk through an example of how incremental processing works in practice?

**Comparison:**

```
Layer	Data Type	Processing	Validation
Bronze	Raw	Incremental	N/A
Silver	Processed	Incremental	Hashes
Gold	Analyzed	Full Batch	Checksums
```

**Code:**

```python
# Example Python code for incremental data processing using Apache Spark
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("MedallionMigration").getOrCreate()

# Read raw Bronze data incrementally
bronze_df = spark.readStream.format("csv") \
    .option("path", "raw_data") \
    .load()

# Transform and write to Silver layer
silver_df = bronze_df.transform(transform_rules)
silver_writer = silver_df.writeStream \
    .outputMode("append") \
    .format("delta") \
    .option("checkpointLocation", "silver_checkpoint")

# Example of validation step before full cutover
validation_query = silver_df.selectExpr("SUM(validation_column)") \
    .writeStream \
    .outputMode("complete") \
    .trigger(once=True) \
    .start()
```

**Example input / output:**

```text
Input (Bronze Layer):
- CSV files with raw data

Output (Silver Layer):
- Delta tables with transformed and validated data
```

**Data:**

```json
{
  "bronze_layer": {
    "path": "/raw_data",
    "format": "csv"
  },
  "silver_layer": {
    "path": "/processed_data/silver",
    "format": "delta",
    "checkpointLocation": "/silver_checkpoint"
  },
  "gold_layer": {
    "path": "/analyzed_data/gold",
    "format": "parquet"
  }
}
```

**Flow:**

1. Incremental ||| Full Batch
2. Hashes ||| Checksums
3. Rollback Mechanism
4. Routine Backups
5. Manual Reversion
6. Automated Rollbacks
7. Version Control
8. Previous State
9. Current State
10. New State
11. Raw Data
12. Bronze Layer
13. Silver Layer
14. Gold Layer

