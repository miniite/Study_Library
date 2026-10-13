---
type: "practice"
difficulty: "Deep"
---

# A fact load duplicated one day's data after an upstream Kafka replay. How do you detect and cleanly remove the dupes without a full reload?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Data Warehouse Fundamentals](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Data_Warehouse_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of detecting and removing duplicate data from a fact table in a data warehouse due to a delayed Kafka replay, we need to focus on robust ETL (Extract, Transform, Load) processes and leveraging data lake metadata effectively. This problem requires deep understanding of both the data flow and the infrastructure supporting it.

#### Triaging the Question

The core of this question is about identifying and removing duplicate records efficiently in a data warehouse without a full reload. The key elements are:

Understanding Data Sources: Kafka topic as an upstream source.
ETL Process: How data is currently processed and loaded into the data warehouse.
Metadata Management: Leveraging timestamps, sequence numbers, or other metadata to identify duplicates.

#### First Principles Solution

From first principles, we need to ensure that our ETL processes are robust enough to handle such edge cases. Here’s a step-by-step solution:

Identify Duplicate Records:

Use unique keys in the fact table to identify records.
Implement a mechanism to flag or mark duplicated records based on timestamps and sequence numbers.

Data Validation:

Validate incoming data against historical data using a data validation layer.
Store metadata about each batch of data processed, including timestamps and sequence numbers.

Incremental Load and Deduplication:

Implement an incremental load strategy where only new or updated records are processed.
Use a staging area to stage the incoming data before loading it into the final fact table.
Detailed Solution
Step 1: Identify Duplicate Records

To identify duplicate records, we can use a combination of primary keys and timestamps:

SQL
WITH deduped_data AS (
    SELECT
        id,
        value,
        load_timestamp,
        ROW_NUMBER() OVER (PARTITION BY id ORDER BY load_timestamp DESC) as row_num
    FROM raw_fact_table
)
DELETE FROM fact_table WHERE id IN (
    SELECT id FROM deduped_data WHERE row_num > 1
);

This SQL query uses window functions to assign a row number based on the load_timestamp, ensuring that only the latest version of each record is kept.

Step 2: Data Validation and Metadata Management

Implement metadata management by storing information about each batch:

SQL
INSERT INTO metadata_table (batch_id, start_time, end_time) VALUES ('2023-10-05', '2023-10-04T09:00:00Z', '2023-10-05T08:59:59Z');

Use this metadata to validate incoming data:

SQL
INSERT INTO fact_table (id, value, load_timestamp)
SELECT
    id,
    value,
    load_timestamp
FROM raw_fact_table
WHERE load_timestamp NOT IN (
    SELECT end_time FROM metadata_table WHERE batch_id = '2023-10-05'
);

This ensures that only records from the current or relevant batches are loaded.

Step 3: Incremental Load Strategy

Implement an incremental ETL process:

PYTHON
# Pseudo-code for an incremental ETL process in Python using Apache Spark
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("IncrementalETL").getOrCreate()

raw_data = spark.read.format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "topic_name") \
    .load()

# Filter out records that are already in the data warehouse
filtered_data = raw_data.filter(
    ~raw_data.value.cast('string').isin(
        spark.read.table("fact_table").select("id").collect()
    )
)

# Write filtered data to a staging area
filtered_data.write.format("parquet").saveAsTable("staging_fact_table")

# Final load into the fact table
final_data = spark.read.parquet("staging_fact_table")
final_data.write.mode('append').insertInto("fact_table")

This script ensures that only new records are processed and loaded, avoiding duplicates.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, to handle duplicate data in a fact table due to delayed Kafka replays, we need to implement robust ETL processes with metadata management and incremental loading. This approach ensures clean and efficient data warehousing without the need for full reloads.

#### Common mistakes
include:

Not leveraging metadata effectively.
Using a simple delete strategy that might not catch all duplicates.
Failing to validate incoming data thoroughly against historical data.

#### Follow-ups
could include discussing how to handle very large datasets, implementing more sophisticated deduplication strategies, and ensuring data integrity across multiple nodes in a distributed environment.

**Code:**

```

-- SQL Deduplication Query WITH deduped_data AS ( SELECT id, value, load_timestamp, ROW_NUMBER() OVER (PARTITION BY id ORDER BY load_timestamp DESC) as row_num FROM raw_fact_table ) DELETE FROM fact_table WHERE id IN ( SELECT id FROM deduped_data WHERE row_num > 1 );

CODE
Copy
```sql
-- Metadata Table Insertion
INSERT INTO metadata_table (batch_id, start_time, end_time) VALUES ('2023-10-05', '2023-10-04T09:00:00Z', '2023-10-05T08:59:59Z');
SQL
Copy
-- Data Validation Query
INSERT INTO fact_table (id, value, load_timestamp)
SELECT
    id,
    value,
    load_timestamp
FROM raw_fact_table
WHERE load_timestamp NOT IN (
    SELECT end_time FROM metadata_table WHERE batch_id = '2023-10-05'
);
PYTHON
Copy
# Pseudo-code for Incremental ETL in Python using Apache Spark
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("IncrementalETL").getOrCreate()

raw_data = spark.read.format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "topic_name") \
    .load()

filtered_data = raw_data.filter(
    ~raw_data.value.cast('string').isin(
        spark.read.table("fact_table").select("id").collect()
    )
)

filtered_data.write.format("parquet").saveAsTable("staging_fact_table")

final_data = spark.read.parquet("staging_fact_table")
final_data.write.mode('append').insertInto("fact_table")
```

**Example input / output:**

```text
-- Initial Data in Fact Table
INSERT INTO fact_table (id, value, load_timestamp) VALUES ('A', 'value1', '2023-10-04 08:30:00');
INSERT INTO fact_table (id, value, load_timestamp) VALUES ('B', 'value2', '2023-10-04 09:30:00');

-- New Data from Kafka Replay
INSERT INTO raw_fact_table (id, value, load_timestamp) VALUES ('A', 'updated_value1', '2023-10-05 10:00:00');
INSERT INTO raw_fact_table (id, value, load_timestamp) VALUES ('B', 'updated_value2', '2023-10-05 11:00:00');

-- Deduplication and Data Validation
WITH deduped_data AS (
    SELECT
        id,
        value,
        load_timestamp,
        ROW_NUMBER() OVER (PARTITION BY id ORDER BY load_timestamp DESC) as row_num
    FROM raw_fact_table
)
DELETE FROM fact_table WHERE id IN (
    SELECT id FROM deduped_data WHERE row_num > 1
);

INSERT INTO fact_table (id, value, load_timestamp)
SELECT
    id,
    value,
    load_timestamp
FROM raw_fact_table
WHERE load_timestamp NOT IN (
    SELECT end_time FROM metadata_table WHERE batch_id = '2023-10-05'
);
```

**Data:**

```json
{
  "batch_id": "2023-10-05",
  "start_time": "2023-10-04T09:00:00Z",
  "end_time": "2023-10-05T08:59:59Z"
}
```

