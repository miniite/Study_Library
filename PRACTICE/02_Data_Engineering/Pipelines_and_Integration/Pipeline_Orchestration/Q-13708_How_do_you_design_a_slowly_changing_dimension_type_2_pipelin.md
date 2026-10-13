---
type: "practice"
difficulty: "Deep"
---

# How do you design a slowly changing dimension type 2 pipeline in Spark that correctly closes out old records and inserts new versions with valid-from/valid-to?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Schema Drift Handling](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Schema_Drift_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a slowly changing dimension type 2 pipeline using Apache Spark, we need to ensure that we maintain historical data by adding a valid_from and valid_to timestamp column whenever there is an update in the dimension table. This approach preserves the history of each record while accommodating new versions with updated attributes.

The core idea here is to split the process into two main steps: identifying changes and updating records accordingly. We will use Apache Spark's DataFrame API for its powerful data manipulation capabilities, ensuring that our pipeline can handle large datasets efficiently.

#### Triage

The question is asking how to implement a type 2 slowly changing dimension in Spark, focusing on maintaining historical records while inserting new versions with updated attributes. The interviewer is probing your understanding of:

Data transformation logic
Use of timestamps for versioning
Handling updates and inserts without losing historical data
Efficient data processing using Spark

We assume the following:

We have a source table that contains the current dimension data.
There is a periodic update process (e.g., hourly, daily) to add new or updated records.
The target table must maintain a history of all changes.

#### First-Principles Solution
Step 1: Identify Changes

First, we need to identify which rows have been updated in the source table compared to the existing data in the target table. This can be achieved using an anti-join or semi-join operation to find records that exist only in the source and not in the target.

SQL
val updatedRecords = sourceTable.join(targetTable, Seq("keyColumn"), "anti")
Step 2: Insert New Records with Valid From/To Timestamps

For each new record identified in updatedRecords, we insert a new row into the target table with a valid_from timestamp set to the current time and valid_to as null (indicating it is still active).

SCALA
val updatedWithValidTimestamps = updatedRecords.withColumn("valid_from", lit(current_timestamp()))
val upsertedNew = updatedWithValidTimestamps.map(row => (row.getString(0), row.getString(1), row.getTimestamp(2)))
upsertedNew.foreach { case (key, value, validFrom) =>
  // Upsert logic
}
Step 3: Update Existing Records with Valid To Timestamps

Next, we need to update existing records in the target table that have been modified. This involves setting their valid_to timestamp to the current time and inserting a new record for the updated version.

SCALA
val existingRecords = sourceTable.join(targetTable, Seq("keyColumn"), "inner")
  .where($"target.valid_to".isNull) // Only consider records not yet marked as deleted

// Update valid_to timestamps
val updatedExisting = existingRecords.withColumn("valid_to", lit(current_timestamp()))

// Insert new versions with valid_from and valid_to
val upsertedExisting = updatedExisting.map(row => (row.getString(0), row.getString(1), row.getTimestamp(2), row.getTimestamp(3)))
upsertedExisting.foreach { case (key, value, validFrom, validTo) =>
  // Upsert logic
}
Step 4: Clean Up Old Records

Finally, we need to clean up old records by marking their valid_to as the current time. This step ensures that historical data is properly archived.

SCALA
val expiredRecords = targetTable.filter($"valid_to".isNull).where(not("keyColumn" in sourceTable.columns))
val cleanedUp = expiredRecords.withColumn("valid_to", lit(current_timestamp()))
cleanedUp.foreach { row =>
  // Mark records as expired
}

#### Closing Summary
and Common Mistakes

To summarize, the key steps are:

Identify updated records.
Insert new versions with valid_from and valid_to timestamps.
Update existing records by setting their valid_to timestamp.
Clean up old records.

#### Common mistakes
to avoid include:

Overwriting historical data instead of updating or inserting correctly.
Failing to handle time zones properly, especially when using current timestamps.
Not testing edge cases like concurrent updates.
Common Follow-Ups
Concurrency: How do you ensure that updates are atomic and consistent?
Performance Optimization: What optimizations can be applied to improve the pipeline's performance?
Error Handling: How do you handle errors during the upsert process?

**Comparison:**

```
Join & Anti Join
Source Table
Updated Records
Target Table Before
New Version Inserted withValid From/To Timestamps
Existing Records (Valid To Null)
Updated Existing Records withValid To Timestamps
Expired Records Marked asExpired
Target Table After
```

**Code:**

```scala
import org.apache.spark.sql.functions._
val updatedRecords = sourceTable.join(targetTable, Seq("keyColumn"), "anti")
val currentTimestamp = current_timestamp()
val updatedWithValidTimestamps = updatedRecords.withColumn("valid_from", lit(currentTimestamp))
updatedWithValidTimestamps.foreach { row =>
  // Upsert logic
}

val existingRecords = sourceTable.join(targetTable, Seq("keyColumn"), "inner")
  .where($"target.valid_to".isNull)
val updatedExisting = existingRecords.withColumn("valid_to", current_timestamp())
updatedExisting.foreach { row =>
  // Upsert logic
}

val expiredRecords = targetTable.filter($"valid_to".isNull).where(not("keyColumn" in sourceTable.columns))
val cleanedUp = expiredRecords.withColumn("valid_to", current_timestamp())
cleanedUp.foreach { row =>
  // Mark records as expired
}
```

**Example input / output:**

| Source Table Update | Target Table Before | Target Table After |
|---------------------|--------------------|--------------------|
| Alice's email updated to alice_new@example.com | {"id":1,"name":"Alice","email":"alice@example.com","valid_from":"2023-01-01T00:00:00Z","valid_to":null}, {"id":2,"name":"Bob","email":"bob@example.com","valid_from":"2023-01-01T00:00:00Z","valid_to":null} | {"id":1,"name":"Alice","email":"alice_new@example.com","valid_from":"2023-01-01T14:00:00Z","valid_to":null}, {"id":2,"name":"Bob","email":"bob@example.com","valid_from":"2023-01-01T00:00:00Z","valid_to":"2023-01-01T14:00:00Z"} |

**Data:**

```json
{
  "sourceTable": {
    "columns": ["id", "name", "email", "valid_from", "valid_to"],
    "data": [
      {"id": 1, "name": "Alice", "email": "alice@example.com"},
      {"id": 2, "name": "Bob", "email": null}
    ]
  },
  "targetTable": {
    "columns": ["id", "name", "email", "valid_from", "valid_to"],
    "data": [
      {"id": 1, "name": "Alice", "email": "alice@example.com", "valid_from": "2023-01-01T00:00:00Z", "valid_to": null},
      {"id": 2, "name": "Bob", "email": "bob@example.com", "valid_from": "2023-01-01T00:00:00Z", "valid_to": null}
    ]
  }
}
```

**Flow:**

1. Join & Anti Join
2. Source Data
3. Updated Records
4. Insert New Versions with ValidFrom/To Timestamps
5. Update Existing Records (SetValid To)
6. Mark Expired Records asExpired
7. Cleaned Up Target Table

