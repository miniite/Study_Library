---
id: "TOOLS/PostgreSQL/PostgreSQL_Platform/Indexing_Multi-join_Queries"
level: 4
type: "concept"
topic: "TOOLS/PostgreSQL/PostgreSQL_Platform"
tools: []
---

# Indexing Multi-join Queries

**L4 Advanced** · [↑ PostgreSQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** For a multi-join query, index the join keys on the side that is probed repeatedly (usually the foreign-key column of the child table) and the filter columns that shrink the row count first — then read the plan to confirm the join order and method.

### 1. The big picture
A multi-table query is a tree of two-way joins. The planner chooses a *join order* and a *join method* per step to minimize estimated cost; indexes change which choices are cheap. The art is to give the planner a cheap path *into* each table: a selective filter index on the driving table and a join-key index on every table that will be looked up row by row. Overindexing is not the answer — every index costs write time, vacuum time and space — so you index to the plan you want, and prove it.

### 2. Core concepts and vocabulary
- **Join key** — the columns equated in `ON`/`USING`.
- **Selectivity** — fraction of rows a predicate keeps; low fraction = highly selective = index-friendly.
- **Nested Loop** — for each outer row, probe the inner side (best with an index and few outer rows).
- **Hash Join** — build a hash table of the smaller input, probe with the larger (needs no index, uses `work_mem`).
- **Merge Join** — both inputs sorted on the key, streamed in step (good if pre-sorted by an index).
- **Composite (multicolumn) index** — one B-tree over several columns; column order matters.
- **Covering index** — contains all needed columns (`INCLUDE`) so an *Index Only Scan* avoids the heap.
- **Partial index** — index with a `WHERE` predicate for a hot subset.
- **Foreign key** — PostgreSQL does **not** auto-index the referencing column.

### 3. How it works, step by step
1. The planner estimates row counts after each filter using column statistics.
2. It enumerates join orders (exhaustively up to `join_collapse_limit`/`from_collapse_limit` tables, then heuristically, GEQO beyond `geqo_threshold` = 12).
3. For each pair it costs nested loop / hash / merge with available index paths.
4. It picks the cheapest tree; you see it in `EXPLAIN`.

Practical recipe:
1. Find the most selective filter; make that table the *driver* (index on the filter column(s)).
2. For each join from the driver outward, ensure the *inner* table has an index leading with the join column(s).
3. Add filter columns after the join column when the same table is both probed and filtered: `(customer_id, status)`.
4. Consider `INCLUDE` for columns only needed in `SELECT`.
5. Re-run `EXPLAIN (ANALYZE, BUFFERS)`; remove indexes that stay unused (`pg_stat_user_indexes.idx_scan = 0`).

### 4. Worked example
```sql
SELECT c.name, o.order_id, p.title, oi.qty
FROM customers   c
JOIN orders      o  ON o.customer_id = c.customer_id
JOIN order_items oi ON oi.order_id   = o.order_id
JOIN products    p  ON p.product_id  = oi.product_id
WHERE c.country = 'IN'
  AND o.order_date >= DATE '2026-01-01'
  AND o.status = 'shipped';

-- indexes to support the plan
CREATE INDEX ON customers (country);                       -- filter (if selective)
CREATE INDEX ON orders (customer_id, order_date)           -- join key + range
  INCLUDE (status) ;                                       -- avoid heap for the filter
CREATE INDEX ON order_items (order_id) INCLUDE (product_id, qty);   -- FK probe, covering
-- products.product_id is a primary key: already indexed
```
Expected plan shape:
```
Nested Loop
  -> Nested Loop
       -> Nested Loop
            -> Bitmap Heap Scan on customers  (country = 'IN')
            -> Index Only Scan on orders_customer_id_order_date_idx
       -> Index Only Scan on order_items_order_id_idx
  -> Index Scan using products_pkey
```
Check `actual rows × loops` at each node: if the first node returns 400,000 customers, a hash join with a seq scan may be better — don't fight it. If estimates are 100× off, run `ANALYZE` or create extended statistics (`CREATE STATISTICS … (dependencies) ON country, status …`).

### 5. Under the hood   *(optional at this level)*
- A B-tree on `(a, b)` can serve `a = ?`, `a = ? AND b = ?`, `a = ? AND b > ?`, and ordering by `a, b`; it cannot efficiently serve `b = ?` alone (full index scan at best). Equality columns first, then range column last.
- Index Only Scan still consults the **visibility map**; on tables with many un-vacuumed changes it falls back to heap fetches (`Heap Fetches: N`). Keep autovacuum healthy.
- Bitmap scans combine multiple single-column indexes (`BitmapAnd/Or`) — a fallback when no composite index fits, at the cost of heap page-order access.
- Hash joins build `work_mem × hash_mem_multiplier` and partition into batches when larger; many tiny nested-loop probes on cold data become random I/O — `random_page_cost` (default 4, ~1.1 on SSD) affects this choice.
- Join *elimination* and predicate pushdown happen for outer joins/unique keys; `LEFT JOIN` on a unique key whose columns aren't used can be removed entirely.

### 6. Performance and design implications
- **Index every foreign key** that you join or delete-cascade through; otherwise parent deletes scan the child table.
- Prefer narrow keys (bigint, uuid v7) — wide composite keys bloat every index.
- Beware of low-selectivity indexes (booleans, tiny status sets); use partial indexes (`WHERE status = 'open'`) instead.
- Function-wrapped columns (`lower(email) = …`, `date(created_at) = …`) need expression indexes or rewriting.
- Measure write cost: each extra index slows `INSERT/UPDATE` and prevents HOT updates when an indexed column changes.
- `join_collapse_limit` and `ANALYZE` freshness matter more than hints; PostgreSQL has no hints — shape the query (CTEs, `OFFSET 0`, materialization) only as a last resort.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE INDEX CONCURRENTLY` | build without blocking writes | production |
| `INCLUDE (cols)` | add payload columns to B-tree | index-only scans |
| `CREATE STATISTICS` | multi-column stats | correlated filters |
| `EXPLAIN (ANALYZE, BUFFERS)` | verify plan | after every change |
| `pg_stat_user_indexes` | index usage | find dead indexes |
| `SET enable_nestloop = off` | session experiment | diagnosing planner choices only |
| `random_page_cost` | cost of random I/O | tune for SSD |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Composite `(join_key, filter)` | different queries need different leading columns |
| Covering index | wide payloads, frequently updated columns |
| Partial index | predicate values vary per query |
| Hash join (no index) | tiny memory, enormous inputs |
| Denormalize / precompute | joins are on hot path and data changes rarely |

### 9. Common mistakes
- No index on the FK column → each parent delete/lookup scans child table → index it.
- Composite index in wrong column order → unused → equality columns first.
- Indexing every column "just in case" → slow writes and bloated buffers → index to measured plans.
- Judging by `EXPLAIN` without `ANALYZE` → trusting wrong estimates → compare estimated vs actual.
- Forcing nested loops with `enable_*` in production → fragile → fix stats/indexes instead.
- Leaving unused duplicate indexes → wasted space → drop after checking `idx_scan`.

### 10. 🎯 Interview kit
**30-second answer:** "Index the join columns on the table being probed (typically child FKs) and the most selective filter on the driving table; use composite indexes with equality columns first, `INCLUDE` for covering, and verify the join order and method with `EXPLAIN ANALYZE`."
**Structure a longer answer:** how the planner orders joins → methods and what each needs → index design rules → verification and cost.
**Likely follow-ups:** *Does PG index FKs automatically?* → no. *Nested loop vs hash?* → few rows with index vs large unsorted sets. *Why isn't my index used?* → low selectivity, stats, type mismatch, function on column. *Index-only scan didn't help?* → visibility map.
**Signals interviewers look for:** you think in terms of row counts and selectivity, not "add an index", and you weigh write cost.

### 11. Check yourself
- [ ] For a 4-table join, pick the driver and list the indexes you'd create.
- [ ] Explain column order in a composite index with an equality and a range predicate.
- [ ] Interpret `Heap Fetches` in an index-only scan.
- [ ] Find and drop an unused index safely.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is an index and how does it speed up queries in PostgreSQL?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35996_What_is_an_index_and_how_does_it_speed_up_queries_in_Postgre.md>)
- [Intermediate] [Your app filters rows by a computed expression like extract(year from created_at) and it never uses the index; explain why and show two ways to make it index-friendly.](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35803_Your_app_filters_rows_by_a_computed_expression_like_extract_.md>)
- [Intermediate] [You need case-insensitive uniqueness on email but also fast case-insensitive lookups; compare a citext column, a unique index on lower(email), and a generated normalized column, and which you'd ship.](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36072_You_need_case-insensitive_uniqueness_on_email_but_also_fast_.md>)
- [Deep] [You store product attributes in jsonb and need to query by a specific key frequently; would you build an expression index on (data->>'brand') or a GIN index, and how does the query shape decide which the planner picks?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36264_You_store_product_attributes_in_jsonb_and_need_to_query_by_a.md>)
- [Deep] [How do you detect unused indexes so you can drop them, using pg_stat_user_indexes?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35904_How_do_you_detect_unused_indexes_so_you_can_drop_them_using_.md>)
- [Deep] [What is a partial index and give a use case for it.](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36369_What_is_a_partial_index_and_give_a_use_case_for_it.md>)
- [Deep] [Your API paginates with keyset on (created_at, id) but timestamps have duplicates and sometimes rows are skipped or repeated; walk me through the exact tuple-comparison predicate that makes keyset correct with ties.](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36067_Your_API_paginates_with_keyset_on_created_at_id_but_timestam.md>)
- [Expert] [What is a Bitmap Heap Scan and Bitmap Index Scan, and when does the planner prefer it over a plain Index Scan?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35894_What_is_a_Bitmap_Heap_Scan_and_Bitmap_Index_Scan_and_when_do.md>)

## Related concepts

- [Index Access Paths](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Index_Access_Paths.md>) *(DBMS Concepts)*
- [Index-only Scans](<../../../01_Databases_and_SQL/Query_Performance/Indexing/Index-only_Scans.md>) *(Indexing)*
- [Leftmost Prefix Rule](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Leftmost_Prefix_Rule.md>) *(SQL Fundamentals)*

## References

- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Density Vector › sys.stats
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — User Columns
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — The Procedures › Restrictions on Spatial Indexes

