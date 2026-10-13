---
type: "practice"
difficulty: "Intermediate"
---

# How would you read a specific older version of a Delta table if a bad load corrupted todays data?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Delta Lake in Databricks](<../../../../TOOLS/Databricks/Databricks_Platform/Delta_Lake_in_Databricks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address how to read a specific older version of a Delta table in Databricks when today's data is corrupted, we need to understand that Delta Lake allows us to access any past state of the table using its versioning capabilities. This involves leveraging Databricks' support for time travel and versioned reads.

Firstly, let’s establish what we are probing:

Understanding Time Travel in Delta Lake: We need to know how to query historical states of a Delta table.
Identifying the Correct Version: Determine which specific version of the data is clean and usable.
Reading Historical Data: Use Databricks APIs or SQL commands to read from this historical state.
Step-by-Step Solution

Identify the Clean Version:

Check Delta Lake metadata for available versions using DESCRIBE HISTORY command in SQL or by querying the underlying catalog.
Identify the version that contains clean data, which might have been before the bad load occurred.

Query Historical Data Using Version Number:

Use Databricks SQL to query a specific version of the table. The syntax involves specifying the AS OF clause with the desired version number or timestamp.

Alternative Approach: Query by Time Range:

If you have an idea about when the bad load occurred, use WHERE clauses in your SQL queries to filter out problematic data.

#### Example
Workflow:

SQL
-- Step 1: Identify clean version
DESCRIBE HISTORY delta_table_name;

-- Step 2: Query historical data using a specific version number
SELECT * FROM delta_table_name AS OF '2023-10-15T14:30:00';

#### Closing Summary
and Common Mistakes

In summary, reading a specific older version of a Delta table involves understanding time travel features in Delta Lake. The key steps are identifying the clean version using metadata queries and then querying that version directly.

#### Common mistakes
include:

Not recognizing that Delta Lake has versioning capabilities.
Failing to use AS OF or time range filters correctly.
Overlooking the need to check for the integrity of data before deciding which version to read.

**Comparison:**

| Time       | Commit ID |
| ------------- | -------------- |
| 2023-10-15T14:00:00 | v1          |
| 2023-10-15T14:30:00 | v2          |
| 2023-10-15T15:00:00 | v3          |

**Code:**

```sql
-- Example SQL query to read a specific version
SELECT * FROM delta_table_name AS OF '2023-10-15T14:30:00';
```

**Example input / output:**

```text
// Input to list versions
DESCRIBE HISTORY delta_table_name;

// Output of the history command
+---------------------+------------+
|         time        | commit_id  |
+---------------------+------------+
| 2023-10-15T14:00:00 | v1         |
| 2023-10-15T14:30:00 | v2         |
| 2023-10-15T15:00:00 | v3         |
+---------------------+------------+
```

**Data:**

```json
{
  "version_history": [
    {"time": "2023-10-15T14:00:00", "commit_id": "v1"},
    {"time": "2023-10-15T14:30:00", "commit_id": "v2"},
    {"time": "2023-10-15T15:00:00", "commit_id": "v3"}
  ]
}
```

**Flow:**

1. Identify Clean Version
2. Check Delta Table History
3. Select Specific Version UsingAS OF Clause
4. Query Historical Data
5. Verify Integrity of Data

