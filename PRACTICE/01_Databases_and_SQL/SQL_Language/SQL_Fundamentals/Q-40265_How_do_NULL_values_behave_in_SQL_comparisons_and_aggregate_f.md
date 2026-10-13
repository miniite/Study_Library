---
type: "practice"
difficulty: "Intermediate"
---

# How do NULL values behave in SQL comparisons and aggregate functions?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ NULL Handling](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/NULL_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question — "How do NULL values behave in SQL comparisons and aggregate functions?" — is probing whether I understand three-valued logic (3VL) and the many places NULL silently changes results. For a backend/fullstack SDE at LEVEL "intermediate," the interviewer is not testing memorization; they want to see that I know where NULL bites in production: WHERE clauses that quietly drop rows, joins that lose data, COUNT/AVG that ignore NULLs, GROUP BY that treats NULLs as equal, and UNIQUE constraints that allow multiple NULLs. A strong answer defines the rules, shows the surprising cases, and ends with the safe idioms every practitioner uses.

Key assumption: standard ANSI SQL semantics (PostgreSQL, Oracle, SQL Server, MySQL all follow these for the core rules; I'll flag vendor quirks where they matter).

First principles: NULL means "unknown," not "empty" or "zero"

NULL is a marker for a missing / unknown value, not a value itself. Because the value is unknown, any comparison with it is also unknown. SQL therefore uses three-valued logic: every predicate evaluates to TRUE, FALSE, or UNKNOWN.

The single most important rule: NULL = NULL is NOT TRUE — it is UNKNOWN. So is NULL <> 5, NULL > 0, and NULL = NULL. You cannot use = or <> to test for NULL. You must use IS NULL / IS NOT NULL, which return proper TRUE/FALSE.

1. Comparisons and the WHERE clause

A WHERE clause keeps a row only when the predicate is TRUE — UNKNOWN rows are discarded, exactly like FALSE. This causes two classic surprises:

WHERE salary > 50000 silently drops every row where salary IS NULL (the comparison is UNKNOWN).
WHERE salary <> 50000 also drops the NULL rows — people expect "everything that isn't 50000," but NULL rows are excluded because NULL <> 50000 is UNKNOWN, not TRUE.

So col = X and col <> X together do not cover all rows; the NULLs fall through both. To include them you must add OR col IS NULL.

2. Boolean logic with UNKNOWN (AND / OR / NOT)

3VL truth tables extend Boolean logic:

TRUE OR UNKNOWN = TRUE (OR short-circuits on TRUE).
FALSE AND UNKNOWN = FALSE (AND short-circuits on FALSE).
TRUE AND UNKNOWN = UNKNOWN; FALSE OR UNKNOWN = UNKNOWN; NOT UNKNOWN = UNKNOWN.

The classic trap: NOT (col = 5). If col IS NULL, col = 5 is UNKNOWN and NOT UNKNOWN is still UNKNOWN — the row is filtered out.

3. NULL in IN / NOT IN — the silent data-loss bug

x IN (1, 2, NULL) is fine, but x NOT IN (1, 2, NULL) almost never behaves as expected. NOT IN expands to x <> 1 AND x <> 2 AND x <> NULL. That last term is UNKNOWN, so the whole AND can never be TRUE — the query returns zero rows whenever the subquery/list contains even one NULL. This is a frequent production bug when the inner query pulls from a nullable column. The fix: use NOT EXISTS (correlated), or add WHERE col IS NOT NULL to the subquery.

4. NULL in JOINs

A join condition a.x = b.x never matches when either side is NULL (equality is UNKNOWN). So NULL keys never join. In an INNER JOIN they're dropped; in a LEFT JOIN the left row survives but gets all-NULL right columns. If you need NULL to match NULL, use a.x IS NOT DISTINCT FROM b.x (Postgres) or a.x <=> b.x (MySQL null-safe equality).

5. Aggregate functions ignore NULL — except COUNT(*)

Every aggregate skips NULL inputs before computing:

COUNT(col) counts only non-NULL values; COUNT(*) counts rows regardless.
SUM, AVG, MIN, MAX ignore NULLs. Crucially, AVG(col) divides by the count of non-NULL values, not the total row count. So AVG over a column with NULLs is not the same as SUM(col)/COUNT(*) — a common reporting discrepancy.
If all inputs are NULL (or the group is empty), SUM, AVG, MIN, MAX return NULL (not 0), while COUNT returns 0. That's why analysts wrap sums in COALESCE(SUM(x), 0).
6. GROUP BY, DISTINCT, ORDER BY, UNION — here NULLs are treated as equal

This is the inconsistency that catches people: for grouping and set operations, SQL treats all NULLs as the same value.

GROUP BY col puts all NULL rows into one group.
SELECT DISTINCT col returns one NULL, not many.
UNION (distinct) collapses duplicate NULLs.
ORDER BY places NULLs together — either first or last. Default ordering differs by vendor (Postgres/Oracle: NULLs last on ASC by default; MySQL/SQL Server: NULLs first). Use ORDER BY col NULLS LAST to be explicit.
7. Constraints and expressions
UNIQUE allows multiple NULLs in most engines (two NULLs aren't "equal," so they don't violate uniqueness) — a subtle way duplicate-looking rows sneak in. (SQL Server's default UNIQUE index allows only one NULL — a vendor difference.)
CHECK (col > 0) passes for a NULL because the check is UNKNOWN, and CHECK only rejects on FALSE. So a CHECK doesn't enforce anything on NULLs unless you also add NOT NULL.
String/arithmetic with NULL propagates NULL: 5 + NULL = NULL, 'a' || NULL = NULL (except MySQL/SQL Server CONCAT, which ignores NULLs). NULL in a CASE follows the same 3VL rules per branch.
Safe idioms every practitioner should know
Test membership of NULL with IS NULL / IS NOT NULL, never = NULL.
Replace NULL for display/math with COALESCE(col, default) (ANSI) or ISNULL/NVL (vendor). COALESCE returns the first non-NULL of its args.
NULLIF(a, b) returns NULL when a = b — handy to avoid divide-by-zero: x / NULLIF(y, 0).
Null-safe equality: IS NOT DISTINCT FROM (Postgres/ANSI) or <=> (MySQL).
Prefer NOT EXISTS over NOT IN when the inner set may contain NULLs.
Summary — what a strong answer emphasizes

NULL means unknown, so SQL uses three-valued logic where any comparison to NULL is UNKNOWN, and WHERE/JOIN/CHECK keep only TRUE rows. That silently drops rows in <>, NOT IN, and joins. Aggregates ignore NULLs (so AVG divides by non-NULL count and empty sums return NULL, not 0), while COUNT(*) counts every row. The one place NULLs are treated as equal is GROUP BY / DISTINCT / UNION / ORDER BY. And UNIQUE allows multiple NULLs. The senior signal is knowing the fixes: IS NULL, COALESCE, NULLIF, NOT EXISTS, and null-safe equality.

Common follow-up probes: "Why does NOT IN return no rows?" (NULL in the list), "How do you count NULLs in a column?" (COUNT(*) - COUNT(col) or SUM(CASE WHEN col IS NULL THEN 1 ELSE 0 END)), and "Difference between COUNT(col) and COUNT(*)?" (NULL handling).

**Comparison:**

```
Context	How NULL behaves	Consequence / fix
= NULL, <> NULL, > NULL	Result is UNKNOWN	Use IS NULL / IS NOT NULL
WHERE / JOIN ON / CHECK	Keeps rows only when TRUE; UNKNOWN dropped	NULL keys never match; add OR col IS NULL
NOT IN (… NULL …)	Whole predicate UNKNOWN → 0 rows	Use NOT EXISTS or filter NULLs out
COUNT(col), SUM, AVG, MIN, MAX	Ignore NULL inputs	AVG divides by non-NULL count
SUM/AVG/MIN/MAX over all-NULL/empty	Return NULL	Wrap in COALESCE(…, 0)
COUNT(*) / COUNT(1)	Count every row	NULL-independent
GROUP BY, DISTINCT, UNION, ORDER BY	NULLs treated as equal/together	One NULL group; ORDER BY … NULLS LAST
UNIQUE constraint	Multiple NULLs allowed (most DBs)	SQL Server allows only one — vendor quirk
Arithmetic / concatenation	Propagates NULL	5 + NULL = NULL; use COALESCE/CONCAT
```

**Code:**

```sql
-- 1. NULL comparisons never return TRUE:
SELECT (NULL = NULL), (NULL <> 5), (NULL > 0);   -- all UNKNOWN (shown as NULL)

-- Correct NULL test:
SELECT * FROM employees WHERE manager_id IS NULL;      -- top-level employees

-- 2. The "<> misses NULLs" trap and its fix:
SELECT * FROM employees WHERE dept <> 'SALES';                 -- drops NULL depts
SELECT * FROM employees WHERE dept <> 'SALES' OR dept IS NULL; -- includes them

-- 3. NOT IN with a NULL returns ZERO rows -> use NOT EXISTS instead:
-- BAD (if any inactive_id is NULL, result is empty):
SELECT * FROM users u
WHERE u.id NOT IN (SELECT inactive_id FROM banned);
-- GOOD:
SELECT * FROM users u
WHERE NOT EXISTS (SELECT 1 FROM banned b WHERE b.inactive_id = u.id);

-- 4. Aggregates ignore NULL; COUNT(*) does not:
SELECT COUNT(*)            AS all_rows,      -- counts every row
       COUNT(bonus)        AS with_bonus,    -- non-NULL bonus only
       AVG(bonus)          AS avg_over_nonnull, -- divides by with_bonus
       COALESCE(SUM(bonus),0) AS total_bonus  -- 0 instead of NULL if all NULL
FROM employees;

-- 5. Count how many NULLs a column has:
SELECT COUNT(*) - COUNT(bonus) AS null_bonus_count FROM employees;

-- 6. Null-safe helpers:
SELECT revenue / NULLIF(orders, 0) AS avg_order_value FROM sales; -- avoids /0
SELECT COALESCE(nickname, first_name, 'Unknown') AS display_name FROM people;
```

**Example input / output:**

```text
Using the `employees` sample above:

- `SELECT COUNT(*) FROM employees;`            → **4**  (all rows)
- `SELECT COUNT(bonus) FROM employees;`         → **2**  (ids 1,3; NULLs skipped)
- `SELECT SUM(bonus) FROM employees;`           → **8000**
- `SELECT AVG(bonus) FROM employees;`           → **4000** (8000 / 2, NOT / 4)
- `SELECT COUNT(*) FROM employees WHERE dept <> 'SALES';` → **1** (only ENG; NULL-dept row 3 dropped)
- `SELECT COUNT(*) FROM employees WHERE dept <> 'SALES' OR dept IS NULL;` → **2** (ENG + NULL)
- `SELECT DISTINCT dept FROM employees;`        → **SALES, ENG, NULL** (one NULL, not two)
- `SELECT dept, COUNT(*) FROM employees GROUP BY dept;` → SALES=2, ENG=1, **NULL=1** (all NULLs one group)
```

**Data:**

```text
employees
 id | name  | dept   | bonus
----+-------+--------+-------
  1 | Asha  | SALES  |  5000
  2 | Ravi  | SALES  |  NULL
  3 | Meena | NULL   |  3000
  4 | Kabir | ENG    |  NULL
```

**Flow:**

1. col = / <> / > NULL
2. col IS NULL / IS NOT NULL
3. WHERE / JOIN / CHECK
4. Aggregate input
5. GROUP BY / DISTINCT /UNION
6. Predicate involves NULL?
7. Result = UNKNOWN
8. Result = TRUE or FALSE
9. Where is it used?
10. Row dropped / no match /check passes
11. Value skipped bySUM/AVG/COUNTcol/MIN/MAX
12. All NULLs treated as one equalvalue
13. Row kept/filtered as expected

