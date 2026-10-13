---
type: "practice"
difficulty: "Deep"
---

# Describe how you'd implement an idempotent PySpark batch write to a partitioned table so reruns don't create duplicates.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reducing Shuffle in PySpark](<../../../../TOOLS/Apache_Spark/PySpark/Reducing_Shuffle_in_PySpark.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement an idempotent PySpark batch write to a partitioned table, ensuring that reruns do not create duplicates, we need to focus on leveraging both PySpark's capabilities and best practices in data processing. The core idea is to ensure atomicity of the write operation while maintaining uniqueness across multiple runs.

#### Triage

The question asks for an implementation of an idempotent PySpark batch write. This means that if the code is run repeatedly, it should not alter the final state of the database beyond the first execution. Specifically, we need to handle scenarios where partitioned tables are involved and ensure that duplicate records are not written during subsequent runs.

#### Assumptions
We are dealing with a large dataset that needs to be processed in batches.
The data is partitioned by certain key columns (e.g., date).
Reruns of the job should not add duplicates, but they should still allow for updates or deletions if necessary.

#### First-Principles Solution
Identify Unique Columns: Determine which columns define the uniqueness of your records.
Use Temp Tables/Cache: Cache the current state in a temporary table before writing to avoid overwriting existing data.
Upsert Logic: Implement an upsert strategy that updates existing records and inserts new ones.
Step-by-Step Solution

Identify Unique Columns:

Suppose we have a DataFrame df with columns [user_id, event_date, event_type].
The unique identifier for each record is the combination of these columns.

Cache Current State:

PYTHON
from pyspark.sql import SparkSession

spark = SparkSession.builder.getOrCreate()
# Load the existing data in a partitioned table into a DataFrame
existing_data_df = spark.read.parquet("/path/to/partitioned/table")

Merge with New Data:

Use DataFrame operations to merge the new and existing datasets.
PYTHON
# Assuming 'new_data' is your incoming DataFrame
merged_df = df.unionByName(existing_data_df, allowMissingColumns=True)

Generate a Temporary Unique ID for Duplicates:

PYTHON
from pyspark.sql.functions import monotonically_increasing_id

# Assign unique IDs to each row in the merged DataFrame
tmp_unique_id = 'tmp_unique_id'
merged_df = merged_df.withColumn(tmp_unique_id, monotonically_increasing_id())

Upsert Logic:

For upsert, we can use a combination of when and otherwise to update or insert based on the presence of the unique identifier.
PYTHON
from pyspark.sql.functions import when

# Define the columns that make the record unique
unique_cols = ['user_id', 'event_date', 'event_type']

# Create a DataFrame for upsert operations
upsert_df = merged_df \
    .withColumn('is_duplicate', existing_data_df.select(unique_cols).isin(merged_df.select(unique_cols)).any()) \
    .withColumn('action', when(existing_data_df.select(unique_cols).isin(merged_df.select(unique_cols)).all(), 'update') \
                .otherwise('insert'))

# Apply the upsert logic
updated_df = upsert_df \
    .withColumn('new_value', when(upsert_df['is_duplicate'], existing_data_df[unique_cols]) \
                .otherwise(merged_df[unique_cols])) \
    .select(unique_cols + ['new_value'])

Write Back to Partitioned Table:

Write the updated DataFrame back to the partitioned table.
PYTHON
# Repartition and write the final DataFrame back to the parquet files
updated_df.repartition('user_id', 'event_date').write.mode('overwrite').parquet("/path/to/partitioned/table")

#### Closing Summary
+ Common Mistakes/Follow-Ups

The key steps involve using temporary tables, merging datasets, generating unique identifiers for duplicates, and applying upsert logic. This ensures that reruns do not add duplicate records while allowing updates or deletions as needed.

#### Common Mistakes

Failing to handle partition keys correctly.
Not caching the current state before writing new data.
Overwriting existing partitions without proper checks for duplicates.

#### Follow-Ups

How would you handle partial failures during the write process?
What are the performance implications of this approach, and how can we optimize it?

**Comparison:**

| user_id | event_date | event_type |
|---------|------------|------------|
| 1       | 2023-01-01 | login      |
| 4       | 2023-01-01 | logout     |
| 5       | 2023-01-02 | login      |

**Code:**

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.getOrCreate()
existing_data_df = spark.read.parquet("/path/to/partitioned/table")
new_data = ...  # Your DataFrame with new data
tmp_unique_id = 'tmp_unique_id'
merged_df = df.unionByName(existing_data_df, allowMissingColumns=True)
merged_df = merged_df.withColumn(tmp_unique_id, monotonically_increasing_id())
upsert_df = merged_df \
    .withColumn('is_duplicate', existing_data_df.select(unique_cols).isin(merged_df.select(unique_cols)).any()) \
    .withColumn('action', when(existing_data_df.select(unique_cols).isin(merged_df.select(unique_cols)).all(), 'update') \
                .otherwise('insert'))
updated_df = upsert_df \
    .withColumn('new_value', when(upsert_df['is_duplicate'], existing_data_df[unique_cols]) \
                .otherwise(merged_df[unique_cols])) \
    .select(unique_cols + ['new_value'])
updated_df.repartition('user_id', 'event_date').write.mode('overwrite').parquet("/path/to/partitioned/table")
```

**Example input / output:**

```text
Input DataFrame:
user_id | event_date | event_type
-------------------------
1       | 2023-01-01 | login
2       | 2023-01-01 | logout
3       | 2023-01-02 | login

Existing Partitioned Table:
user_id | event_date | event_type
-------------------------
1       | 2023-01-01 | login
4       | 2023-01-01 | logout
5       | 2023-01-02 | login

Output DataFrame:
user_id | event_date | event_type
-------------------------
1       | 2023-01-01 | login  # Duplicate, no change
4       | 2023-01-01 | logout # New record inserted
5       | 2023-01-02 | login  # Duplicate, updated to match new data
```

**Data:**

```json
{
  "user_id": [1, 2, 3],
  "event_date": ["2023-01-01", "2023-01-01", "2023-01-02"],
  "event_type": ["login", "logout", "login"]
}
```

**Flow:**

1. Start
2. Read Existing Data
3. Union with New Data
4. Generate Unique ID
5. Upsert Logic
6. Write Back to Partitioned Table
7. End

