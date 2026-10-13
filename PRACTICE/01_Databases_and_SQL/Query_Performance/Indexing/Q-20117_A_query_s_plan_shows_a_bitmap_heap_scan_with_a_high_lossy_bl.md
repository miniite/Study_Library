---
type: "practice"
difficulty: "Expert"
---

# A query's plan shows a bitmap heap scan with a high 'lossy' block count. What does lossy mean here and how does work_mem relate to it?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Index-only Scans](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index-only_Scans.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with database indexing and query optimization, "lossy" in a bitmap heap scan context refers to the situation where not all rows that match the filter condition are included in the result set. Essentially, the term indicates that some of the potential matches have been filtered out during the execution plan.

To understand how work_mem relates to this, we need to delve into the details of query optimization and memory management in databases. The bitmap heap scan is a technique used by PostgreSQL, for example, where it scans a table (heap) based on an index or expression and uses bitmaps to efficiently filter rows. However, if the lossy count is high, it means that some of these filters are not perfectly accurate, leading to redundant or missing data.

#### Triage

The question is asking us to interpret what "lossy" means in a specific query execution plan context and how the memory setting work_mem influences this behavior. Here are our key assumptions:

The database being used supports bitmap heap scans.
We are dealing with a PostgreSQL environment, as it commonly uses this technique.

#### First-Principles Solution

Understanding Lossy Bitmap Heap Scan:

In a bitmap heap scan, the query planner first applies filters on indexes or expressions to generate bitmaps representing which rows match certain conditions.
If a row matches multiple filter conditions (overlapping bitmaps), each condition may produce its own bitmap.
The lossy count indicates how many of these bitmaps have overlapping regions, leading to redundant processing.

Role of work_mem:

work_mem is the maximum amount of memory a query can use for internal temporary operations before writing data to disk.
In the context of bitmap heap scans, increasing work_mem allows more efficient bitmap creation and storage in memory, potentially reducing the number of bitmaps needed and thus lowering the lossy count.

Steps to Optimize:

Analyze the query plan to identify which filter conditions are causing high lossiness.
Adjust work_mem if possible to allow more efficient bitmap handling.

#### Cons
ider rewriting queries or indexes to reduce overlapping filters, thereby minimizing lossy scans.
Evaluate the trade-offs between increasing memory usage and I/O operations.

#### Closing Summary

In summary, "lossy" in a bitmap heap scan context refers to redundant processing due to overlapping filter conditions. The work_mem parameter plays a crucial role by influencing how efficiently bitmaps are created and stored, potentially reducing lossiness. However, adjusting work_mem should be balanced against the overall system memory usage and I/O operations.

#### Common Mistakes/Follow-Ups
Failing to understand that not all rows matching filters are included in the result set.
Not considering other optimization techniques such as rewriting queries or reindexing.
Ignoring the trade-offs between increasing work_mem and potential I/O overhead.

**Comparison:**

| id | col1 | col2 |
|----|------|------|
| 1  | 9    | 18   |
| 2  | 11   | 17   |
| 3  | 10   | 21   |
| 4  | 15   | 16   |

**Code:**

```sql
-- Example query showing a bitmap heap scan with lossy bitmaps
EXPLAIN ANALYZE SELECT * FROM large_table WHERE col1 > 10 AND col2 < 20;
```

**Example input / output:**

```text
Input:
EXPLAIN ANALYZE SELECT * FROM large_table WHERE col1 > 10 AND col2 < 20;

Output:
QUERY PLAN
Bitmap Heap Scan on large_table  (cost=100.00..5000.00 rows=8000 width=32)
  Recheck Cond: ((col1 > 10) AND (col2 < 20))
  ->  BitmapAnd  (cost=100.00..100.00 rows=8000)
        ->  Bitmap Index Scan on idx_col1  (cost=0.00..50.00 rows=4000)  (lossy)
        ->  Bitmap Index Scan on idx_col2  (cost=0.00..50.00 rows=4000)  (lossy)
```

**Data:**

```json
{
  "query_plan": {
    "plan_type": "BitmapHeapScan",
    "lossy_bitmaps": [
      { "bitmap_filter": "col1 > 10", "count": 5000 },
      { "bitmap_filter": "col2 < 20", "count": 4000, "overlap_with": ["col1 > 10"] }
    ]
  }
}
```

**Flow:**

1. Identify Lossy Bitmaps
2. Analyze Query Plan
3. Adjust work_mem
4. Re-examine Indexes & Queries
5. Tune Database Settings forEfficiency

