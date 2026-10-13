---
id: "TOOLS/PostgreSQL/PostgreSQL_Platform/Materialized_CTEs"
level: 4
type: "concept"
topic: "TOOLS/PostgreSQL/PostgreSQL_Platform"
tools: []
---

# Materialized CTEs

**L4 Advanced** · [↑ PostgreSQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Since PostgreSQL 12 a non-recursive CTE is *inlined* into the query like a subquery when it's referenced once and side-effect free; `MATERIALIZED` forces it to be computed once and reused, `NOT MATERIALIZED` forces inlining.

### 1. The big picture
A `WITH` query (common table expression, CTE) names an intermediate result so a query reads top-down instead of nested. Before PostgreSQL 12 every CTE was an **optimization fence**: it was always computed in full and stored, and the outer query's filters could not be pushed into it. That was occasionally a useful hint and often a performance bug. From version 12 the planner treats a simple CTE as inlineable, and you control the behavior explicitly. Understanding this decides whether your readable query is also fast.

### 2. Core concepts and vocabulary
- **CTE** — `WITH name AS (subquery)`; scope is the one statement.
- **Materialization** — computing the CTE result once into a tuplestore (memory, spilling to temp files) and scanning it from there.
- **Inlining** — substituting the CTE's query into each reference, so the planner optimizes it with the outer query.
- **Optimization fence** — a boundary the planner won't push predicates or join reordering across.
- **Recursive CTE** — `WITH RECURSIVE`; always materialized-style iteration (working table).
- **Data-modifying CTE** — `INSERT/UPDATE/DELETE … RETURNING` inside `WITH`; always executed exactly once, to completion.
- **Predicate pushdown** — moving an outer `WHERE` into the inner query so fewer rows are produced.

### 3. How it works, step by step
Rules the planner applies (PG ≥ 12):
1. A CTE is **materialized** if it is `MATERIALIZED`, is recursive, has side effects (data-modifying or volatile functions like `random()`), or is referenced more than once (default behavior).
2. Otherwise (referenced once, `SELECT`, side-effect free) it is **inlined**.
3. `NOT MATERIALIZED` forces inlining even when referenced multiple times (each reference replans the subquery body).
4. Materialized CTE: planner optimizes the body in isolation, runs it once on first use, outer references scan the stored rows; **no predicate pushdown, no statistics on the result** (row estimate is guessed from the body's estimate).

### 4. Worked example
```sql
-- 1. Inlined (default): the filter on customer_id is pushed down into the scan
WITH o AS (SELECT * FROM orders)
SELECT * FROM o WHERE customer_id = 42;

-- 2. Forced fence: scans/stores ALL orders first, then filters  (slow on big tables)
WITH o AS MATERIALIZED (SELECT * FROM orders)
SELECT * FROM o WHERE customer_id = 42;

-- 3. Useful materialization: expensive subquery used twice
WITH monthly AS MATERIALIZED (
  SELECT date_trunc('month', order_date) AS m, sum(amount) AS rev
  FROM orders GROUP BY 1
)
SELECT cur.m, cur.rev, prev.rev AS prev_rev
FROM monthly cur
LEFT JOIN monthly prev ON prev.m = cur.m - interval '1 month';

-- 4. Data-modifying CTE: move rows atomically
WITH moved AS (
  DELETE FROM orders_hot WHERE created_at < now() - interval '30 days' RETURNING *
)
INSERT INTO orders_archive SELECT * FROM moved;
```
Check with `EXPLAIN`: an inlined CTE doesn't appear as a node; a materialized one shows `CTE Scan on o` with a separate `CTE o` subplan. If the outer filter appears under the `CTE Scan` as `Filter: (customer_id = 42)` and the CTE subplan reads the whole table — that's the fence.

### 5. Under the hood   *(optional at this level)*
- Materialized CTE output goes to a tuplestore bounded by `work_mem`; beyond that it spills to temp files.
- Multiple references to a materialized CTE share one tuplestore (each with its own read pointer), so a heavy computation runs once.
- The planner has no column statistics for a CTE scan, so it uses default selectivities; joins on CTE outputs can be badly estimated. Materializing into a `TEMP TABLE` and running `ANALYZE` gives real stats when this matters.
- A fence can be deliberately useful: to force an `OFFSET 0` style barrier, to guarantee a volatile function evaluates once, or to stop the planner from choosing a bad join order for a complicated join.
- A recursive CTE iterates: non-recursive term → working table → recursive term referencing the working table → append to result until no new rows; `UNION` (dedupe) is the cycle-safe form, `CYCLE` clause (PG14+) detects loops.

### 6. Performance and design implications
- Upgraded from < 12? Queries that relied on the fence may change plans; those that suffered from it get faster.
- Use `MATERIALIZED` when the CTE is expensive and referenced several times, or when you need it computed exactly once (side effects, `random()`).
- Use `NOT MATERIALIZED` (or a plain subquery) when a selective outer filter should reach the base table.
- For multi-step ETL with large intermediates, a temp table + `ANALYZE` + indexes beats chained CTEs.
- Keep CTEs for readability and recursion; they are not free if materialized — measure with `EXPLAIN (ANALYZE, BUFFERS)`.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `WITH x AS MATERIALIZED (…)` | force single computation | expensive, reused, or fence needed |
| `WITH x AS NOT MATERIALIZED (…)` | force inlining | filter pushdown |
| `WITH RECURSIVE … CYCLE col SET is_cycle USING path` | cycle-safe recursion | trees/graphs |
| `CREATE TEMP TABLE … AS …; ANALYZE` | materialize with stats | big, reused intermediate |
| `EXPLAIN (ANALYZE, BUFFERS)` | see `CTE Scan` vs inlined | verification |
| `work_mem` | tuplestore memory | large materializations |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Inlined CTE | the body is expensive and referenced many times |
| `MATERIALIZED` | outer filter would drastically cut the body's rows |
| Temp table | single-statement atomicity/readability is required |
| Subquery in `FROM` | you want the same semantics with less ceremony |
| Data-modifying CTE | you need ordering guarantees between sibling modifications (all see the same snapshot) |

### 9. Common mistakes
- Assuming CTEs are always fences (old advice) → needless `OFFSET 0` hacks → trust PG ≥ 12 inlining and test.
- Assuming CTEs are always inlined → expensive body recomputed or materialized → read the plan.
- Joining large materialized CTEs without stats → bad join method → temp table + `ANALYZE`.
- Two sibling data-modifying CTEs touching the same row → undefined effect → don't.
- Forgetting `UNION ALL` vs `UNION` in recursion → infinite loop or needless dedup → choose deliberately and add depth guard.

### 10. 🎯 Interview kit
**30-second answer:** "In PG 12+ a side-effect-free CTE referenced once is inlined; otherwise it's materialized once. `MATERIALIZED`/`NOT MATERIALIZED` override. Materialization blocks predicate pushdown and gives the planner no stats on the result, but it's useful to compute something expensive once."
**Structure a longer answer:** history (fence) → current rules → effect on plan → when to force each → alternatives (subquery, temp table).
**Likely follow-ups:** *Recursive CTE?* → working-table iteration. *Is `WITH … DELETE` atomic?* → single statement, one snapshot. *CTE vs temp table?* → stats and indexes vs convenience.
**Signals interviewers look for:** version awareness, plan reading, not cargo-culting "CTEs are slow".

### 11. Check yourself
- [ ] Predict whether a given CTE is inlined, and verify with `EXPLAIN`.
- [ ] Show a query that gets faster with `NOT MATERIALIZED` and one that benefits from `MATERIALIZED`.
- [ ] Write an archive-and-delete in one statement with a data-modifying CTE.
- [ ] Explain why CTE results lack statistics and what to do.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [What is COALESCE and how is it different from NULLIF?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35896_What_is_COALESCE_and_how_is_it_different_from_NULLIF.md>)
- [Intermediate] [What is a CTE (Common Table Expression) and when would you use WITH?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35930_What_is_a_CTE_Common_Table_Expression_and_when_would_you_use.md>)
- [Intermediate] [What is a database normalization? Explain 1NF, 2NF, and 3NF briefly.](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36240_What_is_a_database_normalization_Explain_1NF_2NF_and_3NF_bri.md>)
- [Intermediate] [What is a view and how is it different from a materialized view?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36351_What_is_a_view_and_how_is_it_different_from_a_materialized_v.md>)
- [Deep] [When would you choose to store a computed value like order total in a column versus calculating it every time you query?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36236_When_would_you_choose_to_store_a_computed_value_like_order_t.md>)
- [Deep] [What is a generated (computed) column with GENERATED ALWAYS AS ... STORED, and give a use case.](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35878_What_is_a_generated_computed_column_with_GENERATED_ALWAYS_AS.md>)
- [Deep] [Explain a recursive CTE and give an example use case like traversing an org hierarchy.](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35941_Explain_a_recursive_CTE_and_give_an_example_use_case_like_tr.md>)
- [Deep] [When would you use a materialized view and how do you keep it fresh?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36354_When_would_you_use_a_materialized_view_and_how_do_you_keep_i.md>)

## Related concepts

- [Recursive CTEs and Materialization](<../../../01_Databases_and_SQL/SQL_Language/CTEs_and_Recursive_Queries/Recursive_CTEs_and_Materialization.md>) *(CTEs and Recursive Queries)*
- [Large Values and TOAST](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Large_Values_and_TOAST.md>) *(SQL Fundamentals)*
- [Statistics and Sargability](<../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Statistics_and_Sargability.md>) *(Execution Plans)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Computed Columns
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page Compression › Computed Columns
- *Data Engineering Design Patterns* — Bartosz Konieczny — A Wrapper in a Table? › Size

