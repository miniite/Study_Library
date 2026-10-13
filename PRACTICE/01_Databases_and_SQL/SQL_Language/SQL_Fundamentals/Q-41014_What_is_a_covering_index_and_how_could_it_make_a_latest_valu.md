---
type: "practice"
difficulty: "Deep"
---

# What is a covering index, and how could it make a 'latest value per host' query use an index-only scan?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Leftmost Prefix Rule](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Leftmost_Prefix_Rule.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A covering index in SQL is a type of index that includes all the columns needed by a query. This means the database can fetch all necessary data directly from the index, without needing to look up additional rows in the underlying table. For a 'latest value per host' query, this technique can enable an index-only scan, reducing disk I/O and speeding up query performance.

Let's break down the question and understand how covering indexes work in this context.

#### Triage

The question is asking about covering indexes specifically in the context of optimizing a "latest value per host" query. The key points are:

Covering Index: A composite index that includes all necessary columns for a particular query.
Index-Only Scan: A PostgreSQL-specific feature where only the index data is used to satisfy the query, reducing disk I/O.

#### Assumptions
We have a table with multiple hosts and their respective values at different times.
The goal is to find the latest value per host efficiently using an index-only scan.

#### First-Principles Solution
Understand the Query: To get the latest value per host, we need to identify the most recent timestamp for each host and retrieve its associated value. This typically involves a GROUP BY clause combined with MAX() function on the timestamp column.
Create a Covering Index: A covering index should include all columns referenced in the query—typically the host ID, timestamp, and value.

Here's an example of how you might define such an index:

SQL
CREATE INDEX idx_latest_value_per_host ON my_table (host_id, ts DESC, value);

This composite index includes host_id, which is used for grouping, ts to determine the latest value, and value as the target column. The DESC order ensures that the most recent timestamp comes first.

Query Optimization: When you run a query like:
SQL
SELECT host_id, MAX(ts) AS max_ts, value
FROM my_table
GROUP BY host_id;

PostgreSQL can use this covering index to perform an index-only scan because it contains all required columns: host_id, the latest ts, and the corresponding value.

Full Solution
SQL
-- Step 1: Create a covering index
CREATE INDEX idx_latest_value_per_host ON my_table (host_id, ts DESC, value);

-- Step 2: Run the query with an index-only scan
SELECT host_id, MAX(ts) AS max_ts, value
FROM my_table
GROUP BY host_id;

#### Closing Summary
and Common Mistakes

To summarize, a covering index is essential for optimizing queries that require fetching the latest values per group. By including all necessary columns in the index, you can leverage an index-only scan, reducing I/O operations and improving query performance.

#### Common mistakes
to avoid:

Not Including All Required Columns: Ensure your index includes every column used in the query.
Ignoring Index Order: The order of columns in a composite index matters; use descending (DESC) for timestamp columns if you want the latest values first.
Overusing Covering Indexes: While useful, covering indexes can increase storage costs and maintenance overhead. Use them judiciously based on your workload.

**Comparison:**

| Column | Description |
|--------|-------------|
| id     | Unique identifier for each row. |
| host_id | Identifier for the host. |
| ts     | Timestamp of the measurement. |
| value  | The measured value. |

**Code:**

```sql
-- Example SQL statements
CREATE INDEX idx_latest_value_per_host ON my_table (host_id, ts DESC, value);
```

**Example input / output:**

```text
-- Input
CREATE INDEX idx_latest_value_per_host ON my_table (host_id, ts DESC, value);

SELECT host_id, MAX(ts) AS max_ts, value
FROM my_table
GROUP BY host_id;

-- Output
-- Results will vary based on the data in 'my_table', but may look like:
-- host_id: 101, max_ts: "2023-10-02T11:00:00Z", value: 6
-- host_id: 102, max_ts: "2023-10-03T12:00:00Z", value: 7
```

**Data:**

```json
{
  "my_table": {
    "columns": ["id", "host_id", "ts", "value"],
    "rows": [
      {"id": 1, "host_id": 101, "ts": "2023-10-01T10:00:00Z", "value": 5},
      {"id": 2, "host_id": 101, "ts": "2023-10-02T11:00:00Z", "value": 6},
      {"id": 3, "host_id": 102, "ts": "2023-10-03T12:00:00Z", "value": 7}
    ]
  }
}
```

**Flow:**

1. No
2. Yes
3. Create Covering Index
4. Is the index used?
5. Run Query with Index-OnlyScan
6. Fetch Latest Values Efficiently

