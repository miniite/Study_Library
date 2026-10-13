---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd migrate a set of legacy Parquet tables to Delta with zero downtime for downstream consumers still reading the old paths.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Delta Lake in Databricks](<../../../../TOOLS/Databricks/Databricks_Platform/Delta_Lake_in_Databricks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To migrate a set of legacy Parquet tables to Delta with zero downtime, we need to ensure that the transition is seamless and does not affect downstream consumers who are currently accessing the data via the old paths. This involves several steps: creating a Delta table, copying data from Parquet to Delta, configuring metastore changes, and updating access points to point to the new Delta tables.

#### Triaging the Question

The question requires understanding how to migrate existing data from a legacy format (Parquet) to a modern storage solution (Delta) without interrupting service. The key aspects include:

Ensuring zero downtime for consumers.
Migrating data efficiently.
Managing schema changes if any are needed.

#### Assumptions
and Key Considerations
Legacy Parquet Tables: Existing tables in Parquet format, currently used by downstream systems.
Delta Lake: A modern storage layer that supports ACID transactions and versioning.
No Downtime for Consumers: Downstream applications must continue to read data from the old paths until all data is migrated.
Step-by-Step Solution
1. Preparation

First, prepare the environment by ensuring that Databricks clusters are set up with both Parquet and Delta support.

SQL
-- Example SQL commands in a Databricks notebook
CREATE TABLE IF NOT EXISTS legacy_parquet_table (
  id INT,
  name STRING,
  value DOUBLE
)
USING PARQUET
LOCATION '/mnt/legacy-parquet';

CREATE TABLE IF NOT EXISTS new_delta_table (
  id INT,
  name STRING,
  value DOUBLE
)
USING DELTA
LOCATION '/mnt/new-delta';
2. Data Migration

Copy the data from Parquet to Delta in batches to minimize impact on performance.

PYTHON
# Using PySpark API
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("Parquet to Delta Migrate").getOrCreate()

parquet_df = spark.read.parquet("/mnt/legacy-parquet")
delta_df = parquet_df.write.format("delta").mode("append").save("/mnt/new-delta")

# Alternatively, using Databricks UI
# Copy Data Wizard -> Select Source and Destination tables -> Run
3. Metastore Configuration

Ensure the metastore is updated to reflect the new Delta table paths.

SQL
-- Example SQL commands in a Databricks notebook
ALTER TABLE legacy_parquet_table SET TBLPROPERTIES ('delta.formatVersion' = '2');
ALTER TABLE new_delta_table SET TBLPROPERTIES ('delta.formatVersion' = '2');
4. Update Access Paths

Update any references to the old Parquet tables in downstream applications, databases, and ETL pipelines to point to the new Delta tables.

SQL
-- Example SQL commands in a Databricks notebook for updating access paths
ALTER TABLE legacy_parquet_table RENAME TO new_delta_table;
5. Testing and Validation

Thoroughly test the new setup by running queries on both old and new paths to ensure data integrity.

SQL
-- Test Data Integrity
SELECT * FROM legacy_parquet_table
SELECT * FROM new_delta_table
6. Decommission Legacy Tables (Optional)

Once validation is successful, safely decommission the legacy tables if no longer needed for read access.

SQL
-- Example SQL commands in a Databricks notebook
DROP TABLE IF EXISTS legacy_parquet_table;

#### Closing Summary
and Common Mistakes

By following these steps, we ensure a smooth transition from Parquet to Delta without downtime. Key mistakes include:

Forgetting to update metastore properties.
Not thoroughly testing the new setup before decommissioning legacy tables.
Incomplete data migration, leading to inconsistencies.

#### Follow-Ups
How would you handle schema changes during the migration?
What about handling deletes and updates in real-time while migrating?
Discuss any potential performance bottlenecks and how to mitigate them.

**Comparison:**

```
Legacy Table	New Delta Table
id, name, value	id, name, value
Metastore Properties
delta.formatVersion = '2'
```

**Code:**

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("Parquet to Delta Migrate").getOrCreate()

# Read Parquet data
parquet_df = spark.read.parquet("/mnt/legacy-parquet")

# Write to Delta table
delta_df = parquet_df.write.format("delta").mode("append").save("/mnt/new-delta")
```

**Example input / output:**

```text
-- Test Data Integrity
SELECT * FROM legacy_parquet_table
-- Expected output matches data in /mnt/legacy-parquet

SELECT * FROM new_delta_table
-- Expected output matches data in /mnt/new-delta
```

**Data:**

```sql
CREATE TABLE IF NOT EXISTS legacy_parquet_table (
  id INT,
  name STRING,
  value DOUBLE
)
USING PARQUET
LOCATION '/mnt/legacy-parquet';

CREATE TABLE IF NOT EXISTS new_delta_table (
  id INT,
  name STRING,
  value DOUBLE
)
USING DELTA
LOCATION '/mnt/new-delta';
```

**Flow:**

1. Prepare Environment
2. Create Legacy Parquet Table
3. Create New Delta Table
4. Migrate Data from Parquet toDelta in Batches
5. Update Metastore Properties
6. Update Access Paths
7. Test Data Integrity
8. Decommission Legacy Tables(Optional)
9. Fine-tune Performance

