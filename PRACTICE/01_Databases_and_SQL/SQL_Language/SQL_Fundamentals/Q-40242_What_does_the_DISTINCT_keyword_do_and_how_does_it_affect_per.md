---
type: "practice"
difficulty: "Basic"
---

# What does the DISTINCT keyword do and how does it affect performance?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Reading EXPLAIN](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Reading_EXPLAIN.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question has two parts: what does DISTINCT do, and how does it affect performance. It's tagged basic concept, so the interviewer wants a clear, correct definition with a simple example, plus enough awareness that you don't sprinkle DISTINCT everywhere as a "fix" without understanding its cost. A strong basic answer defines the keyword precisely, shows one example, explains why it costs something (it needs a sort or hash to detect duplicates), and names one or two cheaper alternatives. I'll keep terms defined and avoid hand-waving.

What DISTINCT does

DISTINCT removes duplicate rows from a query's result set, so every returned row is unique. It appears right after SELECT:

SQL
SELECT DISTINCT country FROM Users;

If 10,000 users live in 30 countries, this returns exactly 30 rows — one per unique country — instead of 10,000.

The key rule beginners miss: DISTINCT applies to the entire selected row (all columns in the SELECT list together), not to a single column. So SELECT DISTINCT city, country FROM Users; returns unique (city, country) pairs — "Springfield, USA" and "Springfield, UK" are two different, both-kept rows. It is not "distinct on city." (Some databases like Postgres offer a DISTINCT ON (col) extension for that, but standard DISTINCT is whole-row.)

Conceptually, DISTINCT runs after the rows are produced by FROM/WHERE and the columns are projected — it's a de-duplication pass over the result.

How DISTINCT affects performance

To know whether two rows are duplicates, the database must compare rows against each other. It can't do that for free — it needs one of two strategies:

Sort all the rows so identical rows land next to each other, then discard neighbors that match. Sorting is roughly O(n log n) and, for big result sets, may spill to disk (a temp file / temp table) if it doesn't fit in memory.
Hash aggregate — build a hash table keyed by the row's values and keep only the first occurrence of each key. This is roughly O(n) but needs memory proportional to the number of distinct rows; if that overflows the memory budget it also spills to disk.

Either way, DISTINCT adds work on top of reading the rows: extra CPU (hashing/comparing), extra memory, and possibly disk I/O for spills. On a small result it's negligible; on millions of rows it can dominate the query time. It also can't stream results as eagerly — a sort-based plan must see all rows before it can emit the first unique one.

Indexes help. If there's an index on the DISTINCT column(s), the engine can often read values already in sorted order and do a cheap "skip duplicates" scan, avoiding a separate sort. So SELECT DISTINCT country is much cheaper when country is indexed.

The most important practical warning

DISTINCT is frequently misused to paper over a bad join that produced accidental duplicate rows (e.g., a one-to-many join fanning out rows). Slapping DISTINCT on top hides the symptom but keeps the expensive fan-out, then pays again to de-duplicate it. The correct fix is usually to fix the join (use the right key, a semi-join with EXISTS, or pre-aggregate the child table) so duplicates never appear. DISTINCT should be used because you genuinely want unique values, not as a band-aid.

DISTINCT vs. GROUP BY vs. EXISTS
SELECT DISTINCT col and SELECT col ... GROUP BY col produce the same unique values, and modern optimizers often generate nearly identical plans. Use GROUP BY when you also need an aggregate (COUNT, SUM); use DISTINCT when you only want unique rows.
To answer "does a matching row exist," prefer WHERE EXISTS (...) or a semi-join over SELECT DISTINCT ... in a subquery — EXISTS can stop at the first match and skips the de-dup pass entirely.
COUNT(DISTINCT col) counts unique values but carries the same sort/hash cost as DISTINCT.
Summary — what a strong answer emphasizes

Define it crisply: DISTINCT returns only unique rows, and it de-duplicates on all selected columns together, not one. Explain the cost honestly: it forces a sort or hash to find duplicates, adding CPU/memory and possibly disk spills — cheap on small sets, costly on large ones, and helped a lot by a matching index. End with judgment: use it when you truly want distinct values, and never as a crutch to hide duplicate-producing joins (fix the join instead). Common mistakes: thinking DISTINCT targets a single column; believing it's "free"; and using it to mask a fan-out join.

**Comparison:**

```
Aspect	Behavior / cost
What it removes	Duplicate rows across ALL selected columns together
Scope	Whole selected row, not a single column
When it runs	After FROM/WHERE and projection, as a final de-dup pass
How it's executed	Sort (O(n log n)) or hash aggregate (O(n), memory ∝ #distinct)
Failure mode at scale	Temp-file / disk spill, high memory, delayed first row
Speed-up	Index on the distinct column(s) enables a cheap skip-scan
Use GROUP BY instead when	You also need an aggregate (COUNT/SUM/…)
Use EXISTS instead when	You only need to test existence — avoids the dedup pass
Anti-pattern	Masking duplicate rows from a bad join — fix the join instead
```

**Code:**

```sql
-- Unique values of one column
SELECT DISTINCT country FROM Users;

-- Whole-row: unique (city, country) PAIRS, not distinct-by-city
SELECT DISTINCT city, country FROM Users;

-- Equivalent unique values via GROUP BY (use when you also need aggregates)
SELECT country FROM Users GROUP BY country;

-- Count of unique values (same sort/hash cost as DISTINCT)
SELECT COUNT(DISTINCT country) FROM Users;

-- Prefer EXISTS over SELECT DISTINCT in a subquery for existence checks
SELECT o.*
FROM Orders o
WHERE EXISTS (SELECT 1 FROM Items i WHERE i.order_id = o.id);
```

**Example input / output:**

```

Users table:

CODE
Copy
id | city        | country
---+-------------+--------
1  | Springfield | USA
2  | Springfield | UK
3  | London      | UK
4  | London      | UK

SELECT DISTINCT country FROM Users;

CODE
Copy
country
-------
USA
UK

SELECT DISTINCT city, country FROM Users; (whole-row dedup)

CODE
Copy
city        | country
------------+--------
Springfield | USA
Springfield | UK
London      | UK

Note the two "Springfield" rows both survive because the (city, country) pairs differ; the two identical "London, UK" rows collapse to one.
```

