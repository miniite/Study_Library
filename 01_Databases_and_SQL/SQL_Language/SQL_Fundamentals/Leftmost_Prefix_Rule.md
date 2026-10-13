---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Leftmost_Prefix_Rule"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: ["MySQL", "PostgreSQL"]
---

# Leftmost Prefix Rule

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A composite B-tree index is sorted by its first column, then the second within it, and so on — so it can only seek efficiently on a *leftmost prefix* of its columns, and the order of columns decides which queries it can serve.

### 1. The big picture
An index on `(a, b, c)` is not three indexes; it's one sorted list, like a phone book sorted by last name, then first name, then city. You can quickly find "Smith", or "Smith, John", but not "everyone named John" — that requires scanning the whole book. This is the **leftmost prefix rule**, and it governs which queries use an index to *seek* (jump to a range) versus *scan* (read everything).

Understanding it lets you design one composite index that serves several queries, order columns sensibly, and recognize why an "obviously indexed" query is still slow — including predicates wrapped in functions (non-*sargable*).

### 2. Core concepts and vocabulary
- **Composite (multi-column) index** — an index on two or more columns, ordered lexicographically.
- **Leftmost prefix** — the first *k* columns of the key; only prefixes can drive a seek.
- **Seek vs scan** — navigate the tree to a start key and read a contiguous range, vs read the whole index/table.
- **Sargable** — "Search ARGument ABLE": a predicate shaped so the engine can use an index seek (`col = @x`, `col > @x`, `col BETWEEN`, `col LIKE 'abc%'`).
- **Equality column** — compared with `=`; keeps the next column ordered within the matched range.
- **Range column** — compared with `<`, `>`, `BETWEEN`, `LIKE 'x%'`; columns after it can't narrow the seek further.
- **Selectivity** — fraction of rows a predicate keeps (lower fraction = more selective).
- **Covering index** — contains every column the query needs, so no lookup to the base table.
- **Key lookup / bookmark lookup** — fetching extra columns from the clustered index per index hit.

### 3. How it works, step by step
For index `IX(country, city, last_name)`:
1. Entries are ordered by `country`; ties by `city`; ties by `last_name`.
2. `WHERE country='IN'` → seek: all entries for `IN` are contiguous.
3. `WHERE country='IN' AND city='Pune'` → seek narrower.
4. `WHERE country='IN' AND last_name='Rao'` → seek on `country`, but `last_name` entries are scattered across cities, so it's applied as a **residual filter** while scanning the `IN` range.
5. `WHERE city='Pune'` (skips the leftmost column) → no prefix, so the engine must scan the entire index (or use a different index).
6. `WHERE country='IN' AND city > 'M' AND last_name='Rao'` → seek on `country` + range on `city`; `last_name` can't help the seek because `city` is a range.

**Rule of thumb for ordering columns:** equality columns first, then the one range/sort column, then columns merely needed for coverage (as `INCLUDE`). Among equality columns, put the most commonly used/selective-and-stable first, but weigh "can this index serve many queries via different prefixes?".

For `ORDER BY`: an index can deliver rows already sorted if, after equality predicates, the remaining index columns match the `ORDER BY` order. `WHERE country='IN' ORDER BY city, last_name` needs no sort with `IX(country, city, last_name)`.

### 4. Worked example
```sql
CREATE TABLE dbo.People (
  id int IDENTITY PRIMARY KEY, country char(2), city varchar(40),
  last_name varchar(40), created datetime2, notes nvarchar(200));

CREATE INDEX IX_People_c_c_l ON dbo.People (country, city, last_name);

-- 1. Seek on prefix (country, city)
SELECT id FROM dbo.People WHERE country = 'IN' AND city = 'Pune';

-- 2. Skips country: scan, not seek
SELECT id FROM dbo.People WHERE city = 'Pune';

-- 3. Sort satisfied by index order
SELECT TOP (20) last_name FROM dbo.People
WHERE country='IN' AND city='Pune' ORDER BY last_name;

-- 4. Non-sargable: function on the indexed column forces a scan
SELECT id FROM dbo.People WHERE UPPER(country) = 'IN';
SELECT id FROM dbo.People WHERE YEAR(created) = 2026;       -- if indexed on created
-- Sargable rewrite
SELECT id FROM dbo.People WHERE created >= '2026-01-01' AND created < '2027-01-01';
```
Check with the actual plan: query 1 shows *Index Seek* with **Seek Predicates** on country and city; query 2 shows *Index Scan* with a **Predicate** on city; query 3 shows no Sort operator. PostgreSQL's `EXPLAIN` shows `Index Cond` for the seek parts and `Filter` for residual ones; MySQL shows `key_len` — the number of key bytes used, which tells you how many prefix columns were used.

### 5. Under the hood
- A B-tree stores keys in sorted order across linked leaf pages; the root/intermediate pages hold separator keys. A seek descends O(log n) levels to the first qualifying leaf entry, then follows leaf links. Only a prefix gives a *contiguous* range; non-prefix predicates map to many scattered ranges.
- **Skip scan.** Some engines (Oracle's *index skip scan*, newer MySQL 8.0, PG 18) can probe each distinct value of a low-cardinality leading column. It works only when the leading column has few distinct values; don't design around it.
- **Statistics** are kept per leading column in detail (histogram) and for column *prefixes* as density; the optimizer estimates multi-column selectivity with density/correlation assumptions, which is why correlated columns (country/city) can be misestimated.
- **Included columns** (SQL Server `INCLUDE`, PG `INCLUDE`) live only at the leaf, are not part of ordering, and don't count toward the key-size limit — use them to cover without widening the seek key.
- **Hash indexes** have no ordering: only equality on all columns; they can't use prefixes, ranges or `ORDER BY`. **BRIN** indexes store min/max per block range and win on append-only, physically ordered data (time series) with tiny size.
- **Clustered vs nonclustered:** the clustered index *is* the table ordered by its key; nonclustered indexes store the clustered key (or a row id) as the pointer for lookups.

### 6. Performance and design implications
- One well-ordered composite often replaces two or three single-column indexes; fewer indexes also mean cheaper writes.
- Don't duplicate prefixes: `IX(a)` is redundant given `IX(a,b)` (except for tiny-key scans or uniqueness semantics).
- Equality-then-range-then-sort designs serve filter + `ORDER BY ... LIMIT` queries with early termination.
- Verify usage: SQL Server `sys.dm_db_index_usage_stats` (seeks/scans/lookups, `last_user_seek`), `sys.dm_db_missing_index_*` (hints only); PostgreSQL `pg_stat_user_indexes.idx_scan`; MySQL `sys.schema_unused_indexes`. Drop unused ones safely by making them invisible (MySQL/Oracle) or disabling (SQL Server) first.
- Types matter: implicit conversions (comparing `varchar` column to `nvarchar` parameter) can make a predicate non-sargable.
- `LIKE '%abc'` and `LIKE '%abc%'` can't seek; use full-text/trigram indexes.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE INDEX … (a, b, c)` | Composite key | Multi-predicate queries |
| `INCLUDE (cols)` | Non-key leaf columns | Covering without wider key |
| Actual plan / `EXPLAIN` | Seek vs scan, residual filter | Verify prefix use |
| `sys.dm_db_index_usage_stats` | Index read/write counts | Find unused/under-used |
| Expression/computed-column index | Make function predicates sargable | `LOWER(email)`, `YEAR(d)` |
| `ALTER INDEX … DISABLE` / `INVISIBLE` | Test drop safely | Index cleanup |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Equality columns first, range last | Different queries need different leading columns (consider 2 indexes) |
| Wide `INCLUDE` for hot queries | Table is write-heavy (index maintenance) |
| Leading column with high selectivity | It's rarely in predicates |
| Hash index (where supported) | You need ranges or ordering |
| BRIN | Data isn't physically correlated with the column |

### 9. Common mistakes
- Index `(b, a)` while queries filter on `a` only → scan → reorder or add `IX(a)`.
- Function on indexed column (`WHERE DATE(col)=…`) → scan → rewrite as range or add expression index.
- Placing the range column before equality columns → later columns can't seek → equality first.
- Many overlapping single-column indexes → write amplification → consolidate.
- Trusting `SELECT *` with an "index" → key lookups on every row → cover or narrow columns.

### 10. 🎯 Interview kit
**30-second answer:** A composite index is sorted by its columns left to right, so only a leftmost prefix gives a contiguous range to seek. Put equality predicates first, then the range/sort column; skipping the first column forces a scan, and functions on columns make predicates non-sargable.
**Structure a longer answer:** B-tree ordering → prefix examples → column ordering rule → covering/`INCLUDE` → verify with plans and usage stats → trade-offs on write cost.
**Likely follow-ups:**
- *Clustered vs nonclustered?* → clustered is the table's physical order; nonclustered holds keys plus pointer.
- *Index for `WHERE … ORDER BY`?* → equality columns, then order-by columns in same direction.
- *When is hash better than B-tree?* → pure equality lookups; no range/order.
- *How do you know an index earns its keep?* → usage stats (seeks, scans, last used) vs write cost; disable before dropping.
**Signals interviewers look for:** can reason from sort order, knows sargability, uses plans/stat views rather than guessing.

### 11. Check yourself
- [ ] For `IX(a,b,c)` classify ten predicate combinations as seek / partial seek / scan.
- [ ] Explain why a range on `b` stops `c` from narrowing the seek.
- [ ] Rewrite `WHERE YEAR(d)=2026` as sargable.
- [ ] Design one index for `WHERE tenant=? AND status=? ORDER BY created DESC LIMIT 20`.
- [ ] Describe how you'd decide whether to drop an index.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../../../TOOLS/MySQL/_index.md>)
- [PostgreSQL](<../../../TOOLS/PostgreSQL/_index.md>)

## Practice questions

- [Basic] [What is an index and why would you add one on conversation_id in a messages table for a chatbot backend?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40766_What_is_an_index_and_why_would_you_add_one_on_conversation_i.md>)
- [Intermediate] [What is the difference between the LIKE operator and full-text search?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40853_What_is_the_difference_between_the_LIKE_operator_and_full-te.md>)
- [Deep] [Compare a BRIN index and a B-tree on an append-only time-ordered column. On what physical layout does BRIN win, and how does clustering affect its effectiveness?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39896_Compare_a_BRIN_index_and_a_B-tree_on_an_append-only_time-ord.md>)
- [Deep] [What is the difference between a clustered and a non-clustered index?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40220_What_is_the_difference_between_a_clustered_and_a_non-cluster.md>)
- [Deep] [How would you build an index to support ORDER BY combined with a WHERE filter efficiently?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40248_How_would_you_build_an_index_to_support_ORDER_BY_combined_wi.md>)
- [Deep] [What is a covering index, and how could it make a 'latest value per host' query use an index-only scan?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-41014_What_is_a_covering_index_and_how_could_it_make_a_latest_valu.md>)
- [Deep] [Explain how a hash index's lack of ordering makes it useless for range scans and ORDER BY, and give one workload where hash beats B-tree despite that. What durability concern did older Postgres hash indexes have?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40451_Explain_how_a_hash_index_s_lack_of_ordering_makes_it_useless.md>)
- [Expert] [Why is a B-tree the default index structure, and when would a hash index be better?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40243_Why_is_a_B-tree_the_default_index_structure_and_when_would_a.md>)

## Related concepts

- [Query Rewrites and Plan Stability](<../../Query_Performance/Execution_Plans/Query_Rewrites_and_Plan_Stability.md>) *(Execution Plans)*
- [Index Access Paths](<../../Database_Internals/DBMS_Concepts/Index_Access_Paths.md>) *(DBMS Concepts)*
- [Index Tuning in Practice](<../../Query_Performance/Indexing/Index_Tuning_in_Practice/_index.md>) *(Indexing)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Performance Considerations
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Clustered Indexes
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Non-Clustered Indexes
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Filtered Indexes
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 9 Index Architecture
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 10 Index Behaviors
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — NoN-Clustered ColumNstore INdexes oN rowstore tables - LUSTERED COLUM STORE! DEXESO OWSTORE › Filtered Non-clustered Columnstore Indexes
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Non-clustered Rowstore Indexes on Columnstore Tables › Filtered Non-clustered Rowstore Indexes
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Ordered Columnstore Indexes › Cost-Benefit Analysis of Ordered Columnstore Indexes
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Clustered Indexes
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Dropping an Index › sys.hashindexes
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — sys.fulltextindexcolumns
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Effects from a Primary Index
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Writing Simple SELECT Queries › Viewing Execution Plans

