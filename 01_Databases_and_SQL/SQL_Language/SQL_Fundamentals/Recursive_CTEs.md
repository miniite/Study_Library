---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Recursive_CTEs"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: ["MySQL", "Oracle", "SQL_Server"]
---

# Recursive CTEs

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A CTE names a subquery for reuse and readability; a *recursive* CTE repeatedly joins its own previous output to an anchor query, which is how SQL walks hierarchies and graphs (org charts, bill of materials, trace trees) — with guards for depth and cycles.

### 1. The big picture
A **Common Table Expression** (`WITH name AS (…)`) lets you give a query a name and use it like a table inside one statement. Non-recursive CTEs make multi-step logic readable (extract → clean → aggregate) and avoid deeply nested subqueries. A **recursive CTE** goes further: it references itself, so SQL can follow parent–child links to arbitrary depth without knowing the depth in advance.

Use cases: employee → manager chains, category trees, bills of materials, folder paths, friends-of-friends, call/trace trees (`parent_span_id`), date or number generation.

### 2. Core concepts and vocabulary
- **CTE** — named temporary result set scoped to a single statement.
- **Chaining** — several CTEs in one `WITH`, each able to reference earlier ones.
- **Anchor member** — the non-recursive query that seeds the recursion (e.g., the CEO, root nodes).
- **Recursive member** — query that joins the CTE to the base table to find the next level.
- **Termination** — recursion stops when an iteration returns no new rows.
- **Depth/level** — counter you add to know how deep a row is.
- **Path** — accumulated string/array of ancestors; used for ordering and cycle detection.
- **Cycle** — data where a node is its own ancestor; causes infinite recursion.
- **MAXRECURSION** — SQL Server's safety limit (default 100).
- **Adjacency list** — tree stored as `(id, parent_id)`; the typical input to recursion.

### 3. How it works, step by step
Syntax shape (ANSI; SQL Server omits the word `RECURSIVE`):
```sql
WITH RECURSIVE tree AS (
    SELECT id, parent_id, name, 0 AS depth, CAST(name AS text) AS path   -- anchor
    FROM   org WHERE parent_id IS NULL
  UNION ALL
    SELECT c.id, c.parent_id, c.name, t.depth + 1, t.path || ' > ' || c.name  -- recursive
    FROM   org c JOIN tree t ON c.parent_id = t.id
)
SELECT * FROM tree;
```
Evaluation:
1. Run the **anchor**; its rows form the initial working set and the result.
2. Run the **recursive member** using only the rows produced by the *previous iteration* as `tree`.
3. Append new rows to the result; they become the next working set.
4. Repeat until an iteration yields zero rows.
5. Return the accumulated result.

Rules: the recursive member must reference the CTE exactly once; the anchor and recursive parts are combined with `UNION ALL` (use `UNION` to dedupe, supported in PG/MySQL, not in SQL Server); column types must match between anchor and recursive parts (cast the path column in the anchor to a wide enough type).

### 4. Worked example
```sql
-- Employee hierarchy with level and management chain (SQL Server)
WITH org AS (
    SELECT e.emp_id, e.name, e.manager_id,
           0 AS lvl,
           CAST(e.name AS varchar(4000)) AS chain
    FROM   dbo.Employees e
    WHERE  e.manager_id IS NULL
  UNION ALL
    SELECT e.emp_id, e.name, e.manager_id,
           o.lvl + 1,
           CAST(o.chain + ' > ' + e.name AS varchar(4000))
    FROM   dbo.Employees e
    JOIN   org o ON e.manager_id = o.emp_id
    WHERE  o.lvl < 20                        -- depth guard
)
SELECT emp_id, name, lvl, chain
FROM   org
ORDER  BY chain
OPTION (MAXRECURSION 50);                    -- hard stop, errors if exceeded
```
Reading: level 0 is the CEO; each pass finds direct reports of the previous pass. Ordering by `chain` puts children directly under their manager.

**Cycle detection** (PostgreSQL):
```sql
WITH RECURSIVE walk AS (
    SELECT id, parent_id, ARRAY[id] AS visited, false AS is_cycle
    FROM nodes WHERE id = 1
  UNION ALL
    SELECT n.id, n.parent_id, w.visited || n.id, n.id = ANY(w.visited)
    FROM   nodes n JOIN walk w ON n.parent_id = w.id
    WHERE  NOT w.is_cycle
)
SELECT * FROM walk;   -- or PG14+: ... CYCLE id SET is_cycle USING path
```
In SQL Server, carry a path string and test `CHARINDEX('|' + CAST(n.id AS varchar) + '|', w.path) = 0`.

**Walking upward** (all ancestors of one employee): anchor on that employee and join `e.emp_id = o.manager_id` — same pattern, reversed direction.

**Plain CTE chaining**:
```sql
WITH cleaned AS (SELECT id, LOWER(TRIM(email)) AS email FROM raw_users),
     deduped AS (SELECT DISTINCT email FROM cleaned)
SELECT COUNT(*) FROM deduped;
```

### 5. Under the hood
- **Evaluation is iterative, not truly recursive:** engines keep a *work table* (SQL Server's spool, PG's WorkTable Scan) and loop; plans show a *Recursive Union* (PG) or *Table Spool / Index Spool* with a nested loops join (SQL Server). Each iteration is a join of the work table with the base table — so index `parent_id` for good performance.
- **CTE materialization:** in SQL Server, non-recursive CTEs are *inlined* like views (referenced twice → computed twice). PostgreSQL ≤11 always materialized CTEs (an optimization fence); since PG12 they inline unless referenced more than once or marked `MATERIALIZED`. Don't assume a CTE is computed once.
- **A CTE is not a performance feature:** it doesn't cache results across statements; if you need reuse use a temp table (which also gets statistics).
- **Recursion limits:** SQL Server default `MAXRECURSION 100` (0 = unlimited); PG has none but memory/time bound; MySQL `cte_max_recursion_depth` (default 1000).
- **Scale:** each level multiplies by branching factor; graphs (friends-of-friends) explode quickly — depth 4–5 over millions of users is billions of rows and duplicates. Use depth caps, dedupe by visited set, or a graph database / precomputed closure table.
- **Alternatives for hierarchies:** `hierarchyid` (SQL Server), `ltree` (PG), nested sets, **closure table** (all ancestor–descendant pairs; fast reads, heavier writes), materialized path (`/1/4/9/`).
- **Recursive CTE also generates series:** `WITH n AS (SELECT 1 AS i UNION ALL SELECT i+1 FROM n WHERE i<100)`.

### 6. Performance and design implications
- Index the join key (`parent_id`) and keep the working set narrow (carry only needed columns).
- Always include a depth limit and/or cycle check on data you don't fully control.
- Filter as early as possible (anchor on the subtree you need rather than computing the whole forest and filtering afterward).
- Avoid string-path accumulation for very deep/wide trees (long strings are costly); use arrays or numeric materialized paths.
- For read-heavy hierarchies queried constantly, precompute a closure table or materialized path instead of recursing per request.
- Check plans for row counts per iteration; a missing index shows as a scan per level.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `WITH … AS (…)` | Non-recursive CTE | Readable multi-step queries |
| `WITH RECURSIVE` (SQL Server: `WITH`) | Recursion | Trees, graphs, series |
| `OPTION (MAXRECURSION n)` | Cap iterations (SQL Server) | Safety |
| `CYCLE … SET … USING …` (PG14+) | Built-in cycle detection | Graph traversal |
| `MATERIALIZED` / `NOT MATERIALIZED` (PG) | Control CTE fencing | Plan tuning |
| `cte_max_recursion_depth` (MySQL) | Depth cap | Safety |
| `hierarchyid` / `ltree` | Native tree types | Heavy hierarchy workloads |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| CTE for readability | You need to reuse the result across statements (temp table) |
| Recursive CTE for moderate-depth trees | Huge or dense graphs (graph DB) |
| Closure table / materialized path | Hierarchy changes constantly |
| Depth/cycle guards | Data is guaranteed acyclic and shallow (still cheap to keep) |
| Temp table + index | Intermediate result is big and joined repeatedly |

### 9. Common mistakes
- No termination/cycle guard → infinite loop or max-recursion error → depth counter + visited path.
- Type mismatch between anchor and recursive column (e.g., path `varchar(10)` in anchor) → truncation error → cast to a wide type.
- Believing CTE results are cached → computed multiple times → temp table when reused.
- Missing index on `parent_id` → scan per level → index it.
- Using `UNION` where `UNION ALL` is required (or vice versa) → wrong or slow results → choose deliberately.
- Computing whole hierarchy then filtering → wasted work → filter anchor.

### 10. 🎯 Interview kit
**30-second answer:** A CTE names a subquery for readability; a recursive CTE has an anchor query plus a recursive member joined to itself, iterating until no new rows. I use it to walk manager hierarchies or trace trees, adding a level counter, a path, and a depth cap or cycle check to prevent infinite recursion.
**Structure a longer answer:** non-recursive vs recursive → anchor/recursive/termination → example (org chart with level and path) → guards (MAXRECURSION, visited array) → performance (index on parent, closure tables, graph DB for big graphs).
**Likely follow-ups:**
- *CTE vs subquery vs temp table?* → CTE is syntax/readability (usually inlined); temp tables materialize with stats.
- *How to avoid infinite recursion on cyclic data?* → depth limit plus visited-path check; fix the data.
- *Friends-of-friends at scale?* → bounded depth with dedupe; beyond that, graph DB or precomputed adjacency.
- *Compute the path from root for each employee?* → accumulate `chain || ' > ' || name` in recursion.
**Signals interviewers look for:** correct structure, safety guards, awareness of materialization and scale limits, alternatives for hierarchies.

### 11. Check yourself
- [ ] Write a recursive CTE listing all reports (direct and indirect) of one manager with depth.
- [ ] Add cycle detection in your engine of choice.
- [ ] Explain how the engine evaluates iteration 3.
- [ ] Say when a CTE is computed once vs multiple times.
- [ ] Name two non-recursive ways to store hierarchies for faster reads.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../../../TOOLS/MySQL/_index.md>)
- [Oracle](<../../../TOOLS/Oracle/_index.md>)
- [SQL Server](<../../../TOOLS/SQL_Server/_index.md>)

## Practice questions

- [Basic] [What is a CTE, and how would you chain multiple CTEs to express an extract-clean-aggregate transformation readably?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40026_What_is_a_CTE_and_how_would_you_chain_multiple_CTEs_to_expre.md>)
- [Basic] [What is the difference between a subquery and a CTE, and how would a CTE make a multi-step RAG-analytics query more readable?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40810_What_is_the_difference_between_a_subquery_and_a_CTE_and_how_.md>)
- [Intermediate] [What is a Common Table Expression (CTE) and how is it different from a subquery?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40320_What_is_a_Common_Table_Expression_CTE_and_how_is_it_differen.md>)
- [Deep] [How would a recursive CTE walk an employee-manager hierarchy and compute each employee's depth level?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40218_How_would_a_recursive_CTE_walk_an_employee-manager_hierarchy.md>)
- [Deep] [How would you store and query a hierarchical / tree structure (like an org chart) in SQL?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40797_How_would_you_store_and_query_a_hierarchical_tree_structure_.md>)
- [Deep] [Explain when you'd reach for a recursive CTE with cycle detection, walk through a bill-of-materials explosion, and describe how the recursion could blow up memory on bad data.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40352_Explain_when_you_d_reach_for_a_recursive_CTE_with_cycle_dete.md>)
- [Deep] [Explain how you'd reconstruct a full agent trace tree from a spans table using a recursive CTE on parent_span_id, and how you cap depth and detect cycles from a buggy planner loop.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40841_Explain_how_you_d_reconstruct_a_full_agent_trace_tree_from_a.md>)
- [Expert] [What guards do you need in a recursive CTE to avoid infinite recursion on cyclic data?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40219_What_guards_do_you_need_in_a_recursive_CTE_to_avoid_infinite.md>)

## Related concepts

- [Recursive CTEs and Materialization](<../CTEs_and_Recursive_Queries/Recursive_CTEs_and_Materialization.md>) *(CTEs and Recursive Queries)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › Advanced SELF JOIN: Detecting Hierarchies beyond One Level
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › Derived Tables and Common Table Expressions (CTEs)
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › Common Table Expressions (CTEs)
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 4: Joins, Set Operations, and Subqueries › Recursive CTEs
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Building on Subqueries, Common Table Expressions, and Unions › 6. Building on Subqueries, Common Table Expressions, and Unions
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Building on Subqueries, Common Table Expressions, and Unions › Using Derived Tables and Common Table Expressions
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Building on Subqueries, Common Table Expressions, and Unions › Using Common Table Expressions
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Writing Advanced Queries › Using Multiple CTEs
- *Ultimate AWS Data Engineering* — Rathish Mohan — Data Warehousing Demystified › Common Table Expressions (CTEs): Reusable Building Blocks
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Table expressions
- *Practical Graph Structures in SQL Server and Azure SQL* — Louis Davidson — Tree Data Structures › Reparenting Nodes
- *SQL Server Tutorial* (YouTube series) — Part 51: Recursive CTE in sql server ([watch](https://www.youtube.com/watch?v=GGoV0wTMCg0))

