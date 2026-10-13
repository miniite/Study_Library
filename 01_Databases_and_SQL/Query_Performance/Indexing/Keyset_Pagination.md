---
id: "01_Databases_and_SQL/Query_Performance/Indexing/Keyset_Pagination"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Query_Performance/Indexing"
tools: []
---

# Keyset Pagination

**L4 Advanced** · [↑ Indexing](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Keyset (cursor) pagination fetches "the next N rows after the last one I saw" using an indexed, unique sort key, so each page costs the same no matter how deep you are — unlike `OFFSET`, which must read and discard every skipped row.

### 1. The big picture
The obvious way to paginate is `ORDER BY … LIMIT 20 OFFSET 20000`. Page 1 is fast; page 1000 is slow, because the engine still produces 20,020 rows in order and throws away the first 20,000. Under concurrent writes it is also *wrong*: an insert at the top shifts everything down by one, so the next page repeats a row (or a delete skips one).

**Keyset pagination** (a.k.a. seek method, cursor/token pagination) replaces "skip n rows" with "start after this key":

```sql
SELECT * FROM posts
WHERE (created_at, id) < (:last_created_at, :last_id)
ORDER BY created_at DESC, id DESC
LIMIT 20;
```
Combined with an index on `(created_at DESC, id DESC)`, the engine seeks directly to the cursor position and reads 20 entries. Cost is O(log n + page size), independent of depth.

### 2. Core concepts and vocabulary
- **OFFSET pagination** — skip a count of rows; cost grows with offset; unstable under writes.
- **Keyset / seek pagination** — filter by the sort key of the last row on the previous page.
- **Cursor / page token** — opaque encoding (often base64) of the last row's sort-key values, returned to the client.
- **Tie-breaker** — a unique column (usually the primary key) appended to the sort key so the order is total and deterministic.
- **Row-value comparison** — `(a, b) < (x, y)` — lexicographic comparison; supported in PostgreSQL, MySQL, SQLite; emulated in SQL Server.
- **Stable ordering** — the same query yields the same order across executions.
- **Deep paging** — requesting pages far from the start.
- **Index order scan** — reading an index in sorted order so `ORDER BY … LIMIT` needs no sort.

### 3. How it works, step by step
1. **Define a total order:** `ORDER BY created_at DESC, id DESC` (id breaks ties between identical timestamps).
2. **Create the matching index:** `(created_at DESC, id DESC)` — or `(tenant_id, created_at DESC, id DESC)` if every query filters by tenant.
3. **First page:** no cursor → `ORDER BY … LIMIT 21` (fetch N+1 to know if there is a next page).
4. **Return a cursor:** last row's `(created_at, id)`.
5. **Next page:** `WHERE (created_at, id) < (:c, :i) ORDER BY … LIMIT 21`. The engine seeks to the first entry strictly less than the cursor and reads forward.
6. **Previous page:** reverse the comparison and the order, then reverse the result in application code.

Expanded form where row-value comparison isn't supported:
```sql
WHERE created_at < :c OR (created_at = :c AND id < :i)
```
The `OR` can defeat index range scans in some optimizers; PostgreSQL handles the row comparison directly, SQL Server often benefits from rewriting as `UNION ALL` of the two ranges.

### 4. Worked example
```sql
-- Table and index
CREATE TABLE posts (
  id bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  tenant_id int NOT NULL,
  created_at timestamptz NOT NULL,
  title text NOT NULL
);
CREATE INDEX ix_posts_tenant_created ON posts (tenant_id, created_at DESC, id DESC);

-- Page 1
SELECT id, created_at, title FROM posts
WHERE tenant_id = 7
ORDER BY created_at DESC, id DESC
LIMIT 21;

-- Page 2 (cursor = last row of page 1: created_at='2025-03-04 10:00:00+00', id=9981)
SELECT id, created_at, title FROM posts
WHERE tenant_id = 7
  AND (created_at, id) < ('2025-03-04 10:00:00+00', 9981)
ORDER BY created_at DESC, id DESC
LIMIT 21;
```
`EXPLAIN` should show `Index Scan using ix_posts_tenant_created` with `Index Cond: ((tenant_id = 7) AND (ROW(created_at, id) < ROW(...)))` and **no Sort**. Compare `Buffers:` between page 1 and page 5000 — they're equal. With `OFFSET 100000` the same query reads ~100,000 index entries.

SQL Server version: `SELECT TOP (25) * FROM Employees WHERE EmployeeID > @LastEmployeeID ORDER BY EmployeeID;` — same idea with a single-column key.

### 5. Under the hood
- **Why OFFSET is O(offset).** The executor pulls rows from the ordered scan and counts. Even with a perfect index it walks `offset + limit` entries; without it, it sorts all matching rows (top-N heap sort of `offset+limit` rows).
- **Why `ORDER BY created_at DESC LIMIT 20` is fast with an index on `created_at`.** The B-tree leaf level is sorted and doubly linked; read the last 20 entries (backwards for DESC) and stop. Without the index: scan the whole table with a top-N heap or full sort.
- **Why adding `ORDER BY id` "for stability" can introduce a Sort.** If the index is `(created_at)` only, `ORDER BY created_at, id` isn't an index prefix order in engines that don't append the PK (PostgreSQL). Define the index as `(created_at, id)` — or in SQL Server/InnoDB, where the clustering/PK is implicitly appended to secondary indexes, the order may already be satisfied. For `WHERE a = ? ORDER BY b, id`, the index must be `(a, b, id)`.
- **Sort direction mixing.** `ORDER BY created_at DESC, id ASC` needs an index with mixed directions; engines that scan an index backwards only support all-ASC or all-DESC from a single-direction index. PostgreSQL/SQL Server allow per-column directions; MySQL 8 honours DESC in indexes.
- **Skew (celebrity users).** `(user_id, created_at)` with 5 M rows for one user is still O(page) *per page* with keyset; a time-out typically comes from OFFSET, a missing tie-breaker or a count query. Add keyset, a `LIMIT`, and avoid exact total counts (use "has more" or approximate counts).
- **Non-unique or nullable sort keys.** Without a unique tie-breaker, rows sharing the boundary value get skipped or duplicated. Nullable keys need explicit `NULLS FIRST/LAST` handling and a COALESCE-free predicate pattern; simplest is making the sort key `NOT NULL`.
- **Ranked/scored results.** Sorting by a computed relevance score has no stable index; use the tuple `(score, id)` with the score captured in the cursor (and accept that scores can change between pages), or snapshot results server-side, or paginate the underlying candidate set.
- **Extraction jobs.** For exporting a large table in chunks: `WHERE id > :last_id ORDER BY id LIMIT 10000` repeatedly, with `id` indexed (the PK). Each chunk is an index range read; the job is resumable by persisting `last_id`.
- **Limits of the technique.** No "jump to page 237" (cursor must be reached sequentially); total counts and page numbers are inherently expensive; changing the sort order changes the required index. Offer escape hatches: bounded OFFSET for small result sets, "jump to date" (a seek by timestamp), or search.

### 6. Performance and design implications
- **Index per sort order.** Each user-selectable sort needs its own matching index; limit sort options.
- **Always LIMIT page+1** to compute `has_next` without `COUNT(*)`.
- **Combine with filters:** equality filters become leading index columns, then the sort columns.
- **Consistency:** keyset gives stable pagination under inserts at the head and deletes; rows updated so their sort key moves may appear twice or disappear — acceptable for feeds, not for audits (use a snapshot/`created_at` cut-off).
- **Cursor hygiene:** treat tokens as opaque, sign/encrypt if exposure of keys matters, include sort and filter version.
- **Org-level:** ship one shared pagination helper (sort key + tie-breaker + cursor codec + required index declared) rather than bespoke code per team.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `WHERE (a, b) < (?, ?)` | Row-value keyset predicate | PostgreSQL, MySQL, SQLite |
| `CREATE INDEX … (a DESC, b DESC)` | Index matching sort | Mixed/DESC sorts |
| `TOP (n)` / `LIMIT n` / `FETCH FIRST n ROWS ONLY` | Row limit | Page size |
| `OFFSET … FETCH NEXT` | Standard offset paging | Shallow pages, small data |
| `EXPLAIN` | Confirm no Sort and index range condition | Always |
| Opaque cursor (base64 JSON) | Client-visible position | API design |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Infinite scroll, feeds, APIs, exports | Users need random page jumps and total counts |
| Large tables / deep paging | Small, static result sets (OFFSET is fine) |
| Writes occur between page requests | Sort key is non-unique and no tie-breaker is possible |
| Sort order is fixed or few | Arbitrary user-defined multi-column sorts |

### 9. Common mistakes
- **Keyset on a non-unique column** → duplicate/missing rows at page boundaries → add PK as tie-breaker in both `ORDER BY` and predicate.
- **Predicate and ORDER BY disagree** (`<` with ASC) → wrong/empty pages → keep direction and comparison consistent.
- **Index doesn't match** (`(created_at)` while ordering by `created_at, id`) → Sort node → create the exact index.
- **Using `OFFSET` for the "next" link under writes** → duplicates → keyset.
- **Exposing raw primary keys as cursors** → enumeration risk → opaque/signed tokens.
- **`COUNT(*)` for "total pages" every request** → slow → approximate or omit.
- **Ignoring tenant filter in the index** → seeks across tenants → leading `tenant_id`.

### 10. 🎯 Interview kit
**30-second answer:** Keyset pagination remembers the last row's sort-key values and queries `WHERE key < last ORDER BY key LIMIT n` against a matching index, so every page is an index seek plus n reads. OFFSET must scan and discard rows, so deep pages are slow and unstable under writes. You need a unique tie-breaker and an index that matches the order.

**Structure a longer answer:** OFFSET problems (cost, instability) → keyset predicate and index → tie-breaker and nullability → API cursor design → limits (no random access, counts) → example with tenant filter.

**Likely follow-ups:**
- *Exact index and WHERE for `(created_at, id)`?* → Index `(created_at DESC, id DESC)`; `WHERE (created_at, id) < (?, ?) ORDER BY created_at DESC, id DESC LIMIT n`.
- *User with 5 M rows times out?* → Ensure keyset (not OFFSET), seek on `(user_id, created_at, id)`, no count, limit page size.
- *Why did `ORDER BY id` add a Sort?* → Index order is `(created_at)`; sort key must be an index prefix including tie-breaker.
- *Ranked search results?* → `(score, id)` cursor with snapshot caveats, or deterministic candidate sets.
- *Defend keyset as default; what breaks?* → Random jump, non-unique keys; provide escape hatches.

**Signals interviewers look for:** concrete SQL, index-matching, total ordering, awareness of consistency and API trade-offs.

### 11. Check yourself
- [ ] Write keyset SQL and index for `(created_at DESC, id DESC)`.
- [ ] Explain the duplicate-row bug of OFFSET under inserts.
- [ ] Describe why a missing tie-breaker breaks pages.
- [ ] Show how to paginate a ranked result.
- [ ] Describe a chunked export using `id > last_id`.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [You need to show the newest 20 posts. How does an index on created_at make ORDER BY created_at DESC LIMIT 20 fast?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20015_You_need_to_show_the_newest_20_posts_How_does_an_index_on_cr.md>)
- [Intermediate] [You must serve WHERE customer_id = ? ORDER BY created_at DESC LIMIT 50 and also a rare full history export for the same customer. Design the index and explain how both queries use it.](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20076_You_must_serve_WHERE_customer_id_ORDER_BY_created_at_DESC_LI.md>)
- [Intermediate] [Your client caches a paginated list and a user reports a duplicated row across pages during active writes. How does non-stable ordering plus OFFSET cause this, and how does keyset fix it?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19839_Your_client_caches_a_paginated_list_and_a_user_reports_a_dup.md>)
- [Deep] [How would you extract a large table in ordered chunks using a keyset (WHERE id > last_id) approach, and what index makes it reliable?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20316_How_would_you_extract_a_large_table_in_ordered_chunks_using_.md>)
- [Deep] [What is keyset (cursor-based) pagination and why is it faster than OFFSET?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19506_What_is_keyset_cursor-based_pagination_and_why_is_it_faster_.md>)
- [Deep] [How would you design an index so that WHERE tenant_id = ? ORDER BY created_at DESC LIMIT 20 is fast in a multi-tenant app?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19577_How_would_you_design_an_index_so_that_WHERE_tenant_id_ORDER_.md>)
- [Deep] [For keyset pagination on (created_at, id), what exact index and WHERE clause do you need for stable ordering?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19661_For_keyset_pagination_on_created_at_id_what_exact_index_and_.md>)
- [Deep] [You add ORDER BY id at the end of a paginated query 'for stability' and the plan changes to a sort. Why, and how do you keep both stable order and index usage?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20091_You_add_ORDER_BY_id_at_the_end_of_a_paginated_query_for_stab.md>)

## Related concepts

- [MySQL Optimizer Behavior](<../../../TOOLS/MySQL/MySQL_Platform/MySQL_Optimizer_Behavior.md>) *(MySQL Platform)*
- [Large Values and TOAST](<../../SQL_Language/SQL_Fundamentals/Large_Values_and_TOAST.md>) *(SQL Fundamentals)*
- [Consistent Pagination](<../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Consistent_Pagination.md>) *(REST APIs)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Performance Considerations
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Best Practices
- *Database Internals* — Alex Petrov — Transaction Processing and Recovery › FIFO and LRU

