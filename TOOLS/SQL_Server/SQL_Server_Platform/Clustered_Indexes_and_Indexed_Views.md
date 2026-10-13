---
id: "TOOLS/SQL_Server/SQL_Server_Platform/Clustered_Indexes_and_Indexed_Views"
level: 4
type: "concept"
topic: "TOOLS/SQL_Server/SQL_Server_Platform"
tools: ["SQL_Server"]
---

# Clustered Indexes and Indexed Views

**L4 Advanced** · [↑ SQL Server Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A clustered index *is* the table — its leaf level holds the data rows in key order — and an indexed view is a view whose result has been materialized by giving it a unique clustered index.

### 1. The big picture
Every SQL Server table is either a **heap** (unordered pages) or a **clustered table** (a B-tree whose leaf pages are the data pages). Choosing the clustered key therefore decides how rows are physically organized, how inserts behave, and what every nonclustered index carries as its row locator.

An **indexed view** extends the same idea: a view is normally just a stored query, but once you create a unique clustered index on it, SQL Server stores the result set and keeps it synchronized on every change to the base tables. This pre-computes expensive joins and aggregates, at the cost of write overhead.

### 2. Core concepts and vocabulary
- **Clustered index** — B-tree whose leaf level is the table's data rows; only one per table.
- **Heap** — a table with no clustered index; rows located by RID (file:page:slot).
- **Clustering key** — the column(s) ordering the data; copied into every nonclustered index as the row locator.
- **Nonclustered index** — separate B-tree of key columns plus locator (clustering key or RID).
- **Key lookup** — fetching extra columns from the clustered index for a row found in a nonclustered index.
- **Page split** — a full page divides in two when a row must be inserted in the middle; causes fragmentation and extra logging.
- **Fill factor** — percent of each leaf page filled at build time, leaving room for growth.
- **Indexed view (materialized view)** — a schema-bound view with a unique clustered index.
- **`SCHEMABINDING`** — ties the view to base-table definitions so they can't change underneath it.
- **`NOEXPAND`** — hint forcing the optimizer to use the view's stored index.

### 3. How it works, step by step
**Clustered index**
1. `CREATE CLUSTERED INDEX` (or a `PRIMARY KEY`, which is clustered by default) sorts the rows and builds a B-tree above them.
2. A seek navigates root → intermediate → leaf, typically 3–4 page reads even for hundreds of millions of rows.
3. Inserts go to the leaf page that owns the key range; if it is full, a split occurs.
4. Nonclustered indexes store the clustering key; a lookup is a seek into the NC index then a seek into the clustered index.

**Indexed view**
1. Create the view `WITH SCHEMABINDING`, using two-part names and allowed constructs only.
2. `CREATE UNIQUE CLUSTERED INDEX` on it materializes the rows.
3. On any DML against the base tables, the engine updates the view's rows as part of the same transaction.
4. Queries may be rewritten to read the view's index even when they reference only base tables (automatic matching, Enterprise/Developer editions); in other editions use `WITH (NOEXPAND)`.

### 4. Worked example
```sql
CREATE TABLE dbo.Orders (
    OrderID   INT IDENTITY PRIMARY KEY CLUSTERED,   -- narrow, ever-increasing
    CustomerID INT NOT NULL,
    OrderDate DATE NOT NULL,
    Amount    DECIMAL(12,2) NOT NULL
);
GO
CREATE VIEW dbo.vCustomerTotals WITH SCHEMABINDING AS
SELECT CustomerID,
       SUM(Amount)   AS TotalAmount,   -- Amount is NOT NULL (required for SUM)
       COUNT_BIG(*)  AS Cnt            -- required with GROUP BY
FROM dbo.Orders
GROUP BY CustomerID;
GO
CREATE UNIQUE CLUSTERED INDEX IX_vCustomerTotals ON dbo.vCustomerTotals (CustomerID);

SELECT CustomerID, TotalAmount FROM dbo.vCustomerTotals WITH (NOEXPAND)
WHERE CustomerID = 42;
```
The final query is a one-row seek on the view's index instead of an aggregate over `Orders`. Every insert into `Orders` now also updates one row in the view; check the plan of an `INSERT` to see the extra operator maintaining it.

### 5. Under the hood
- Clustered leaf pages are linked in a doubly linked list so range scans read sequentially; intermediate pages hold key + child page pointer.
- If the clustered index isn't unique, SQL Server adds a hidden 4-byte *uniquifier* to duplicate keys, widening every NC index.
- Row locators: heap → 8-byte RID; clustered table → clustering key. A wide clustering key (e.g. a `uniqueidentifier` or composite of strings) inflates every nonclustered index and the buffer pool.
- Random keys (`NEWID()`) land on arbitrary pages, so splits and ~70% page density are common; `NEWSEQUENTIALID()` or an identity avoids this.
- Indexed-view maintenance is synchronous; heavy-write base tables can serialize on view rows (a single aggregate row becomes a hotspot).
- Restrictions: no `OUTER JOIN`, subqueries, `DISTINCT`, `MIN/MAX`, `COUNT(*)` (use `COUNT_BIG`), non-deterministic functions, or `UNION`.

### 6. Performance and design implications
- Ideal clustering key: **narrow, unique, static, ever-increasing** (identity/date+id). Satisfy range queries that always read together (e.g. `OrderDate`) only when that also meets the rest.
- Heaps suit bulk-loaded staging tables; forwarded records hurt scans.
- Indexed views shine for read-heavy aggregates (reporting on OLTP, DW summary tables); poor for write-heavy base tables.
- Measure: `sys.dm_db_index_physical_stats` (fragmentation, page density), `sys.dm_db_index_usage_stats` (reads vs updates), `SET STATISTICS IO`.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE CLUSTERED INDEX` | Orders the table | Choosing storage order |
| `PRIMARY KEY NONCLUSTERED` | PK without clustering | Cluster on another key |
| `FILLFACTOR = n` | Leaves free space in leaf pages | Split-prone indexes |
| `WITH SCHEMABINDING` | Locks view to base schema | Required for indexed view |
| `WITH (NOEXPAND)` | Forces use of view index | Non-Enterprise editions, predictability |
| `COUNT_BIG(*)` | Row count in aggregated view | Mandatory in grouped indexed views |
| `sys.dm_db_index_physical_stats` | Fragmentation / density | Maintenance decisions |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Clustered on identity/sequence | Random GUID clustered key under heavy inserts |
| Clustered on a business range column | That column is wide or frequently updated |
| Heap | Frequent updates growing rows, or point lookups needed |
| Indexed view | Base tables are write-hot or aggregates need unsupported functions |

### 9. Common mistakes
- Clustering on a wide or volatile key → huge NC indexes and row movement → pick a surrogate narrow key.
- Assuming the PK must be the clustered index → missed range-query opportunity → decide explicitly.
- Rebuilding indexes on a schedule without checking need → log growth, no gain → use density/fragmentation thresholds.
- Creating an indexed view and wondering why it is ignored → edition or hints → use `NOEXPAND`.
- Forgetting that DML on base tables pays the view maintenance cost → slowdowns after "optimizing" reads.

### 10. 🎯 Interview kit
**30-second answer:** The clustered index defines the physical row order and is the table itself; one per table. Nonclustered indexes point back via the clustering key, so a narrow, unique, increasing key is best. An indexed view materializes a query by adding a unique clustered index.

**Structure a longer answer:** clarify (OLTP vs reporting) → mechanism (B-tree, locator, splits) → trade-offs → scenario (orders table with identity key and an aggregate view).

**Likely follow-ups:**
- *Why only one clustered index?* → rows can be stored in one order.
- *Why avoid GUID clustered keys?* → random inserts, splits, wide NC indexes.
- *Heap vs clustered?* → RID locator, forwarded records, no ordering.
- *Why `COUNT_BIG`?* → required so the engine can maintain deletes of aggregated groups.

**Signals interviewers look for:** the lookup/locator relationship, awareness of write cost, not blindly creating indexes.

### 11. Check yourself
- [ ] Explain how a nonclustered seek finds the data row on a clustered table versus a heap.
- [ ] Choose a clustering key for a high-insert table and defend it against a GUID.
- [ ] Create an indexed view with a grouped aggregate and prove via the plan that it is used.
- [ ] Describe what an insert into a base table does to the indexed view.
<!-- explainer:end -->

## Implementations (tools)

- [SQL Server](<../_index.md>)

## Practice questions

- [Basic] [What is an index in SQL Server and how does it speed up queries?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41485_What_is_an_index_in_SQL_Server_and_how_does_it_speed_up_quer.md>)
- [Intermediate] [When would a GUID clustered key cause problems, and how do NEWSEQUENTIALID or a surrogate int key address them?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41363_When_would_a_GUID_clustered_key_cause_problems_and_how_do_NE.md>)
- [Deep] [How do filtered indexes work, and give a production scenario where a filtered index dramatically reduced index size and improved a query.](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41360_How_do_filtered_indexes_work_and_give_a_production_scenario_.md>)
- [Deep] [How would you design an online index rebuild to avoid blocking a 24x7 OLTP table, and what are the limitations of ONLINE = ON?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41388_How_would_you_design_an_online_index_rebuild_to_avoid_blocki.md>)
- [Deep] [What is Query Store, and how would you use it to detect and force a good plan after a regression?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41414_What_is_Query_Store_and_how_would_you_use_it_to_detect_and_f.md>)
- [Deep] [How would you index and query a JSON column efficiently in SQL Server, and what are the limits versus the native json type in newer versions?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41435_How_would_you_index_and_query_a_JSON_column_efficiently_in_S.md>)
- [Deep] [Explain the difference between OLTP and OLAP indexing/design decisions, and how you would separate reporting load from a busy transactional database.](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41458_Explain_the_difference_between_OLTP_and_OLAP_indexing_design.md>)
- [Expert] [You see a nested loop join over a large outer input causing millions of seeks. How would you push the optimizer toward a hash join, and when is that the wrong fix?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41379_You_see_a_nested_loop_join_over_a_large_outer_input_causing_.md>)

## Related concepts

- [Filtered Indexes and Plan Cache](<../T-SQL/Filtered_Indexes_and_Plan_Cache.md>) *(T-SQL)*
- [Stored Procedure Performance and Design](<../../../01_Databases_and_SQL/Database_Objects/Stored_Procedures/Stored_Procedure_Performance_and_Design.md>) *(Stored Procedures)*
- [Index Access Paths](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Index_Access_Paths.md>) *(DBMS Concepts)*
- [Large Batch Updates](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Large_Batch_Updates.md>) *(SQL Fundamentals)*

## References

- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 3 Methods For Capturing Query Performance Metrics
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 4 Analyzing Query Behavior Using Execution Plans
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 5 Statistics, Data Distribution, And Cardinality
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 6 Using The Query Store For Query Performance And Execution Plans
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Performance Considerations
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › Best Practices for Using PIVOT
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › Performance Considerations with XML
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 7: Dynamic SQL, Triggers, and Table-Valued Functions (TVFs) › Benefits of spexecutesql
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — What are Columnstore Indexes RE COLUMNSTORE INDEXES
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — What are Columnstore Indexes RE COLUMNSTORE INDEXES › Introducing Columnstore Indexes
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — What are Columnstore Indexes RE COLUMNSTORE INDEXES › Benefits of Columnstore Indexes in SQL Server
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — What are Columnstore Indexes RE COLUMNSTORE INDEXES › Faster Analytic Reads
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Exploring Database Concepts › Service vs. Application
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Thinking About Performance
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Grouping and Summarizing Data › Thinking About Performance
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Manipulating Data › Thinking About Performance
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Data ��difi�ati�n
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Extents
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Columnstore Structure › Examining Pages
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — DBCC EXTENTINFO
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Row Data Print Option › Page Fragmentation
- *SQL Server Tutorial* (YouTube series) — Part 61: Creating a large table with random data for performance testing ([watch](https://www.youtube.com/watch?v=RizVYigF4GI))
- *SQL Server Tutorial* (YouTube series) — Part 143: Sql server query plan cache ([watch](https://www.youtube.com/watch?v=tX8n51IeV3E))

