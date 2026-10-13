---
type: "practice"
difficulty: "Intermediate"
---

# When would you use a JOIN versus a subquery, and which is generally faster?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ WHERE vs HAVING](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/WHERE_vs_HAVING.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question is "when do you use a JOIN vs a subquery, and which is generally faster?" At intermediate level, the wrong answer is a blanket "JOINs are always faster." The interviewer is probing whether you understand that (a) many JOINs and subqueries are semantically equivalent and the query optimizer often transforms one into the other, so raw performance depends on the execution plan, indexes, and data distribution, not the syntax; and (b) you still have a mental model of when each is the natural, readable choice and when one has a real edge. A strong answer: clarifies they overlap, gives decision rules, calls out the cases where they genuinely differ (semi-join/anti-join, aggregation, row-multiplication), and closes with "measure with EXPLAIN." Assumption: a modern cost-based optimizer (PostgreSQL, MySQL 8+, SQL Server, Oracle).

First principle: they're often the same query

#### Cons
ider "customers who placed an order." You can write it three ways: IN subquery, EXISTS correlated subquery, or an INNER JOIN + DISTINCT. A good optimizer frequently rewrites all three into the same semi-join and produces an identical plan. So the honest headline is: for equivalent queries, neither is inherently faster — the optimizer usually converges. Performance differences come from indexes, join order, cardinality estimates, and whether the query accidentally does extra work (like row multiplication or DISTINCT).

When a JOIN is the right choice

Use a JOIN when you need columns from both tables in the result. A subquery in WHERE can only filter; it can't project columns from the inner table into the output. If you want the customer name and their order total, you must JOIN.

JOINs are also the natural tool for combining/enriching data (star-schema fact + dimensions), and they're generally the most readable for "give me a wide row." At scale, an equi-JOIN on indexed keys lets the optimizer pick hash-join or merge-join strategies that are very efficient.

When a subquery is the right choice

Use a subquery when you only need to filter or test existence and don't want the inner table's columns:

EXISTS / NOT EXISTS — pure existence test. Often the fastest option for "has at least one match," because it can short-circuit on the first matching row (index seek) instead of materializing all matches. NOT EXISTS is also the correct, NULL-safe way to do an anti-join.
Scalar subquery in SELECT — one computed value per row (e.g., a running count) when a JOIN would complicate the grouping.
Derived table / CTE in FROM — pre-aggregate, then join to the aggregate. This is cleaner and often faster than a correlated subquery that recomputes the aggregate per row.
Readability — a subquery can express "rows above the average" far more clearly than a self-join.
The performance traps (where they genuinely differ)

JOIN + DISTINCT for existence. Writing SELECT DISTINCT c.* FROM customers c JOIN orders o ... to mean "customers with any order" forces the DB to produce all matching pairs and then de-duplicate — expensive if a customer has thousands of orders. EXISTS avoids that entirely. Here the subquery is usually faster.

Row multiplication. A JOIN to a one-to-many table multiplies rows, which can silently inflate SUM/COUNT (the classic "my revenue is doubled because I joined two child tables"). A subquery/derived table that pre-aggregates avoids the fan-out. Here the subquery/CTE is safer and often faster.

NOT IN with NULLs. NOT IN (SELECT col …) returns zero rows if col has any NULL. Use NOT EXISTS (correlated) or a LEFT JOIN ... WHERE right IS NULL instead. This is a correctness issue, not just speed.

Correlated subquery re-execution. A correlated subquery that isn't decorrelated by the optimizer runs per outer row → O(N·M). If you see that in EXPLAIN, rewrite it as a JOIN or a pre-aggregated derived table.

IN with a large/duplicate-heavy list can be slower than a JOIN or EXISTS; the optimizer may or may not deduplicate efficiently.

So — which is "generally faster"?

The defensible interview answer:

For existence checks, EXISTS (subquery) is generally as fast or faster than JOIN + DISTINCT.
For returning columns from multiple tables, a JOIN is the right and usually faster tool.
For aggregation without fan-out, a pre-aggregated derived table/CTE beats both a naive JOIN and a correlated subquery.
For most equivalent, well-indexed cases, the optimizer makes them the same — so measure with EXPLAIN (ANALYZE) rather than guessing.

The meta-point that impresses interviewers: prefer the clearest form, ensure the join/filter keys are indexed, and let the optimizer's plan (verified via EXPLAIN) drive any rewrite. Syntax choice is secondary to indexing and cardinality.

Summary

JOIN when you need data from both tables or are enriching rows; subquery (especially EXISTS) when you only filter or test existence, and derived tables/CTEs when you must pre-aggregate to avoid fan-out. Neither is universally faster — for equivalent queries the optimizer typically converges — but subqueries win for pure existence (no DISTINCT needed) and pre-aggregation (no row multiplication), while JOINs win when you must project inner columns. Always confirm with EXPLAIN ANALYZE and make sure the keys are indexed.

#### Follow-up
probes: "Show the EXPLAIN difference for IN vs EXISTS vs JOIN," "Why did your revenue double after adding a JOIN?" (fan-out), and "How do you do a NULL-safe anti-join?" (NOT EXISTS).

**Comparison:**

```
Need	Best choice	Why
Columns from both tables	JOIN	Subquery in WHERE can't project inner columns
"Has at least one match"	EXISTS subquery	Short-circuits; no DISTINCT; index seek
Anti-join ("has none")	NOT EXISTS	NULL-safe, unlike NOT IN
Aggregate w/o row inflation	Derived table / CTE	Pre-aggregate avoids fan-out double counting
One value per row	Scalar subquery in SELECT	Simpler than reshaping a GROUP BY
Equivalent, indexed filter	Either	Optimizer usually converges — measure
Query form	Typical cost driver	Faster when
JOIN + DISTINCT	producing + deduping all matches	you need matched columns
EXISTS	first-match seek	pure existence, indexed key
IN (subquery)	dedupe/hash of inner set	small/unique inner set
Correlated subquery	per-outer-row re-exec (O(N·M))	optimizer decorrelates / EXISTS
```

**Code:**

```sql
-- GOAL A: "customers who have at least one order" (existence only)

-- Subquery with EXISTS — usually the fastest; short-circuits, no DISTINCT.
SELECT c.customer_id, c.name
FROM customers c
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id);

-- JOIN version needs DISTINCT (dedupe) -> can be slower for heavy children.
SELECT DISTINCT c.customer_id, c.name
FROM customers c
JOIN orders o ON o.customer_id = c.customer_id;

-- GOAL B: "customer name AND their total order amount" -> MUST use a JOIN
-- (a WHERE-subquery cannot project the order columns).
SELECT c.customer_id, c.name, SUM(o.amount) AS total_spent
FROM customers c
JOIN orders o ON o.customer_id = c.customer_id
GROUP BY c.customer_id, c.name;

-- GOAL C: avoid fan-out when a customer has orders AND reviews.
-- Naive JOIN of both children double-counts; pre-aggregate in derived tables.
SELECT c.customer_id,
       COALESCE(o.total, 0)   AS order_total,
       COALESCE(r.cnt, 0)     AS review_count
FROM customers c
LEFT JOIN (SELECT customer_id, SUM(amount) total FROM orders  GROUP BY customer_id) o
       ON o.customer_id = c.customer_id
LEFT JOIN (SELECT customer_id, COUNT(*)    cnt   FROM reviews GROUP BY customer_id) r
       ON r.customer_id = c.customer_id;

-- GOAL D: NULL-safe anti-join ("customers with NO orders").
SELECT c.customer_id
FROM customers c
WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id);

-- Always confirm the actual plan:
EXPLAIN ANALYZE
SELECT c.customer_id FROM customers c
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id);
```

**Example input / output:**

```text
customers: (1,'Aisha'), (2,'Ben'), (3,'Chen')
orders:    (101,1,500),(102,1,300),(103,3,900)

GOAL A (existence): EXISTS and JOIN+DISTINCT both return -> {1 Aisha, 3 Chen}
  but JOIN produces pairs (1,101),(1,102),(3,103) then DEDUPES to 2 rows;
  EXISTS stops at the first order per customer -> less work for Aisha.

GOAL B (JOIN, per-customer total):
  Aisha -> 800, Chen -> 900   (Ben absent: INNER JOIN drops no-order customers)

GOAL D (anti-join, no orders): returns {2 Ben}
```

**Data:**

```text
customers(customer_id PK, name)
orders(order_id PK, customer_id FK -> customers, amount)   -- INDEX orders(customer_id)
reviews(review_id PK, customer_id FK -> customers)         -- INDEX reviews(customer_id)

Key indexing rule for speed regardless of syntax:
  the JOIN / EXISTS correlation column (orders.customer_id) MUST be indexed.
```

**Flow:**

1. Yes
2. No
3. Yes, at least one
4. Yes, none exist
5. No, need an aggregate
6. Yes
7. No
8. Do I need columns from thesecond table in the OUTPUT?
9. Use a JOIN
10. Am I just testing existence?
11. EXISTS subquery
12. NOT EXISTS - NULL-safeanti-join
13. Will a JOIN multiply rows /fan-out?
14. Pre-aggregate in a derivedtable / CTE, then JOIN
15. EXPLAIN ANALYZE + ensurekeys indexed

