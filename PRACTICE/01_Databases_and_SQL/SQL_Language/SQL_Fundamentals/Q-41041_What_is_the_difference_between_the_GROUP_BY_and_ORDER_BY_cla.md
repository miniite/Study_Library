---
type: "practice"
difficulty: "Basic"
---

# What is the difference between the GROUP BY and ORDER BY clauses?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ GROUP BY vs PARTITION BY](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/GROUP_BY_vs_PARTITION_BY.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: Explain how GROUP BY and ORDER BY differ. Because it's a basic concept question, the interviewer wants to confirm you don't confuse two clauses that both take column lists but do completely different jobs: GROUP BY collapses rows into groups (for aggregation), while ORDER BY sorts the rows of the result. A weak candidate says "both arrange data"; a strong one explains what each produces, when each runs, and shows a query using both together.

Plan: Define each clause from first principles, contrast them across the dimensions that matter (purpose, effect on row count, where they sit in logical execution order, whether aggregates are involved), then show a single query that uses both correctly, and close with the classic traps (WHERE vs HAVING, selecting non-grouped columns).

Reaching the answer from first principles

Think about what SQL is doing to a set of rows. A query starts with rows from FROM/JOIN, filters them (WHERE), then optionally groups them, then optionally sorts them, then returns them. GROUP BY and ORDER BY live in two different stages of that pipeline.

GROUP BY — collapse rows into groups for aggregation

GROUP BY col takes all rows that share the same value(s) of col and folds them into one output row per group. It is almost always paired with an aggregate function (COUNT, SUM, AVG, MIN, MAX) that computes a single value across each group. So GROUP BY typically reduces the number of rows: 1,000 orders across 3 regions become 3 rows, one per region.

SQL
SELECT region, COUNT(*) AS num_orders, SUM(amount) AS total
FROM Orders
GROUP BY region;

Here the 1,000 order rows collapse to one row per region. A key rule: every column in the SELECT must either appear in GROUP BY or be inside an aggregate — otherwise the value is ambiguous (which of the many rows in the group should it show?). Standard SQL and most databases reject that; MySQL historically allowed it and returned an arbitrary value, which is a footgun.

To filter groups (after aggregation) you use HAVING, not WHERE. WHERE filters individual rows before grouping; HAVING filters after, on aggregate results (e.g., HAVING COUNT(*) > 10).

ORDER BY — sort the final rows

ORDER BY col [ASC|DESC] does not change how many rows you get; it only changes their sequence. It's the last logical step, applied to whatever rows the query produced (grouped or not). Without it, SQL gives no guaranteed order — the order you happen to see is an implementation detail and can change.

SQL
SELECT name, salary FROM Employees ORDER BY salary DESC;   -- highest paid first

ORDER BY can sort by multiple columns (ORDER BY dept, salary DESC), by column position (ORDER BY 2), or by an expression/alias.

The two together

They compose naturally: group first, then sort the group rows. This is extremely common in reporting:

SQL
SELECT region, SUM(amount) AS total
FROM Orders
WHERE status = 'PAID'      -- filter rows first
GROUP BY region            -- collapse into one row per region
HAVING SUM(amount) > 1000  -- keep only big regions
ORDER BY total DESC;       -- sort those region rows, biggest first

Logical execution order is: FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY. That order explains two facts freshers often miss: WHERE can't reference an aggregate (grouping hasn't happened yet), but ORDER BY can reference a SELECT alias like total (it runs after SELECT).

Summary — what a strong answer emphasizes
Purpose: GROUP BY = aggregate/collapse rows into groups; ORDER BY = sort the output rows.
Effect on row count: GROUP BY usually reduces rows (one per group); ORDER BY never changes the number of rows.
Companionship: GROUP BY is used with aggregate functions and filtered by HAVING; ORDER BY stands alone and just arranges the result.
Order of execution: GROUP BY runs before SELECT; ORDER BY runs last, so it can use SELECT aliases.
They are independent and often combined — group to aggregate, then order to present.

#### Common mistakes
thinking both "arrange" data (only ORDER BY sorts); using WHERE to filter aggregates instead of HAVING; selecting a column that's neither grouped nor aggregated; and assuming rows come back sorted just because you used GROUP BY (they do not — you still need ORDER BY).

#### Follow-up
probes: "What's the difference between WHERE and HAVING?", "Can ORDER BY use a column not in SELECT?" (yes, generally, since it operates on the row source), and "Does GROUP BY guarantee sorted output?" (no — historically some engines side-effected sorting, but you must not rely on it).

**Comparison:**

```
Aspect	GROUP BY	ORDER BY
Purpose	Collapse rows into groups for aggregation	Sort the result rows
Effect on row count	Usually reduces (one row per group)	No change (same rows, reordered)
Used with	Aggregate functions (COUNT/SUM/AVG/MIN/MAX)	Any column/expression; ASC or DESC
Companion filter	HAVING (filters groups, post-aggregation)	none
Logical execution order	Before SELECT	Last (after SELECT) — can use SELECT aliases
Guarantees sorted output?	No	Yes
```

**Code:**

```sql
-- GROUP BY: one row per region, with aggregates
SELECT region, COUNT(*) AS num_orders, SUM(amount) AS total
FROM Orders
GROUP BY region;

-- ORDER BY: same rows, just sorted
SELECT name, salary
FROM Employees
ORDER BY salary DESC;

-- Both together (with row filter WHERE and group filter HAVING)
SELECT region, SUM(amount) AS total
FROM Orders
WHERE status = 'PAID'
GROUP BY region
HAVING SUM(amount) > 1000
ORDER BY total DESC;
```

**Example input / output:**

```text
Using the Orders sample above.

`GROUP BY region` (with SUM) collapses 4 rows into 3:

| region | total |
|--------|-------|
| North  | 1200  |
| South  | 300   |
| East   | 900   |

Then `ORDER BY total DESC` sorts those 3 group-rows (does not add/remove rows):

| region | total |
|--------|-------|
| North  | 1200  |
| East   | 900   |
| South  | 300   |
```

**Data:**

```text
Orders
+----+--------+--------+--------+
| id | region | amount | status |
+----+--------+--------+--------+
|  1 | North  |  500   | PAID   |
|  2 | North  |  700   | PAID   |
|  3 | South  |  300   | PAID   |
|  4 | East   |  900   | PAID   |
+----+--------+--------+--------+
```

**Flow:**

1. FROM / JOIN
2. WHEREfilter rows
3. GROUP BYcollapse into groups
4. HAVINGfilter groups
5. SELECTprojections & aggregates
6. ORDER BYsort final rows

