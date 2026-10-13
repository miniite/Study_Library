---
type: "practice"
difficulty: "Intermediate"
---

# Why does sorting data before writing Parquet dramatically improve later pruning, and how do you decide the sort key for a lake table?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Min Max Statistics and Data Skipping](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Min_Max_Statistics_and_Data_Skipping.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

Sorting data before writing it to Parquet files can significantly enhance query performance by improving pruning efficiency during subsequent queries. The key idea is that sorting allows more effective use of index-based pruning, reducing the amount of data scanned and processed.

#### Triaging the Question

The core of this question revolves around understanding how sorting impacts the efficiency of querying large datasets stored in Parquet format. Specifically, we need to explore why sorted data improves query performance through pruning mechanisms and discuss how to select an appropriate sort key for a lake table. The interviewer is probing your knowledge of indexing strategies and their impact on query execution.

#### First-Principles Solution
1. Understanding Parquet's Pruning Mechanism:
Parquet uses columnar storage, which means each column can be compressed and stored separately.
During a query, metadata in the Parquet files allows for efficient pruning of columns that do not match the query predicates.
2. Sorting Data and Its Impact on Querying:
Pruning Efficiency: When data is sorted, Parquet's row groups can be more effectively pruned during queries. For example, if a query involves filtering by a specific value in an ordered column, only the relevant row group needs to be scanned.
Reduced Data Read: By sorting and partitioning based on common filter columns (e.g., timestamp or ID), fewer rows need to be read from disk.
3. Choosing the Sort Key:
The sort key should align with frequent query patterns. For instance, if most queries are filtering by date, sorting by date can significantly improve performance.

#### Cons
ider the trade-offs between different columns:
Frequency of Query: Columns used in more frequent queries should be prioritized.
Data Distribution: Evenly distributed data reduces skew and improves overall query efficiency.
4. Practical Example:
Assume a lake table with columns [timestamp, user_id, event_type].
If most queries filter by timestamp, sorting the data on timestamp would be beneficial.
A potential schema might look like this:
SQL
CREATE TABLE events (
    timestamp TIMESTAMP,
    user_id INT,
    event_type STRING
)
PARTITIONED BY (timestamp)
STORED AS PARQUET;
5. Deciding the Sort Key:
For a timestamp-based lake table:
SQL
CREATE TABLE events (
    timestamp TIMESTAMP,
    user_id INT,
    event_type STRING
)
CLUSTERED BY (timestamp) SORTED BY (user_id, event_type)
PARTITIONED BY (year(timestamp), month(timestamp))
STORED AS PARQUET;
This sorts the data by timestamp and further by user_id and event_type, ensuring that queries can quickly locate relevant records.

#### Closing Summary

Sorting data before writing it to Parquet files enhances query performance by allowing more effective pruning mechanisms. The sort key should be chosen based on common query patterns, reducing unnecessary data reads. In practice, sorting by timestamp is often beneficial in time-series data lakes.

#### Common Mistakes
and Follow-Ups:
Mistake: Choosing a less frequent or non-relevant column as the sort key can lead to suboptimal performance.

#### Follow-Up
What happens if multiple columns are frequently queried? How do you decide between them?
Mistake: Ignoring data skew, which can still occur even with sorting. Partitioning by additional dimensions (e.g., year, month) helps mitigate this.

**Comparison:**

| Sort Key | Common Queries |
|----------|----------------|
| timestamp | filter by date, time range queries |
| user_id   | identify users with specific events |
| event_type| log different types of user actions |

**Code:**

```sql
CREATE TABLE events (
    timestamp TIMESTAMP,
    user_id INT,
    event_type STRING
)
CLUSTERED BY (timestamp) SORTED BY (user_id, event_type)
PARTITIONED BY (year(timestamp), month(timestamp))
STORED AS PARQUET;
```

**Example input / output:**

```text
Input Query:
SELECT * FROM events WHERE timestamp >= '2023-10-01' AND user_id = 12345

Output Response:
Only the rows within the relevant row group (partitioned by year and month) are scanned.
```

**Data:**

```json
{
  "events": [
    {"timestamp": "2023-10-01", "user_id": 12345, "event_type": "login"},
    {"timestamp": "2023-10-02", "user_id": 67890, "event_type": "purchase"},
    ...
  ]
}
```

**Flow:**

1. Yes
2. No
3. Input Query
4. Is the query on a sortedcolumn?
5. Prune and scan relevant rowgroup
6. Scan all data, then apply filters

