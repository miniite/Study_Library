---
type: "practice"
difficulty: "Basic"
---

# As a data engineer, when a staging table is loaded fresh every night and then read once by a transform, is it worth indexing it? Explain your reasoning.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Index Tuning in Practice](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index_Tuning_in_Practice/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a staging table that gets freshly loaded each night and is only read once during transformation, indexing may not be as beneficial compared to other scenarios. In this context, let's break down why indexing might or might not be worth the effort.

#### Triaging the Question

The core of the question revolves around whether it’s worthwhile to create an index on a staging table that gets fully refreshed daily and is read by a single process only once. The interviewer is likely probing into understanding your knowledge of when indexes are useful, how they affect write performance, and their overall cost-benefit in this specific scenario.

#### Assumptions
Table Size: The staging table is large enough to warrant consideration for indexing.
Read Operation: The read operation is a one-time, full-table scan performed by the transform process.
Index Maintenance Overhead: There’s no ongoing maintenance needed on this index since the table is fully refreshed daily.

#### First-Principles Solution
1. Write Performance Impact
Insert/Update Deletion Operations: When a staging table is fully refreshed, any existing data will be deleted and reinserted. Indexes can slow down these operations because they need to be rebuilt or updated for each new entry.
2. Read Operation Efficiency
Full-Table Scan vs. Indexed Scan: If the read operation involves scanning the entire table (which is common with full refreshes), an index might not provide much benefit since it would still require a large amount of I/O to access all rows.
Query Patterns: For a one-time, full-table scan, an index may slow down data loading because the database needs to maintain and update the index during this process. However, if there are specific queries that can leverage an index for certain filters or ranges, it might be worth considering.
3. Cost-Benefit Analysis
Insert Performance: Assuming a significant number of rows being inserted every night, the cost of maintaining an index (both in terms of space and write performance) may outweigh its benefits.
Read Frequency: Since the table is only read once daily, any optimization gains from indexing might not be substantial compared to the overhead during writes.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In conclusion, for a staging table that gets fully refreshed every night and is read by a single process once, it might not be worth creating an index. The overhead of maintaining an index could outweigh its benefits in terms of write performance. However, if there are specific queries within the transform job that can benefit from indexing, then adding indexes to those columns could improve overall performance.

#### Common Mistakes
Over-indexing: Creating too many indexes on a table can significantly impact write performance and increase storage costs.
Underestimating Write Performance Impact: Ignoring the fact that full-refresh operations can be slower with an index in place due to index maintenance overhead.

#### Follow-Up
Questions
If there were specific columns frequently queried, would you consider adding indexes?
How might your answer change if the transform process was more complex and ran multiple times a day instead of once?

**Code:**

```sql
-- Example SQL for creating an index (hypothetical scenario)
CREATE INDEX idx_stage_column ON staging_table(column_name);
```

