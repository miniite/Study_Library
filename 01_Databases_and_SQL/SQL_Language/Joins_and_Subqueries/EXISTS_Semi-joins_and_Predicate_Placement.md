---
id: "01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/EXISTS_Semi-joins_and_Predicate_Placement"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries"
tools: []
---

# EXISTS Semi-joins and Predicate Placement

**L4 Advanced** · [↑ Joins and Subqueries](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** `EXISTS` asks "is there at least one matching row?" and is executed as a *semi-join* that stops at the first match and never multiplies rows; where you put a predicate (`ON`, `WHERE`, inside the subquery) decides both correctness (for outer joins) and speed (how early rows are filtered).

### 1. The big picture
Many questions are existence questions: "customers who have placed an order", "products never sold", "orders with at least one late item". You can write them with `JOIN`, `IN`, or `EXISTS`. They look interchangeable, but they differ in two ways that matter: whether duplicates appear (a join fans out, a semi-join does not) and how NULLs behave (`NOT IN` breaks, `NOT EXISTS` does not). Underneath, the optimizer picks a physical join algorithm — nested loops, merge, or hash — and a *semi* or *anti-semi* variant of it.

The second half of the topic is *predicate placement*: the same condition in `ON`, in `WHERE` or in a pre-filtering subquery gives identical results for inner joins but different results for outer joins, and can give different plans.

### 2. Core concepts and vocabulary
- **Semi-join** — returns each row of the left input at most once if *at least one* match exists on the right; right-side columns are never returned. `EXISTS` and `IN` are semi-joins.
- **Anti-semi-join** — returns left rows with *no* match; `NOT EXISTS`, `LEFT JOIN … IS NULL`.
- **Correlated subquery** — inner query references the outer row (`o.cust_id = c.cust_id`); logically evaluated per outer row.
- **Fan-out** — a plain join repeating the left row once per match; semi-joins avoid it.
- **Nested Loop Join** — for each outer row, probe the inner input; best for small outer inputs with an index on the inner join key.
- **Hash Join** — build a hash table on one input, probe with the other; good for large unsorted inputs; may spill to `tempdb` when the memory grant is short.
- **Merge Join** — walks two inputs already sorted on the join key in one pass; cheapest when both sides come pre-sorted from indexes.
- **Predicate pushdown** — filtering as early as possible (at the scan) so fewer rows reach the join.
- **Three-valued logic** — comparisons with NULL yield UNKNOWN, which filters rows out.

### 3. How it works, step by step
**EXISTS**
1. For each candidate left row the engine asks "does the subquery return a row?". The select list of the subquery is ignored (`SELECT 1`, `SELECT *` are the same).
2. It stops at the first row found (short-circuit) — no duplicates, no aggregation.
3. The optimizer usually *decorrelates* it into a `Left Semi Join` with a hash, merge or nested-loop implementation, so it is not literally run per row.

**IN vs EXISTS vs JOIN** (book guidance: `IN` for small static lists, `EXISTS` for correlated existence, `JOIN` when you need columns from the other table; test with `SET STATISTICS IO, TIME ON`)
1. `IN (subquery)` — semi-join; with NULLs in the subquery, `NOT IN` returns *no rows* (because `x <> NULL` is UNKNOWN).
2. `EXISTS / NOT EXISTS` — NULL-safe; usually the best choice for existence and non-existence.
3. `JOIN` — may duplicate; need `DISTINCT` or `GROUP BY` to repair, which costs a sort/hash.

**Predicate placement**
1. Inner join: `ON` and `WHERE` are equivalent; the optimizer treats them identically.
2. `LEFT JOIN`: a right-table predicate in `ON` filters the *right side before matching* (left rows survive with NULLs); in `WHERE` it filters the *final result* and removes NULL-padded rows, silently turning the join into an inner join.
3. Left-table predicates: in `WHERE` (filtering left rows) — putting them in `ON` of a left join does *not* remove left rows, it only prevents matches.
4. Pre-filter subqueries (`JOIN (SELECT … WHERE …) x`) are inlined; the optimizer treats them the same as `WHERE` in most cases, so use them for clarity, not as a hint.

### 4. Worked example
```sql
-- Customers with at least one order (no duplicates, stops at first match)
SELECT c.cust_id, c.name
FROM customers c
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.cust_id = c.cust_id);

-- Customers with NO orders: the NULL-safe way
SELECT c.cust_id
FROM customers c
WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.cust_id = c.cust_id);

-- Equivalent anti-join with LEFT JOIN
SELECT c.cust_id
FROM customers c LEFT JOIN orders o ON o.cust_id = c.cust_id
WHERE o.order_id IS NULL;          -- test a NOT NULL column of the right side

-- Trap: NOT IN with a NULL in the subquery returns zero rows
SELECT c.cust_id FROM customers c
WHERE c.cust_id NOT IN (SELECT cust_id FROM orders);   -- if any orders.cust_id IS NULL => empty result

-- Predicate placement on LEFT JOIN
SELECT c.cust_id, o.order_id
FROM customers c
LEFT JOIN orders o ON o.cust_id = c.cust_id AND o.status = 'OPEN';  -- all customers kept
SELECT c.cust_id, o.order_id
FROM customers c
LEFT JOIN orders o ON o.cust_id = c.cust_id
WHERE o.status = 'OPEN';                                            -- only customers with open orders
```
Reading the plan: the `EXISTS` query typically shows *Nested Loops (Left Semi Join)* with an index seek on `orders(cust_id)` for small outer inputs, or *Hash Match (Left Semi Join)* for large ones. The `LEFT JOIN … IS NULL` form compiles to *Left Anti Semi Join* on modern SQL Server, so cost is similar to `NOT EXISTS`.

### 5. Under the hood   *(optional at L4)*
- **Nested loops**: outer rows × inner probes; cost ≈ outer_rows × cost_per_seek. Excellent with an index on the inner key and a selective outer; terrible when both are large and unindexed (quadratic).
- **Merge join**: both inputs sorted on the key (via clustered/nonclustered indexes or a Sort operator); linear scan of each; requires an equality predicate, and many-to-many needs a worktable. Cheaper than hash when both inputs already arrive sorted (e.g. clustered keys, as with a header/detail pair keyed on the order id); if the optimizer has to add a Sort for either side, hash is usually cheaper.
- **Hash join**: memory-hungry; the build side should be the smaller input. Adaptive/batch-mode variants exist in newer versions. A spill shows up as a warning on the Hash Match operator and as `tempdb` writes.
- **Semi-join short-circuit**: a hash semi-join de-duplicates the build side or stops probing after the first hit; nested loops semi-join stops the inner seek after one row.
- **Why `COUNT(*) > 0` is worse than `EXISTS`**: it counts all matches; `EXISTS` stops.
- **Pre-joining (denormalising)** — the data-engineering view: when analytic queries repeatedly perform the same expensive join, materialize it (a wide table or materialized view); in columnar warehouses joins are fast but still repeated work.

### 6. Performance and design implications
- Index the correlated column on the inner table (`orders(cust_id)`) — turns the semi-join into cheap seeks and often yields a merge join.
- Prefer `EXISTS` to `JOIN + DISTINCT` for filter-only needs: no sort/hash for de-duplication.
- Keep predicates **sargable** (no function on the column) so they can be pushed to seeks.
- Check the plan for estimated vs actual rows; wrong estimates choose nested loops for a big input (slow) or hash for a tiny input (memory grant waste).
- Pre-aggregate before the join when only aggregates are needed from a many side; the trade-off is an extra derived step that may prevent some rewrites and needs care with grain.
- Use `SET STATISTICS IO, TIME ON` and compare logical reads between `IN`, `EXISTS` and `JOIN` variants rather than assuming.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `EXISTS (SELECT 1 …)` | Semi-join existence test | "has at least one" |
| `NOT EXISTS (…)` | Anti-semi-join, NULL-safe | "has none" |
| `LEFT JOIN … WHERE r.pk IS NULL` | Anti-join via outer join | When you also need left columns and prefer join syntax |
| `IN (list)` | Membership in a short static list | Constants |
| `OPTION (HASH JOIN \| MERGE JOIN \| LOOP JOIN)` | Forces an algorithm (SQL Server) | Diagnosis only, not production |
| `SET STATISTICS IO, TIME ON` | Reads and CPU | Compare variants |
| Actual execution plan | Join type, spills, estimates | Always |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `EXISTS` for "has any" | You need columns from the other table (use a join) |
| `NOT EXISTS` for "has none" | Using `NOT IN` against a nullable column |
| Nested loops with an index | Both inputs are big — hash/merge wins |
| Merge join (pre-sorted inputs) | Sorting both inputs just to merge — usually hash is cheaper |
| Predicate in `ON` for outer joins to keep left rows | Predicate in `WHERE` on the right table unless you *want* inner-join behaviour |

### 9. Common mistakes
- `NOT IN` with a nullable subquery column → empty result, no error → use `NOT EXISTS` or filter `IS NOT NULL`.
- Right-table filter in `WHERE` of a left join → customers without orders vanish → move to `ON`.
- `JOIN` then `DISTINCT` to emulate existence → extra sort and hidden bugs with multi-column rows → `EXISTS`.
- Wrapping the join column in a function → no seek, scan + hash → compare raw columns.
- Forcing a join hint in production → plan breaks as data changes → fix statistics/indexes instead.
- Believing `EXISTS` is always faster than `IN` — modern optimizers often produce the same plan; measure.

### 10. 🎯 Interview kit
**30-second answer:** "`EXISTS` is a semi-join: it returns each outer row once if the correlated subquery finds at least one row, and it stops at the first match, so it can't duplicate rows. `NOT EXISTS` is the NULL-safe way to find non-matches; `NOT IN` returns nothing if the subquery contains a NULL. For outer joins, predicates on the optional side belong in `ON`, otherwise the join collapses into an inner join."

**Structure a longer answer:** state the question type (existence / non-existence / need columns) → pick `EXISTS` / `NOT EXISTS` / `JOIN` → NULL semantics → physical join algorithms and when each wins → predicate placement and sargability → verify with the plan.

**Likely follow-ups:**
- *Merge join vs hash join?* → merge needs both inputs sorted on the key and streams them in linear time; hash builds an in-memory table of the smaller side and tolerates unsorted input.
- *When is a nested loop best?* → small outer input and an index on the inner key.
- *Subquery that pre-filters vs `WHERE` in the outer query?* → for inner joins they're equivalent after optimization; for outer joins they differ in semantics.
- *How to avoid fan-out?* → pre-aggregate the many side, or use `EXISTS` when columns are not needed.

**Signals interviewers look for:** NULL awareness with `NOT IN`, the outer-join `WHERE` trap, algorithm vocabulary (semi/anti-semi, build/probe), and evidence-based tuning (plans and `STATISTICS IO`).

### 11. Check yourself
- [ ] Write "customers who never ordered" three ways and say which is NULL-unsafe.
- [ ] Show with data how moving a predicate from `ON` to `WHERE` changes a left join.
- [ ] Explain why a hash join may spill and how to spot it.
- [ ] Choose a join algorithm for (a) 10 rows × 10M indexed rows, (b) two 10M-row sorted indexes, (c) two 10M-row heaps.
- [ ] Rewrite a `JOIN + DISTINCT` existence query with `EXISTS`.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [How can an index on the join column improve a join's performance for a fresher to understand?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30462_How_can_an_index_on_the_join_column_improve_a_join_s_perform.md>)
- [Deep] [Interviewer probe: your join query is slow — what is the first thing you check about the join keys?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30479_Interviewer_probe_your_join_query_is_slow_what_is_the_first_.md>)
- [Deep] [Your CTE is being re-executed multiple times where you expected it to run once — how do you force or avoid materialization across engines?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30306_Your_CTE_is_being_re-executed_multiple_times_where_you_expec.md>)
- [Deep] [When you join a large table to a tiny lookup, why does the optimizer usually build the hash on the small side, and when is that wrong?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30321_When_you_join_a_large_table_to_a_tiny_lookup_why_does_the_op.md>)
- [Deep] [A join to a view is slow because the view has DISTINCT — how does that block predicate pushdown and how do you restructure it?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30338_A_join_to_a_view_is_slow_because_the_view_has_DISTINCT_how_d.md>)
- [Deep] [When a subquery's result is small and stable, how does the optimizer decide to cache/spool it versus re-run it, and can you hint that?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30356_When_a_subquery_s_result_is_small_and_stable_how_does_the_op.md>)
- [Deep] [Interviewer probe: you added an index but the join got slower — how can a new index mislead the optimizer into a worse join method?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30371_Interviewer_probe_you_added_an_index_but_the_join_got_slower.md>)
- [Deep] [How would you tune a query that joins a partitioned fact table to a dimension where partition pruning is not happening because the filter is on the dimension not the fact?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Q-30390_How_would_you_tune_a_query_that_joins_a_partitioned_fact_tab.md>)

## Related concepts

- [Query Rewrites and Plan Stability](<../../Query_Performance/Execution_Plans/Query_Rewrites_and_Plan_Stability.md>) *(Execution Plans)*
- [MySQL Optimizer Behavior](<../../../TOOLS/MySQL/MySQL_Platform/MySQL_Optimizer_Behavior.md>) *(MySQL Platform)*
- [Join Algorithm Selection](<../../Query_Performance/Indexing/Join_Algorithm_Selection.md>) *(Indexing)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Performance Considerations
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Best Practices
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › Best Practices
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › Performance Considerations
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Choosing Technologies Across the Data Engineering Lifecycle › Asymmetric Optimization
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Chapter 8. Queries, Modeling, and Transformation › Optimize your JOIN strategy and schema
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Joining Tables › Avoiding an Incorrect Join Condition
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Joining Tables › Self-Joins
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Joining Tables › Thinking About Performance
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Joining Tables › Nested Loop
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Optimization › 10.5 #58 Working against the optimizer
- *Data Algorithms with Spark* — Mahmoud Parsian — **CHAPTER 11** › CHAPTER 11
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 6 Using The Query Store For Query Performance And Execution Plans
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 9 Index Architecture
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 14 Query Design Analysis

