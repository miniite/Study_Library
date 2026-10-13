---
type: "practice"
difficulty: "Deep"
---

# How would you handle a source that resends the last hour of data on every extract to avoid missing rows, and dedupe it in SQL?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Deduplication in SQL](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Deduplication_in_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue where a source resends the last hour of data during each extraction to ensure no rows are missed, we need to design an efficient solution that leverages SQL to handle duplicates properly while maintaining data integrity. The key challenge is ensuring that only new or updated records from the last hour are inserted or updated in the target database.

In essence, the question revolves around how to write a robust SQL query that can filter out duplicate rows efficiently and manage time-based filtering effectively.

#### Triage

The core of this problem involves:

Filtering Data by Time: Ensuring only the latest hour's data is processed.
Handling Duplicates: Managing records that might have been resent due to resends from the source system.
Efficiency and Performance: Optimizing queries for large datasets.

#### First-Principles Solution
Define the Last Hour Data: Use a timestamp or date column in your data to filter only rows from the last hour.
Identify Duplicates: Ensure that any duplicate records are handled appropriately, usually by marking them as MERGE targets for upsert operations.
Step-by-Step Solution

Schema and Data Understanding: Assume we have a table source_data with columns: id, value, timestamp.

Filter the Last Hour Data:

SQL
SELECT id, value, timestamp
FROM source_data
WHERE timestamp >= CURRENT_TIMESTAMP - INTERVAL '1 hour';

Handling Duplicates: To handle duplicates, we can use a MERGE statement to upsert (insert or update) the data into the target table.

#### Example
MERGE Statement:

SQL
WITH latest_data AS (
  SELECT id, value, timestamp
  FROM source_data
  WHERE timestamp >= CURRENT_TIMESTAMP - INTERVAL '1 hour'
)
MERGE INTO target_data t
USING latest_data l ON (t.id = l.id)
WHEN MATCHED THEN
  UPDATE SET t.value = l.value
WHEN NOT MATCHED THEN
  INSERT (id, value, timestamp) VALUES (l.id, l.value, l.timestamp);

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: The solution involves filtering the last hour's data using a WHERE clause with a time interval. Then, using a MERGE statement to handle duplicates efficiently.

#### Common Mistakes
Failing to include the time interval in the filter condition.
Not handling duplicates correctly (e.g., not using MERGE for upsert operations).
Ignoring performance optimizations, such as adding appropriate indexes on columns used in the join.

#### Example
Data
JSON
{
  "source_data": [
    {"id": 1, "value": "A", "timestamp": "2023-10-05T14:00:00Z"},
    {"id": 2, "value": "B", "timestamp": "2023-10-05T14:30:00Z"}
  ]
}
Dry-Run Example
SQL
WITH latest_data AS (
  SELECT id, value, timestamp
  FROM source_data
  WHERE timestamp >= '2023-10-05 14:00:00'
)
MERGE INTO target_data t
USING latest_data l ON (t.id = l.id)
WHEN MATCHED THEN
  UPDATE SET t.value = l.value
WHEN NOT MATCHED THEN
  INSERT (id, value, timestamp) VALUES (l.id, l.value, l.timestamp);
Self-Check

The solution is correct and complete.

**Comparison:**

| id | value |
|----|-------|
| 1  | A     |
| 2  | B     |
| id | value | timestamp         |
|----|-------|------------------|
| 1  | A'    | 2023-10-05T14:00:00Z |
| 2  | B'    | 2023-10-05T14:30:00Z |

**Code:**

```sql
WITH latest_data AS (
  SELECT id, value, timestamp
  FROM source_data
  WHERE timestamp >= CURRENT_TIMESTAMP - INTERVAL '1 hour'
)
MERGE INTO target_data t
USING latest_data l ON (t.id = l.id)
WHEN MATCHED THEN
  UPDATE SET t.value = l.value
WHEN NOT MATCHED THEN
  INSERT (id, value, timestamp) VALUES (l.id, l.value, l.timestamp);
```

**Example input / output:**

```text
WITH latest_data AS (
  SELECT id, value, timestamp
  FROM source_data
  WHERE timestamp >= '2023-10-05 14:00:00'
)
MERGE INTO target_data t
USING latest_data l ON (t.id = l.id)
WHEN MATCHED THEN
  UPDATE SET t.value = l.value
WHEN NOT MATCHED THEN
  INSERT (id, value, timestamp) VALUES (l.id, l.value, l.timestamp);
```

**Data:**

```json
{
  "source_data": [
    {"id": 1, "value": "A", "timestamp": "2023-10-05T14:00:00Z"},
    {"id": 2, "value": "B", "timestamp": "2023-10-05T14:30:00Z"}
  ]
}
```

**Flow:**

1. Start
2. Filter Last Hour Data
3. Merge Into Target Table
4. Upsert Records
5. End

