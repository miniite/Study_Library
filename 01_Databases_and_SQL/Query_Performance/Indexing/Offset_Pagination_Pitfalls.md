---
id: "01_Databases_and_SQL/Query_Performance/Indexing/Offset_Pagination_Pitfalls"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Query_Performance/Indexing"
tools: []
---

# Offset Pagination Pitfalls

**L4 Advanced** · [↑ Indexing](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** `OFFSET n` makes the database produce and throw away `n` rows before returning your page, so deep pages get slower linearly; keyset ("seek") pagination jumps straight to where the last page ended using an index, at the price of losing "jump to page N".

### 1. The big picture
Almost every list screen, search result, feed or export needs *pagination*: return a small slice of a big ordered result. SQL Server gives you three tools — `TOP`, `OFFSET … FETCH` (SQL Server 2012+; other engines spell it `LIMIT … OFFSET`) and `ROW_NUMBER()` in a CTE or derived table. All three are easy to write and look innocent on page 1.

The trap is that **`OFFSET` is not a seek, it is a skip**. The engine has no way to know where row 100,001 of an ordered result begins without producing the 100,000 rows before it. So page 1 costs "read 20 rows", page 5,000 costs "read 100,000 rows and discard 99,980". Latency, logical reads and CPU grow linearly with the page number, which is exactly the "users on page 500 see slow loads" complaint.

The alternative is *keyset pagination* (also called seek pagination or cursor/token pagination): instead of "skip N rows", you say "give me rows after the last key I saw", expressed as an ordinary `WHERE` predicate that an index can seek on. Cost per page becomes constant, regardless of depth.

### 2. Core concepts and vocabulary
- **Pagination / paging** — returning a result set one fixed-size chunk (page) at a time.
- **Page size** — rows per page (`@PageSize`); **page number** — which chunk (`@PageNo`).
- **`TOP (n)`** — returns the first n rows of the ordered result; no way to skip, so it is only "page 1" by itself.
- **`OFFSET m ROWS FETCH NEXT n ROWS ONLY`** — requires `ORDER BY`; skips m rows, returns the next n.
- **`ROW_NUMBER() OVER (ORDER BY …)`** — numbers every row; you then filter `RowNum BETWEEN …`. Same skip-cost, just spelled differently.
- **Keyset / seek pagination** — filter with the last-seen sort key(s) so the engine seeks into an index.
- **Tie-breaker** — a unique column (usually the primary key) appended to the sort so the order is total and deterministic.
- **Cursor / token** — the opaque value returned to clients that encodes the last-seen key(s).
- **Row-value comparison** — `(a, b) > (x, y)`; supported in PostgreSQL/MySQL, **not** in SQL Server, which needs the expanded `OR` form.
- **Page drift** — rows appearing twice or being skipped between pages because the underlying data changed between requests.

### 3. How it works, step by step
**What the engine does for `ORDER BY OrderDate, OrderID OFFSET 100000 ROWS FETCH NEXT 20 ROWS ONLY`:**
1. Locate the start of the ordered stream (scan an index in key order, or scan + sort if no suitable index).
2. Read and count rows until 100,000 have been passed — the engine fetches them (and, if the index doesn't cover the query, usually does a key lookup per row) and discards them.
3. Return the next 20 rows and stop (a `Top` operator in the plan).

Work done ≈ offset + page size. Row #100,001 needs 100,020 rows read; no index can shortcut the *counting*.

**What the engine does for keyset:**
1. The client sends the last row's sort key, e.g. `(OrderDate, OrderID) = ('2025-03-14', 88231)`.
2. The query says "rows strictly after that key, in the same order, take 20".
3. The optimizer does an *index seek* directly to that key and reads 20 rows forward. Work ≈ page size, always.

| | OFFSET / ROW_NUMBER | Keyset |
|---|---|---|
| Cost of page *k* | grows with *k* | constant |
| Jump to arbitrary page | yes | no (only next/previous) |
| Stable under inserts/deletes | no (drift) | yes (anchored to a key) |
| Needs deterministic unique order | recommended | **required** |
| Total page count | easy (`COUNT(*)`) | separate query / estimate |

### 4. Worked example
Setup — a large orders table with an index that matches the sort:

```sql
CREATE TABLE dbo.Orders (
    OrderID    INT IDENTITY PRIMARY KEY,
    OrderDate  DATE NOT NULL,
    CustomerID INT  NOT NULL,
    Total      DECIMAL(12,2) NOT NULL
);
CREATE INDEX IX_Orders_Date_ID ON dbo.Orders (OrderDate, OrderID)
    INCLUDE (CustomerID, Total);

-- (a) OFFSET paging: page 5,000 of size 20
DECLARE @PageNo INT = 5000, @PageSize INT = 20;
SELECT OrderID, OrderDate, CustomerID, Total
FROM dbo.Orders
ORDER BY OrderDate, OrderID
OFFSET (@PageNo - 1) * @PageSize ROWS FETCH NEXT @PageSize ROWS ONLY;

-- (b) Keyset paging: "next page after the last row I showed"
DECLARE @LastDate DATE = '2025-03-14', @LastID INT = 88231;
SELECT TOP (20) OrderID, OrderDate, CustomerID, Total
FROM dbo.Orders
WHERE OrderDate > @LastDate
   OR (OrderDate = @LastDate AND OrderID > @LastID)
ORDER BY OrderDate, OrderID;
```

Run both with `SET STATISTICS IO, TIME ON;` and look at the plan:
- (a) shows an **Index Scan** feeding a `Top` operator; `Actual Number of Rows` at the scan ≈ 99,980 + 20, and logical reads rise in proportion to `@PageNo`.
- (b) shows an **Index Seek** with a *Seek Predicate* on `OrderDate` (and `OrderID`); about 20 rows, a handful of logical reads, the same on every page.

The `OR` form is the portable-to-SQL-Server way of writing the row comparison `(OrderDate, OrderID) > (@LastDate, @LastID)`. Sargability note: the `OR` can confuse the optimizer into scanning the whole range from `@LastDate`; a common cure is to split into a `UNION ALL` of two seeks (`OrderDate = @LastDate AND OrderID > @LastID` and `OrderDate > @LastDate`), then `TOP (20)` over the union.

### 5. Under the hood
- **Why the index can't help the skip.** A B-tree index navigates by *key value*, not by *ordinal position*. It cannot answer "what is the 100,001st entry" without walking leaf pages and counting (no order-statistic tree). Leaf pages are linked in key order, so after the seek you walk forward — which is why keyset is cheap: it seeks by value, then reads forward 20 rows.
- **`ROW_NUMBER()` version** reads and numbers every row up to the upper bound (`RowNum <= @PageNo * @PageSize`), often with a `Segment` / `Sequence Project` pair, then filters. Equivalent cost to OFFSET; no better.
- **Count query.** "Page 5 of 1,200" needs `COUNT(*)`, which is itself a scan; for huge tables show "next/previous" or an approximate count (`sys.dm_db_partition_stats`) instead.
- **Non-covering index.** If the index doesn't contain every selected column, each of the skipped rows may even pay a key lookup, multiplying the waste. Covering the page query with `INCLUDE` columns helps both styles but keyset benefits most.
- **Consistency.** OFFSET pages are re-evaluated on every request: an insert at the top shifts everything and users see duplicates. Keyset anchors to a concrete key, so new rows appear only at the edge.
- **Distributed engines.** On Elasticsearch/Cassandra/BigQuery deep `from/size` is capped or very expensive for the same reason, and `search_after`/page tokens are the equivalent of keyset.

### 6. Performance and design implications
- Measure with `SET STATISTICS IO` (logical reads per page number) and the actual plan's "Actual Rows Read" on the scan/seek.
- Make the sort order match an index exactly (same columns, same direction, same leading column order); otherwise every page needs a full sort (`Sort` operator, memory grant, spills).
- Always include a **unique tie-breaker**. Sorting by `OrderDate` alone with keyset skips or duplicates rows that share a date.
- Mixed directions (`DESC, ASC`) can't use the simple expanded comparison; invert the comparison per column or build the index with matching directions.
- Cap `OFFSET` in public APIs (e.g. reject page > 100) and steer bulk consumers to keyset or token-based export.
- For chunked extraction/export, don't `OFFSET`: loop `WHERE Id > @last ORDER BY Id` with `TOP (n)`, remembering the max Id of each chunk.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ORDER BY … OFFSET m ROWS FETCH NEXT n ROWS ONLY` | Skip-and-take paging (needs `ORDER BY`) | Small/shallow result sets, admin screens, "jump to page" requirements |
| `TOP (n)` + `WHERE key > @last` | Keyset page | Infinite scroll, feeds, APIs, exports |
| `ROW_NUMBER() OVER (...)` | Number rows, filter by range | Legacy code; also when you need the number itself |
| `SET STATISTICS IO, TIME ON` | Show reads and CPU per query | Compare page 1 vs deep pages |
| `OPTION (RECOMPILE)` | Fresh plan per parameter set | When a cached plan suits page 1 but not deep pages |
| `CREATE INDEX … (sortcols) INCLUDE (...)` | Index matching sort order | Always, for the paging query |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| **OFFSET** — result is small, pages rarely go deep, users genuinely need "page 7 of 12" | Table is large, API is public, or clients/crawlers can request any page number |
| **Keyset** — infinite scroll, feeds, large exports, APIs, anything where only next/previous is needed | Product requires random page access or arbitrary sort columns chosen by the user (each needs its own index) |
| **Hybrid** — keyset for sequential paging, OFFSET capped to first N pages, or a lookup (first-key-of-each-page) table to translate "page N" into a key | Hybrid adds code complexity; keep it behind one pagination helper |
| **Token API** — return an opaque `next_cursor` so you can change internals later | Tokens that embed raw keys can leak ordering/IDs; sign or encode them |

### 9. Common mistakes
- **No `ORDER BY`** (or non-unique one) → pages overlap or miss rows because SQL gives no ordering guarantee → add a deterministic ORDER BY ending in the PK.
- **Using OFFSET for "chunked extraction" of a huge table** → each chunk slower than the last; the job never finishes → loop on `WHERE id > @last`.
- **Keyset on a non-unique column only** → ties dropped at page boundaries → append the PK to both ORDER BY and the predicate.
- **Wrong comparison operator** (`>=` re-returns the last row; `>` drops tied rows if tie-breaker is missing) → duplicates/gaps → use the compound form shown above.
- **Index doesn't match ORDER BY** → full sort every page → create an index with the same key order/direction.
- **`COUNT(*)` on every page request** → doubles the cost → cache it, estimate, or drop total counts.
- **Believing "an index fixes OFFSET"** → it only removes the sort, not the skip → move to keyset or limit depth.

### 10. 🎯 Interview kit
**30-second answer:** "`OFFSET` has to generate and discard all the skipped rows, so page N costs O(N) — deep pages get slower even with an index. Keyset pagination remembers the last row's sort key and uses `WHERE key > last ORDER BY key LIMIT n`, which is an index seek, so every page costs the same. The cost is you can't jump to an arbitrary page, and the order must be deterministic with a unique tie-breaker."

**Structure a longer answer:** clarify (data size, depth, jump-to-page need, sort options) → mechanism (skip vs seek) → trade-offs (random access, consistency, count) → migration (keep page numbers for the first few pages, expose `next_cursor`, build a matching index, cap offset) → close with metric: p99 latency and logical reads at deep pages.

**Likely follow-ups:**
- *How do you keep "jump to page" while migrating?* → Serve pages 1–N with OFFSET (cheap), deeper pages via cursor; or maintain a page-boundary table; or approximate by sampling keys; deprecate numeric pages gradually behind a compatibility shim.
- *How do you page by a nullable or multi-column sort?* → Tie-breaker, handle `NULL` ordering explicitly, expanded `OR` predicate or `UNION ALL` seeks.
- *Previous page?* → Reverse comparison and reverse order, then reverse the result in the app.
- *How do you know you're done migrating?* → Zero OFFSET > threshold in the slow log/Query Store; p99 flat across depth.
- *Data changes between requests?* → Keyset is stable; OFFSET drifts.

**Signals interviewers look for:** you explain *why* (counting/discarding), you mention the unique tie-breaker, you talk about indexes matching the sort, and you acknowledge the product trade-off rather than declaring keyset universally better.

### 11. Check yourself
- [ ] Explain, without code, why page 5,000 is slower than page 1 even with a perfect index.
- [ ] Write the keyset predicate for `ORDER BY CreatedAt DESC, Id DESC` for the "next" and "previous" pages.
- [ ] Show in an execution plan the difference between an Index Scan + Top and an Index Seek.
- [ ] Design a cursor token for an API and say what you would put in it and what you wouldn't.
- [ ] Outline a migration plan for 100 OFFSET-based endpoints and how you'd measure completion.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [You have to speed up pagination on a large table. Why can page 5000 with OFFSET be slow and what is a simpler alternative?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20042_You_have_to_speed_up_pagination_on_a_large_table_Why_can_pag.md>)
- [Intermediate] [Your infinite-scroll feed uses OFFSET and users on page 500 see slow loads. Walk through migrating to keyset pagination while the frontend still needs a 'jump to page' affordance.](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19809_Your_infinite-scroll_feed_uses_OFFSET_and_users_on_page_500_.md>)
- [Deep] [Why does OFFSET-based pagination over a huge export table get slower on later pages, and what simpler approach works for chunked extraction?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20315_Why_does_OFFSET-based_pagination_over_a_huge_export_table_ge.md>)
- [Deep] [Why can OFFSET-based pagination (LIMIT 10 OFFSET 100000) become slow, and what is the fix?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19505_Why_can_OFFSET-based_pagination_LIMIT_10_OFFSET_100000_becom.md>)
- [Deep] [How would you make an existing OFFSET-based paginated endpoint fast on deep pages while the frontend still expects page numbers during the migration?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20164_How_would_you_make_an_existing_OFFSET-based_paginated_endpoi.md>)
- [Deep] [You're deciding the paved-road default for pagination across every list API in the company. Keyset is faster but breaks 'jump to page N.' How do you set an org-wide standard that balances performance, product needs, and migration cost of legacy OFFSET endpoints?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19891_You_re_deciding_the_paved-road_default_for_pagination_across.md>)
- [Deep] [A hiring-manager round asks how you'd sequence a decade-old system's move off OFFSET pagination org-wide when hundreds of endpoints and clients depend on page numbers. Lay out the migration strategy, backward-compat shims, and how you'd measure done.](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19948_A_hiring-manager_round_asks_how_you_d_sequence_a_decade-old_.md>)
- [Expert] [You set random_page_cost too high on an SSD-backed database. What symptom would you expect in query plans?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19574_You_set_random_page_cost_too_high_on_an_SSD-backed_database_.md>)

## Related concepts

- [Cursor Pagination](<../../../TOOLS/Python/FastAPI/Cursor_Pagination.md>) *(FastAPI)*
- [Consistent Pagination](<../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Consistent_Pagination.md>) *(REST APIs)*
- [MySQL Optimizer Behavior](<../../../TOOLS/MySQL/MySQL_Platform/MySQL_Optimizer_Behavior.md>) *(MySQL Platform)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Pagination: TOP, OFFSET-FETCH, ROWNUMBER
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Pagination with OFFSET-FETCH
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Choosing between TOP, OFFSET-FETCH, and ROWNUMBER()
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Writing Advanced Queries › Paging

