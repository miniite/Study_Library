---
id: "01_Databases_and_SQL/Query_Performance/Indexing/Composite_Indexes_and_Column_Order"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Query_Performance/Indexing"
tools: []
---

# Composite Indexes and Column Order

**L4 Advanced** · [↑ Indexing](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A composite index is one B-tree sorted by several columns *in order*, so it can only be searched efficiently from its leftmost column rightwards — which makes column order the whole design decision.

### 1. The big picture
A single-column index answers "find rows where `a = ?`". Real queries filter on several columns at once (`customer_id = ? AND order_date >= ?`), sort (`ORDER BY order_date DESC`) and join. A **composite (multi-column, compound) index** stores entries sorted by the first column, then — within equal first values — by the second, and so on, exactly like a phone book sorted by last name then first name.

That ordering is a physical property, not a hint. It determines which predicates can be turned into a **seek** (jump to a position in the tree and read a contiguous range), which become mere **residual filters** applied while scanning, and whether the index can deliver rows already sorted so no sort step is needed.

This lesson sits under *Indexing* as the "design" step: after you know how a B-tree works (see the B-tree and index-types lessons) and before covering indexes (`INCLUDE`), partial indexes and governance at scale.

### 2. Core concepts and vocabulary
- **Key column** — a column that participates in the index ordering. Order of key columns is significant.
- **Leftmost prefix** — the first *k* key columns of the index. The index can be used for seeks on any leftmost prefix: `(a)`, `(a,b)`, `(a,b,c)`.
- **Seek predicate** — a condition the engine uses to navigate the tree (equality on leading columns, one range on the next column).
- **Residual predicate** — a condition evaluated on each index entry after the seek range has been located; it filters but doesn't narrow the range read.
- **Selectivity** — fraction of rows a predicate matches (low fraction = highly selective). **Cardinality** — number of distinct values.
- **Sargable** — "search-argument-able": a predicate form the engine can use against an index (column compared directly to a value, not wrapped in a function).
- **Index intersection** — combining two single-column indexes by intersecting their row-id sets (bitmap AND). Exists in PostgreSQL (bitmap scans) and MySQL (index merge); SQL Server usually prefers one index plus lookups.
- **Skip scan** — an optimization (Oracle, recent MySQL, SQLite; PostgreSQL 18) that lets a leading column with few distinct values be "skipped" so a trailing column can still be searched.

### 3. How it works, step by step
Take `CREATE INDEX IX_Orders_CustDate ON Orders (CustomerID, OrderDate);`. Logical leaf content, sorted:

| CustomerID | OrderDate | row locator |
|---|---|---|
| 101 | 2025-01-03 | … |
| 101 | 2025-02-11 | … |
| 101 | 2025-03-20 | … |
| 102 | 2025-01-05 | … |
| 102 | 2025-04-02 | … |

1. **`WHERE CustomerID = 101 AND OrderDate >= '2025-02-01'`** — the engine descends the tree to the first entry `(101, 2025-02-01)` and reads forward until `CustomerID` changes. Both predicates are seek predicates; the range read is exactly the matching entries.
2. **`WHERE CustomerID = 101`** — still a seek on the leftmost prefix; reads all of customer 101's entries (a contiguous block).
3. **`WHERE OrderDate >= '2025-02-01'`** — the matching entries are scattered across every customer's block. The engine can't build a single contiguous range, so it must scan the whole index (or whole table) and filter. The index is "used" only as a narrower copy of the data, not for navigation.
4. **`WHERE CustomerID = 101 ORDER BY OrderDate`** — after the seek, entries are already in `OrderDate` order, so the sort disappears. This works because the leading column is pinned by *equality*.
5. **`WHERE CustomerID > 100 ORDER BY OrderDate`** — a *range* on the first column breaks the ordering of the second column across customers, so a sort is needed again.

The core rule the books state: an index "does not need to use all the columns… it does, however, need to start with the leftmost column and use the columns from left to right, in order."

### 4. Worked example
```sql
-- SQL Server / PostgreSQL-flavoured demo
CREATE TABLE dbo.MythFour (
    SalesOrderID int PRIMARY KEY,
    OrderDate    date NOT NULL,
    DueDate      date NOT NULL,
    ShipDate     date NULL
);
CREATE INDEX IX_MythFour ON dbo.MythFour (OrderDate, DueDate, ShipDate);

-- A: leftmost column            -> Index Seek
SELECT OrderDate FROM dbo.MythFour WHERE OrderDate = '2013-07-01';

-- B: leftmost + second column   -> Index Seek (both predicates seek)
SELECT OrderDate FROM dbo.MythFour
WHERE OrderDate = '2013-07-01' AND DueDate = '2013-07-13';

-- C: only the 2nd column        -> Index Scan (whole index read, then filtered)
SELECT OrderDate FROM dbo.MythFour WHERE DueDate = '2013-07-13';

-- D: only the 3rd column        -> Index Scan
SELECT OrderDate FROM dbo.MythFour WHERE ShipDate = '2013-07-08';
```
Query A and B show a Seek in the plan; C and D show a Scan, with the predicate listed under *Predicate* (residual) rather than *Seek Predicates*. Compare `SET STATISTICS IO ON` reads: A/B read a handful of pages, C/D read every leaf page of the index. The myth being busted: "any column of a multi-column index can be filtered efficiently".

### 5. Under the hood
- **Index key as a concatenated tuple.** Internally each entry is compared lexicographically: compare column 1; only if equal compare column 2, etc. Navigation in the upper (non-leaf) pages uses the same comparison, which is why only a prefix can be turned into a descent target.
- **One range column, then residual.** After the first non-equality column (range, `LIKE 'abc%'`, `IN` treated as multiple seeks in some engines), later key columns can no longer narrow the contiguous range; they act as filters on entries inside the range. Hence the rule of thumb **Equality, Sort, Range** (ESR in MongoDB's terminology): equality columns first, then the sort column, then range columns.
- **Selectivity ordering is a myth in isolation.** Advice such as "put the most selective column first" is only a tie-breaker. If two columns are both used with equality, either order gives the same seek; the better choice is the order that also serves other queries (leftmost prefix reuse) or `ORDER BY`. A highly selective column that is only sometimes filtered should not lead if every query filters on a different column first.
- **Statistics are per leading column (histogram) plus density for prefixes.** SQL Server stores a histogram only on the first key column and density vectors for prefixes; correlated columns (`city`, `pincode`) therefore get under- or over-estimated combined selectivity because the optimizer assumes independence. Remedies: multi-column statistics / extended statistics (`CREATE STATISTICS … (city, pincode)` in PostgreSQL; filtered statistics in SQL Server), or a composite index which gives prefix density.
- **Skip scan.** Where supported, the engine enumerates distinct values of leading columns and does a seek for each; it pays off only if the leading column has few distinct values.
- **Columnar lakes behave the same way.** Composite sort keys in files are lexicographic: queries must constrain the leading sort columns, otherwise most blocks still have to be read. The index tree and the sorted file are the same idea.

### 6. Performance and design implications
- **Writes pay for every index.** Each insert/update/delete touches each index that contains an affected column. A five-column index on a hot table roughly adds one extra B-tree insert plus wider log records.
- **Width matters.** Wider keys → fewer entries per page → deeper tree, more cache used, slower scans. Keep key columns narrow and push non-searched columns to `INCLUDE`.
- **A composite index replaces its prefix.** `(a,b)` makes a separate `(a)` index redundant (except for uniqueness or if a narrower index is much cheaper to scan). Find these with index usage DMVs / `pg_stat_user_indexes`.
- **Combine with ORDER BY/LIMIT.** `WHERE tenant_id = ? ORDER BY created_at DESC LIMIT 20` is best served by `(tenant_id, created_at DESC)`: seek, read 20 entries, stop.
- **Measure.** Logical reads (`SET STATISTICS IO`), plan operators (Seek vs Scan, Sort present?), `EXPLAIN (ANALYZE, BUFFERS)`, estimated vs actual rows.
- **Low-cardinality booleans.** Indexing `is_active` alone rarely helps: half the table qualifies. As a *trailing* column or as a **partial/filtered** index predicate (`WHERE is_active`), it can be valuable.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE INDEX ix ON t (a, b DESC)` | Composite index with per-column direction | Matching mixed `ORDER BY a, b DESC` |
| `INCLUDE (c, d)` | Stores extra non-key columns in leaf only | Cover a query without widening the key |
| `CREATE STATISTICS s (a,b) ON t` / `CREATE STATISTICS … (dependencies)` (PG) | Multi-column statistics | Correlated columns misestimated |
| `SET STATISTICS IO, TIME ON` / `EXPLAIN (ANALYZE, BUFFERS)` | Measure reads and time | Before/after comparison |
| `sys.dm_db_index_usage_stats`, `pg_stat_user_indexes` | Index usage counters | Finding unused / redundant indexes |
| `sys.dm_db_missing_index_details` | Optimizer's missing-index suggestions | Starting point only; never apply blindly |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Queries repeatedly filter the same 2–3 columns together | Each query filters a different single column (use separate indexes or index intersection) |
| Equality columns can lead, range/sort column follows | The only distinguishing predicate is a function/expression on the column (see functional indexes) |
| You can reuse the index as a prefix for other queries | Index would have >4–5 key columns "just in case" — write cost and size explode |
| A query's `ORDER BY` can be satisfied by index order | Two teams require conflicting column orders: consider two indexes, or a read model |

### 9. Common mistakes
- **Ordering columns by selectivity alone** → index useless for queries that omit the "most selective" column → design from the query predicates (equality first, then sort, then range).
- **Expecting a trailing-column-only filter to seek** → full index scan → add an index with that column leading, or restructure.
- **Adding a column to an existing index and "breaking" plans** → key width grew, density estimates changed, or the new column changed which prefix serves the query; compare plans before/after with a production-sized copy.
- **Leaving redundant prefixes** → double write cost → drop `(a)` when `(a,b)` exists after verifying no uniqueness role.
- **Ignoring correlated columns** → bad join/scan choice because estimated rows are far too low → create multi-column statistics.
- **Relying on ORM column order** — the order of predicates in `WHERE` doesn't matter; the order of *index key columns* does.

### 10. 🎯 Interview kit
**30-second answer:** A composite index is sorted by its columns left to right, so it supports seeks only on a leftmost prefix: equality columns first, then the sort column, then one range column. A filter on a non-leading column alone can't seek and degrades to a scan.

**Structure a longer answer:** clarify the queries (which predicates, which sort, how many rows returned) → explain the sorted-tuple mechanism and the leftmost-prefix rule → give the equality/sort/range ordering rule and `WHERE a=? AND b>? AND c=?` as an example (`(a, c, b)`) → trade-offs (write cost, width, redundancy) → verify with a plan and read counts.

**Likely follow-ups:**
- *`WHERE a=? AND b>? AND c=?` — what index?* → `(a, c, b)`: both equalities lead, the range goes last so `c` can still seek; `(a,b,c)` would apply `c` only as a residual filter.
- *Can `(last_name, first_name)` serve `first_name` alone?* → Not as a seek; at best a full index scan (or skip scan if the engine supports it and `last_name` has few values).
- *Do two single indexes work as well as one composite?* → Engines may intersect them (bitmaps), but that costs two index reads plus a merge; a composite does one seek.
- *Is a boolean worth indexing?* → Rarely alone; use partial/filtered index or place it as a trailing column.
- *Why did the optimizer ignore my new index?* → Predicate not sargable, stale/skewed statistics, or the query returns too large a fraction so scan is cheaper.

**Signals interviewers look for:** you talk about the *query* first, not the columns; you know equality→sort→range; you mention write cost and redundancy; you verify with plans rather than guess.

### 11. Check yourself
- [ ] Explain, using a sorted list, why `(a,b)` can't seek on `b` alone.
- [ ] Design the index for `WHERE a=? AND b>? AND c=? ORDER BY d` and justify each position.
- [ ] Show in a plan the difference between a seek predicate and a residual predicate.
- [ ] Explain why selectivity-first ordering is a heuristic, not a rule.
- [ ] Describe how correlated columns break cardinality estimates and name two fixes.
<!-- explainer:end -->

## Practice questions

- [Basic] [Why can a WHERE clause with LIKE 'ORD-2026%' on an order_id use an index while LIKE '%2026' cannot?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20320_Why_can_a_WHERE_clause_with_LIKE_ORD-2026_on_an_order_id_use.md>)
- [Intermediate] [How would you decide whether to put status or created_at first in a composite index for WHERE status = 'open' ORDER BY created_at?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20011_How_would_you_decide_whether_to_put_status_or_created_at_fir.md>)
- [Intermediate] [Your composite index has an unused trailing column bloating it — how do you safely prove it's unused before dropping it?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-00549_Your_composite_index_has_an_unused_trailing_column_bloating_.md>)
- [Deep] [Why should equality columns generally come before range columns in a composite index used by an ETL filter query?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20364_Why_should_equality_columns_generally_come_before_range_colu.md>)
- [Deep] [How would you index for a query with both an equality filter and a range filter, like status='open' AND created_at > X?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19558_How_would_you_index_for_a_query_with_both_an_equality_filter.md>)
- [Deep] [You have a composite index on (a, b, c) — explain the leftmost-prefix rule and which of these predicates it can serve: a=?, b=?, a=? AND c=?.](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-00514_You_have_a_composite_index_on_a_b_c_explain_the_leftmost-pre.md>)
- [Deep] [How would you index for a query that filters on a=? and does a LIKE 'prefix%' on b — can one composite index serve both?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-00575_How_would_you_index_for_a_query_that_filters_on_a_and_does_a.md>)
- [Deep] [You inherit a query with three OR'd conditions across different columns. The planner does a full scan. Walk through converting it to a UNION or bitmap-OR-friendly form and the index set needed.](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19719_You_inherit_a_query_with_three_OR_d_conditions_across_differ.md>)

## Related concepts

- [Leftmost Prefix Rule](<../../SQL_Language/SQL_Fundamentals/Leftmost_Prefix_Rule.md>) *(SQL Fundamentals)*
- [Index Access Paths](<../../Database_Internals/DBMS_Concepts/Index_Access_Paths.md>) *(DBMS Concepts)*
- [MySQL Optimizer Behavior](<../../../TOOLS/MySQL/MySQL_Platform/MySQL_Optimizer_Behavior.md>) *(MySQL Platform)*
- [Query Rewrites and Plan Stability](<../Execution_Plans/Query_Rewrites_and_Plan_Stability.md>) *(Execution Plans)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Covering Joins with Composite Indexes
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Optimizing OR Predicates
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Columnstore Indexes
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Creating an Index
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Columnstore Index
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Index Physical Stats Summary › Categories
- *Designing Data-Intensive Applications* — Martin Kleppmann — Storage and Retrieval › Multi-column indexes
- *Designing Data-Intensive Applications* — Martin Kleppmann — Storage and Retrieval › Sort Order in Column Storage
- *Designing Data-Intensive Applications* — Martin Kleppmann — I
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Storage Design Patterns › Composite sort keys
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — is to create a hotels table in your Cassandra data model using a similar key design. Over time, the
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Ordered Columnstore Indexes › Inserting Data into an Ordered Columnstore Index

