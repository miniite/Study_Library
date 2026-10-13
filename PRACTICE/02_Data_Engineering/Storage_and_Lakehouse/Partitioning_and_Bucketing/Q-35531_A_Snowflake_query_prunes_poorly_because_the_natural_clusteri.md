---
type: "practice"
difficulty: "Deep"
---

# A Snowflake query prunes poorly because the natural clustering key is scattered across micro-partitions — how do you decide the clustering key and measure pruning improvement?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Micro-partitions vs Manual Partitions](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Micro-partitions_vs_Manual_Partitions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address a Snowflake query that prunes poorly due to a scattering of records across micro-partitions, we need to carefully select the appropriate clustering key. This involves understanding how partitioning works in Snowflake, evaluating the distribution of data, and measuring the effectiveness of different keys before deploying them. Let's break down this process from first principles.

Partitioning in Snowflake divides data into larger chunks called partitions, while clustering keys optimize sorting within these partitions. Clustering is crucial for query performance as it helps to filter and retrieve relevant data more efficiently. When records are scattered across micro-partitions (small clusters of rows), the optimizer has a harder time pruning unneeded parts of the table.

First, we need to identify key characteristics of our dataset that can be used as clustering keys. Commonly, these include date fields or categorical columns that are frequently part of WHERE conditions in queries. For instance, if you have a large fact table with daily logs and often query data by specific dates, using the date column as your clustering key could significantly improve performance.

To measure pruning improvement, we can use Snowflake’s query plan to analyze how well our chosen key is working. We'll compare the initial query performance before and after changing the clustering key, focusing on metrics like Rows Scanned, Bytes Returned, and Executions in the explain plan output.

**Code:**

```sql
-- Example Snowflake SQL to check query plan with a clustering key
EXPLAIN SELECT * FROM large_fact_table WHERE date_column = '2023-10-01' AND other_column = 'value';
```

**Example input / output:**

```text
-- Initial Explain Plan Output
Plan hash: 7654321

-----------------------------------------------------------------------------------------
| Id | Operation                   | Name           | Rows   | Bytes  | Cost (%CPU)| Time     |
-----------------------------------------------------------------------------------------
| 0  | SELECT STATEMENT            |                | 5000   | 125000 |    5000 (10)| 00:00:63 |
| 1  |  TABLE ACCESS FULL          | large_fact_table | 5000   | 125000 |    5000 (10)| 00:00:63 |
-----------------------------------------------------------------------------------------

-- After Clustering Key Change
Plan hash: 7654321

-----------------------------------------------------------------------------------------
| Id | Operation                   | Name           | Rows   | Bytes  | Cost (%CPU)| Time     |
-----------------------------------------------------------------------------------------
| 0  | SELECT STATEMENT            |                | 500   | 12500  |    500 (10)| 00:00:63 |
| 1  |  PARTITION RANGE SINGLE     |                | 500   | 12500  |    500 (10)| 00:00:63 |
|* 2 |   TABLE ACCESS BY LOCAL INDEX ROWID| large_fact_table |      1 |       |         |          |
-----------------------------------------------------------------------------------------
Predicate Information (identified by operation id):
---------------------------------------------------

   2 - filter("date_column"='2023-10-01' AND "other_column"='value')
```

**Flow:**

1. Yes
2. No
3. Identify Key Characteristics
4. Is the key used in WHEREconditions?
5. Use as Clustering Key
6. Consider alternative keys orcomposite keys
7. Migrate to new clustering keyand re-run queries
8. Evaluate pruning improvementwith explain plan

