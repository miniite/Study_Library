---
type: "practice"
difficulty: "Basic"
---

# What is the difference between WHERE and HAVING clauses in SQL?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1 · [↑ WHERE vs HAVING](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/WHERE_vs_HAVING.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question asks for the difference between the WHERE and HAVING clauses in SQL. Both filter rows, so on the surface they look interchangeable — but they operate at different stages of query execution. The interviewer is probing whether I understand SQL's logical processing order: specifically that WHERE filters individual rows before grouping/aggregation, while HAVING filters groups after GROUP BY and aggregation. A strong candidate connects this to why you can't put an aggregate like SUM() in WHERE, and why filtering early with WHERE is more efficient.

#### Assumptions
standard SQL semantics (works the same in MySQL, PostgreSQL, SQL Server, Oracle). Plan: state the core rule, walk through the logical query-processing order to explain why the difference exists, give a concrete example that needs both clauses, cover performance, then summarize with common mistakes.

Core rule
WHERE filters rows — it runs before GROUP BY and aggregation. It can reference columns of the base table but cannot reference aggregate functions (SUM, COUNT, AVG, MAX, MIN), because at that point the groups don't exist yet.
HAVING filters groups — it runs after GROUP BY and aggregation. It is designed to reference aggregate functions (and the grouping columns). You use it to answer "keep only the groups whose aggregate satisfies a condition," e.g., "departments with more than 5 employees."
Why — the logical processing order

SQL clauses don't execute top-to-bottom as written; there's a defined logical order. Knowing it makes the WHERE/HAVING distinction obvious:

FROM / JOIN — assemble the source rows.
WHERE — filter individual rows (no aggregates allowed here).
GROUP BY — collapse the surviving rows into groups.
HAVING — filter the groups (aggregates allowed and expected).
SELECT — compute the output columns / aggregates.
ORDER BY — sort the final result.
LIMIT / TOP — cut the result set.

Because WHERE (step 2) runs before GROUP BY (step 3), it literally has no groups or aggregate values to test — that's the technical reason WHERE COUNT(*) > 5 is a syntax error. HAVING (step 4) runs after the groups and their aggregates are formed, so it can test them. Also note column aliases defined in SELECT (step 5) generally aren't visible to WHERE or HAVING since those run earlier (engine-dependent; MySQL is lenient with HAVING).

Concrete example (needs both)

"Among orders placed in 2024, find customers who spent more than ₹1,00,000 total."

The 2024 condition filters rows → belongs in WHERE (drop non-2024 rows before grouping — cheaper and correct).
The total > ₹1,00,000 condition tests an aggregate per customer → belongs in HAVING (after GROUP BY customer_id).
SQL
SELECT customer_id, SUM(amount) AS total_spend
FROM   orders
WHERE  order_date >= '2024-01-01' AND order_date < '2025-01-01'  -- rows, pre-group
GROUP  BY customer_id
HAVING SUM(amount) > 100000                                       -- groups, post-agg
ORDER  BY total_spend DESC;

If I mistakenly tried WHERE SUM(amount) > 100000, the query fails. If I moved the date filter into HAVING (e.g., HAVING order_date >= ...), it's wrong/invalid because after grouping there's no single order_date per group.

Performance

WHERE is generally more efficient for anything that can be expressed on raw rows, because it shrinks the row set before the expensive GROUP BY/aggregation step, and it can use indexes on the base columns. HAVING runs on the already-aggregated result, so pushing a predicate into WHERE when possible reduces work. Rule of thumb: filter as early as possible — put row conditions in WHERE, and reserve HAVING strictly for conditions on aggregates.

A subtle case: a query with no GROUP BY but a HAVING treats the whole table as one group — legal but rare. And a HAVING that references only non-aggregate columns could have been a WHERE — that's a code smell (slower, less clear).

#### Common mistakes
/ follow-up probes
Putting an aggregate in WHERE (e.g., WHERE COUNT(*) > 1) → error; must be HAVING.
Putting a plain row filter in HAVING when it belongs in WHERE → works but slower and misleading.
Thinking HAVING requires GROUP BY — it doesn't strictly, but it's almost always used with it.

#### Follow-up
"Can you use both in one query?" → Yes, and you often should (row filter in WHERE, aggregate filter in HAVING). "Why can't WHERE see aggregates?" → because it executes before grouping.

Summary — what a strong answer emphasizes: WHERE filters rows before grouping (no aggregates); HAVING filters groups after aggregation (aggregates allowed). Ground it in the logical processing order (FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY), show a query using both correctly, and note that filtering early with WHERE is faster because it reduces rows before aggregation.

**Comparison:**

```
Aspect	WHERE	HAVING
Filters	Individual rows	Groups
Execution stage	Before GROUP BY / aggregation	After GROUP BY / aggregation
Can use aggregate functions	No	Yes
Can reference plain columns	Yes	Only grouping columns (or via aggregate)
Requires GROUP BY	No	Almost always used with it
Performance	Reduces rows early (can use indexes)	Runs on aggregated result
```

**Code:**

```sql
-- WHERE filters rows (pre-group); HAVING filters groups (post-aggregation)
SELECT   department_id, COUNT(*) AS headcount, AVG(salary) AS avg_sal
FROM     employees
WHERE    status = 'ACTIVE'          -- row-level filter, runs BEFORE grouping
GROUP BY department_id
HAVING   COUNT(*) > 5               -- group-level filter, aggregate allowed here
         AND AVG(salary) > 60000
ORDER BY avg_sal DESC;

-- INVALID: aggregate in WHERE (groups don't exist yet at WHERE-time)
-- SELECT department_id FROM employees WHERE COUNT(*) > 5 GROUP BY department_id;  -- ERROR
```

**Example input / output:**

```text
Query:
  SELECT department_id, COUNT(*) AS headcount, AVG(salary) AS avg_sal
  FROM employees
  WHERE status = 'ACTIVE'
  GROUP BY department_id
  HAVING COUNT(*) >= 2;

Step-by-step (logical order):
  1. FROM       -> 5 rows
  2. WHERE      -> drop id=3 (LEFT); 4 rows remain
  3. GROUP BY   -> dept 10: {id1,id2}, dept 20: {id4,id5}
  4. aggregate  -> dept10: count=2 avg=60000 ; dept20: count=2 avg=47500
  5. HAVING     -> count>=2 keeps both groups
Result:
  department_id | headcount | avg_sal
  10            | 2         | 60000
  20            | 2         | 47500
```

**Data:**

```text
employees
+----+--------------+---------+--------+
| id | department_id| salary  | status |
+----+--------------+---------+--------+
| 1  | 10           | 70000   | ACTIVE |
| 2  | 10           | 50000   | ACTIVE |
| 3  | 10           | 90000   | LEFT   |
| 4  | 20           | 40000   | ACTIVE |
| 5  | 20           | 55000   | ACTIVE |
+----+--------------+---------+--------+
```

**Flow:**

1. FROM / JOIN
2. WHEREfilter ROWS - no aggregates
3. GROUP BYform groups
4. HAVINGfilter GROUPS - aggregates ok
5. SELECTcompute columns/aggregates
6. ORDER BY / LIMIT

