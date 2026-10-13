---
type: "practice"
difficulty: "Intermediate"
---

# What is a Common Table Expression (CTE) and how is it different from a subquery?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Recursive CTEs](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Recursive_CTEs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question asks what a Common Table Expression (CTE) is and how it differs from a subquery. This is an "intermediate experience" question for a backend/fullstack SDE, so the interviewer expects a practitioner-level answer: the correct definition, the practical differences (readability, reuse, recursion, materialization behavior), and enough nuance to show I've actually used both and know when each matters — not just a syntax comparison.

What they're really probing: Do I know that a CTE and a subquery are often logically equivalent, so the real differences are about structure/reuse/recursion and, on some engines, optimization/materialization? Do I know the one thing a subquery cannot do (recursion)? Assumption: standard SQL, primarily PostgreSQL / MySQL 8+ / SQL Server / Oracle (all support CTEs via the WITH keyword; MySQL added them in 8.0).

Plan: define a CTE, define a subquery, put them side by side on the same task, then enumerate the meaningful differences (reuse, readability, recursion, materialization/performance), and close with a decision rule and gotchas.

What a CTE is

A CTE is a named, temporary result set defined with the WITH keyword that exists only for the duration of the single statement that follows it. You write WITH cte_name AS ( SELECT ... ) and then reference cte_name in the main query as if it were a table. You can define multiple CTEs in one WITH clause (comma-separated), and later CTEs can reference earlier ones — which lets you build a query as a readable, top-down pipeline of named steps instead of a deeply nested blob.

SQL
WITH high_value AS (
    SELECT customer_id, SUM(amount) AS total
    FROM orders
    GROUP BY customer_id
    HAVING SUM(amount) > 10000
)
SELECT c.name, h.total
FROM high_value h
JOIN customers c ON c.id = h.customer_id;
What a subquery is

A subquery is a query nested inside another query — in the FROM clause (a "derived table"/inline view), in SELECT (a scalar subquery returning one value), or in WHERE/HAVING (via IN, EXISTS, = (SELECT ...)). It has no name that the outer query addresses by identifier the way a CTE does (a derived table gets an alias, but it lives inline). The same task as above with a derived table:

SQL
SELECT c.name, h.total
FROM (
    SELECT customer_id, SUM(amount) AS total
    FROM orders
    GROUP BY customer_id
    HAVING SUM(amount) > 10000
) AS h
JOIN customers c ON c.id = h.customer_id;

Both queries produce the identical result. So the difference is not correctness — it's structure, reuse, capability, and sometimes performance.

The meaningful differences

1. Readability & structure. A CTE reads top-to-bottom like named steps in a pipeline; subqueries read inside-out and get hard to follow when nested 2–3 deep. For anything non-trivial, CTEs are dramatically more maintainable. This is the everyday reason practitioners reach for WITH.

2. Reuse within the same statement. A CTE is defined once and referenced multiple times in the same query. A subquery must be repeated everywhere it's needed. If you need the same derived set in a self-join or in two branches of a UNION, a CTE is written once; the subquery would be duplicated (and potentially executed twice). This is a genuine capability and often a performance win.

3. Recursion — the thing only a CTE can do. A recursive CTE (WITH RECURSIVE) can reference itself, which lets you traverse hierarchies and graphs: org charts, category trees, bill-of-materials, "all managers above this employee." A subquery cannot be recursive. If the interviewer wants the one capability difference, this is it.

4. Materialization / optimization behavior (the nuanced part). This differs by engine and is where seniority shows:

In PostgreSQL before v12, CTEs were an optimization fence — always materialized into a temp result, and the planner could not push predicates through them. That could make a CTE slower than the equivalent subquery. From PG 12+, non-recursive, single-reference, side-effect-free CTEs are inlined by default (behaving like a subquery), and you can force behavior with MATERIALIZED / NOT MATERIALIZED.
In SQL Server and MySQL 8, CTEs are generally inlined/expanded and optimized like derived tables — usually no performance penalty vs. a subquery.
Takeaway: on modern engines a CTE and a subquery usually optimize the same; the historical "CTEs are always slower/faster" claims are engine- and version-specific. Verify with EXPLAIN.

5. Scope. A CTE is visible to the entire statement (and to later CTEs in the same WITH). A subquery is visible only where it's written. Correlated subqueries can reference the outer query's columns (row-by-row), which a plain CTE cannot do the same way — so correlated EXISTS/IN subqueries remain the right tool for row-level existence checks.

When to use which

Use a CTE for: multi-step transformations you want to name and read top-down; a derived set referenced more than once; recursion/hierarchy traversal; breaking a giant query into reviewable stages. Use a subquery for: a small, single-use inline value or filter (scalar subquery in SELECT, EXISTS/IN in WHERE); correlated row-by-row checks; cases where you specifically want the planner to inline and don't need a name.

Summary — what a strong answer emphasizes

A CTE (WITH name AS (...)) is a named temporary result set scoped to one statement; a subquery is a query nested inline in another. For most single-use cases they're logically equivalent and modern optimizers treat them alike. CTEs win on readability, reuse within a statement, and are the only way to do recursion (WITH RECURSIVE). Subqueries win for small inline/scalar uses and correlated existence checks. The materialization behavior is engine/version-specific (notably PG ≤11 fenced CTEs; PG 12+ inlines), so measure with EXPLAIN rather than assuming.

#### Common mistakes
claiming CTEs are "always faster" (false — engine-dependent); thinking a subquery can recurse (it can't); duplicating a subquery instead of using a CTE for reuse; forgetting a CTE only lives for the single statement it prefixes.

#### Follow-up
probes: "Write a recursive CTE for an employee→manager hierarchy" or "your CTE got slow on Postgres 11 — why, and how do you fix it?" (answer: optimization fence; upgrade or restructure / use NOT MATERIALIZED on 12+).

**Comparison:**

```
Aspect	CTE (WITH)	Subquery
Named?	Yes — referenced by name	No (derived table has an inline alias only)
Reuse in same statement	Yes — define once, use many times	No — must be repeated
Readability of complex queries	High — top-down named steps	Low — nested inside-out
Recursion (WITH RECURSIVE)	Yes (only CTEs)	No
Correlated (row-by-row) reference	No (plain CTE)	Yes (EXISTS/IN/scalar)
Scope/lifetime	The single following statement	Only where it is written
Materialization	Engine/version-specific (PG ≤11 fenced; PG 12+ inlined; SQL Server/MySQL inlined)	Usually inlined/optimized as derived table
Best for	Multi-step pipelines, reuse, hierarchy traversal	Small inline values, existence checks, single-use filters
```

**Code:**

```sql
-- Same result, two styles ---------------------------------------

-- (a) CTE: named, readable, reusable
WITH order_totals AS (
    SELECT customer_id, SUM(amount) AS total
    FROM orders
    GROUP BY customer_id
)
SELECT c.name, t.total
FROM order_totals t
JOIN customers c ON c.id = t.customer_id
WHERE t.total > 10000;

-- (b) Subquery (derived table): inline, single-use
SELECT c.name, t.total
FROM (
    SELECT customer_id, SUM(amount) AS total
    FROM orders
    GROUP BY customer_id
) AS t
JOIN customers c ON c.id = t.customer_id
WHERE t.total > 10000;

-- Reuse: CTE referenced twice, written once (self-join on the derived set)
WITH totals AS (
    SELECT customer_id, SUM(amount) AS total FROM orders GROUP BY customer_id
)
SELECT a.customer_id, a.total, AVG(b.total) AS peer_avg
FROM totals a
CROSS JOIN totals b            -- reuses `totals` without repeating the SELECT
GROUP BY a.customer_id, a.total;

-- Recursion: ONLY a CTE can do this (employee -> manager chain)
WITH RECURSIVE chain AS (
    SELECT id, name, manager_id, 1 AS depth
    FROM employees
    WHERE id = 42                       -- anchor: start employee
    UNION ALL
    SELECT e.id, e.name, e.manager_id, c.depth + 1
    FROM employees e
    JOIN chain c ON e.id = c.manager_id -- recursive: climb to the manager
)
SELECT * FROM chain ORDER BY depth;

-- Correlated subquery: a job for subqueries, not a plain CTE
SELECT c.name
FROM customers c
WHERE EXISTS (SELECT 1 FROM orders o
              WHERE o.customer_id = c.id AND o.amount > 5000);

-- PostgreSQL 12+: control materialization explicitly
WITH t AS NOT MATERIALIZED (SELECT * FROM big_table WHERE flag = true)
SELECT * FROM t WHERE id = 7;   -- lets the planner push the id=7 predicate in
```

**Example input / output:**

```

Recursive CTE input: employees table above, anchor id = 42.

Output (climbing the management chain):

CODE
Copy
id | name | manager_id | depth
---+------+------------+------
42 | Dev  |     20     |   1
20 | Mgr  |      5     |   2
 5 | VP   |      1     |   3
 1 | CEO  |   NULL     |   4

The CTE starts at Dev (depth 1), then repeatedly joins each row to its manager, adding one level of depth per iteration until it reaches the CEO (manager_id NULL). A subquery cannot express this loop.
```

**Data:**

```text
employees table (for the recursive example)
id | name  | manager_id
---+-------+-----------
1  | CEO   |   NULL
5  | VP    |    1
20 | Mgr   |    5
42 | Dev   |   20
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Need a derived result inside aquery
8. Recursion / hierarchy?
9. CTE required: WITHRECURSIVE
10. Referenced more than onceor query hard to read?
11. Use a CTE - name and reuse it
12. Row-by-row correlated check?
13. Correlated subquery: EXISTS /IN
14. Simple inline subquery is fine

