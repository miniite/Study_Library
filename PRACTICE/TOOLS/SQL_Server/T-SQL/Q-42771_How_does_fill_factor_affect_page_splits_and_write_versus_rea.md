---
type: "practice"
difficulty: "Deep"
---

# How does fill factor affect page splits and write versus read performance, and how would you tune it for a high-insert table?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Filtered Indexes and Plan Cache](<../../../../TOOLS/SQL_Server/T-SQL/Filtered_Indexes_and_Plan_Cache.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: Fill factor is a crucial T-SQL parameter that can significantly impact write (insert) and read performance on SQL Server tables. By understanding its role in page splits, you can tune it effectively for high-insert environments.

When using T-SQL to develop database schemas, particularly for tables with frequent insert operations, the fill factor plays a critical role in managing page splits and overall table performance. Essentially, the fill factor is a percentage that determines how much space to reserve at the end of each data page when inserting rows. This parameter helps balance write (insert) performance against read performance by controlling the likelihood of row overflow and minimizing page splits.

#### Triage

The question is asking for an in-depth explanation on:

How fill factor affects page splits: Understanding what happens to a page when it fills up, and how this impacts insert operations.
Impact on write (insert) performance: Considering how different fill factors influence the number of writes required due to row overflow or page splits.
Impact on read performance: Explaining how the fill factor influences query execution plans and overall data retrieval efficiency.
First Principles

Let's break this down step by step:

Page Structure in SQL Server:

A data page is a fundamental unit of storage in SQL Server, with each page holding up to 8060 bytes (or rows) of data.
When inserting a row into a full page, the row must be split between two pages if it exceeds the current available space.

Fill Factor and Row Overflow:

A fill factor of 100% means that all the usable space on each page is used for storing rows.
Lowering the fill factor (e.g., to 90%) allows some unused space at the end of each page, reducing the need for immediate row splits.

Page Splits and Their Impact:

When a page split occurs, it involves copying half of the existing data to a new page and placing the new row on either the original or the new page.
This process can be resource-intensive, particularly during peak insert times, as it requires both read (to copy data) and write operations.

Write Performance:

A higher fill factor (closer to 100%) will result in fewer row splits but more frequent writes when rows exceed the available space.
Conversely, a lower fill factor leads to more efficient writes during insertions but increases the likelihood of page splits, which can degrade performance.

Read Performance:

Page splits do not directly impact read operations; however, they can affect how SQL Server chooses its execution plan.
If there are many page splits in an index, it may lead to more seeks and less use of indexes, impacting overall query performance.
Tuning for High-Insert Tables

For a high-insert table:

Initial Considerations:

Start with a fill factor that balances write efficiency and minimizes the frequency of row splits.
For tables with frequent inserts, consider starting with a lower fill factor (e.g., 90% or even less) to reduce the risk of page splits.

Monitoring Performance:

Regularly monitor statistics such as sys.dm_db_index_physical_stats and sys.dm_db_index_operational_stats to assess performance.
Look for signs like high insert times, frequent page splits, and degraded query plans.

Adjusting Fill Factor:

If row splits are observed frequently, consider lowering the fill factor further.
Conversely, if writes are becoming a bottleneck due to too much space reserved on each page, you might increase the fill factor slightly.

Other Considerations:

Ensure proper indexing strategies are in place to support high-insert scenarios.
Regularly rebuild indexes or perform online index operations to maintain optimal performance.

#### Closing Summary

In summary, the fill factor is a critical parameter for optimizing write and read performance on SQL Server tables. A lower fill factor can reduce page splits during inserts but may lead to more frequent writes, while a higher fill factor minimizes row splits but risks frequent writes. Effective tuning requires balancing these factors based on specific workload characteristics.

#### Common Mistakes
Overly Low Fill Factor: Using very low values (e.g., 50%) can lead to excessive page splits and write overhead.
Static Tuning: Not monitoring performance over time and adjusting the fill factor as needed can lead to suboptimal settings.
Ignoring Index Maintenance: Focusing solely on fill factor without considering proper indexing strategies.

**Comparison:**

| Column   | Data Type |
|----------|-----------|
| id       | INT       |
| name     | VARCHAR(50) |
| created_at | DATETIME  |
| Indexes |
|---------|
| idx_high_insert (FILLFACTOR = 90, ON column1, column2) |

**Code:**

```sql
-- Example of setting a fill factor for an index creation
CREATE NONCLUSTERED INDEX idx_high_insert ON table_name(column1, column2)
WITH (FILLFACTOR = 90);
```

**Example input / output:**

```

Input:

SQL
Copy
INSERT INTO high_insert_table (column1, column2) VALUES ('value1', 'value2');

Output:

With a fill factor of 90%, the insert operation may or may not result in a page split depending on the existing data distribution.
If the insert causes a page split, it will involve copying half of the data and writing the new row.
```

**Data:**

```json
{
    "table_name": {
        "columns": ["id", "name", "created_at"],
        "index": {
            "idx_high_insert": ["fill_factor": 90]
        }
    }
}
```

**Flow:**

1. Yes
2. No
3. Insert Operation
4. Is Page Full?
5. Page Split
6. Write New Row to New Page
7. Write New Row to ExistingPage
8. Maintain Index

