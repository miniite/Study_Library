---
id: "TOOLS/PostgreSQL/PostgreSQL_Platform/PostgreSQL_Query_Basics"
level: 4
type: "concept"
topic: "TOOLS/PostgreSQL/PostgreSQL_Platform"
tools: []
---

# PostgreSQL Query Basics

**L4 Advanced** · [↑ PostgreSQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A PostgreSQL query is a declarative description of the rows you want — `SELECT` expressions, from `FROM` sources, filtered by `WHERE`, shaped by `GROUP BY/HAVING`, ordered and limited — and the planner, not you, decides how to fetch them.

### 1. The big picture
SQL is declarative: you state *what* result you want, and PostgreSQL's parser → rewriter → planner → executor pipeline chooses *how*. Learning the clauses and their logical evaluation order, plus how `NULL` and types behave, is the foundation for everything else (joins, windows, tuning). PostgreSQL adds friendly extensions on top of standard SQL: `ILIKE`, `LIMIT`, `RETURNING`, `::` casts, `DISTINCT ON`, rich string/array/JSON operators.

### 2. Core concepts and vocabulary
- **Relation** — table, view, or subquery result with rows and typed columns.
- **Expression** — column references, literals, operators, function calls, `CASE`.
- **Alias** — `AS name` for a column or table.
- **Predicate** — boolean expression in `WHERE`/`ON`/`HAVING`.
- **`NULL`** — "unknown"; propagates through arithmetic and comparisons.
- **Cast** — `CAST(x AS int)` or `x::int`.
- **Schema** — namespace of objects; resolved via `search_path`.
- **Sargable** — a predicate form that can use an index (`col = $1`, `col >= $1`), not `lower(col) = $1`.
- **psql meta-commands** — `\dt`, `\d table`, `\l`, `\dn`, `\timing`, `\x`.

### 3. How it works, step by step
1. **Parse** — syntax tree; **analyze** — resolve names/types.
2. **Rewrite** — expand views and rules.
3. **Plan** — choose scans, join order, methods using statistics.
4. **Execute** — pull rows through the plan nodes.

Logical clause order: `FROM` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` (and window functions) → `DISTINCT` → `ORDER BY` → `LIMIT/OFFSET`. That's why a column alias works in `ORDER BY` but not in `WHERE`.

### 4. Worked example
```sql
-- explore
\dt                       -- tables in search_path
\d customers              -- columns, indexes, constraints of one table
\l                        -- databases on the server

-- core query
SELECT c.customer_id,
       upper(c.name)                       AS name,
       c.created_at::date                  AS signup_day,
       CASE WHEN c.is_premium THEN 'premium' ELSE 'standard' END AS tier,
       COALESCE(c.phone, 'n/a')            AS phone
FROM   customers AS c
WHERE  c.country IN ('IN', 'US')
  AND  c.name ILIKE 'a%'                   -- case-insensitive (PG extension)
  AND  c.deleted_at IS NULL
ORDER  BY c.created_at DESC NULLS LAST, c.customer_id
LIMIT  10;

-- string and number helpers
SELECT 'a' || 'b' AS concat_op,            -- NULL if any operand is NULL
       concat('a', NULL, 'b') AS concat_fn,-- ignores NULL -> 'ab'
       trim('  x  ') AS trimmed,
       round(4.5) AS r,  ceil(4.5) AS c, floor(4.5) AS f,   -- 5, 5, 4 (numeric rounds half up)
       10 % 3 AS modulo,                   -- 1
       '42'::int + 1 AS casted;

-- change data safely
BEGIN;
UPDATE customers SET is_premium = true WHERE customer_id = 7 RETURNING *;
ROLLBACK;   -- or COMMIT
```
Notes: `round(4.5)` on `numeric` gives 5 (half away from zero) while on `double precision` follows the C library's round-half-to-even; `\dt` shows only tables visible on the `search_path`; `\l` lists all databases on the server, not just yours.

### 5. Under the hood   *(optional at this level)*
- Types drive behavior: integer division truncates (`7/2 = 3`); cast one side to `numeric` for fractions. `text` and `varchar(n)` perform identically; `char(n)` pads.
- Comparison with `NULL` yields `NULL`; `WHERE` keeps only `TRUE`, so `x = NULL` matches nothing; use `IS NULL`, `IS DISTINCT FROM`. `COUNT(*)` counts rows, `COUNT(col)` counts non-null values. Aggregates ignore `NULL` (except `COUNT(*)`).
- `ORDER BY` default puts `NULL` last for `ASC`, first for `DESC`; override with `NULLS FIRST/LAST`.
- `LIMIT` without `ORDER BY` returns an arbitrary subset; physical order is not guaranteed, especially after updates or with parallel scans.
- Parameterized (prepared) statements reuse parse work and prevent SQL injection; after several executions the planner may choose a generic plan (`plan_cache_mode`).

### 6. Performance and design implications
- Select only the columns you need; `SELECT *` defeats index-only scans and widens network payloads.
- Keep predicates sargable: write `created_at >= '2026-01-01' AND created_at < '2027-01-01'` instead of `EXTRACT(year FROM created_at) = 2026`.
- `LIKE 'abc%'` can use a B-tree (with `text_pattern_ops` or a C collation); `LIKE '%abc'` cannot — consider `pg_trgm`.
- Let the database aggregate: ship results, not rows, to the application.
- `EXPLAIN` the query whenever table sizes grow by an order of magnitude.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `\d`, `\dt`, `\l`, `\dn`, `\df` | inspect objects | exploration |
| `\timing`, `\x` | time queries, expanded output | psql sessions |
| `EXPLAIN [ANALYZE]` | show plan | tuning |
| `SET search_path` | schema resolution | multi-schema apps |
| `LIMIT/OFFSET`, `FETCH FIRST` | row cap | paging |
| `-- comment`, `/* block */` | comments | scripts, migrations |
| `ALTER TABLE … RENAME COLUMN/TO` | rename | schema changes |
| `BEGIN/COMMIT/ROLLBACK`, `SAVEPOINT` | transactions | atomic changes |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `ILIKE` for quick case-insensitive search | big tables → use `citext`/trigram/expression index |
| `COALESCE` for defaults | masking real missing data in analytics |
| `CASE` for conditional columns | very complex logic better in a view/function |
| Explicit casts | relying on implicit conversion (can disable indexes) |
| Qualified column names | single-table scripts (optional) |

### 9. Common mistakes
- `WHERE col = NULL` → no rows → `IS NULL`.
- Alias used in `WHERE` → error → repeat expression or wrap in subquery.
- Integer division surprises → truncated results → cast to `numeric`.
- Unquoted mixed-case identifiers → folded to lowercase → name them lowercase or quote consistently.
- `LIMIT` without `ORDER BY` → nondeterministic → always order.
- Forgetting `COMMIT` in a manual transaction → locks held → end transactions promptly.

### 10. 🎯 Interview kit
**30-second answer:** "A query is evaluated logically as FROM, WHERE, GROUP BY, HAVING, SELECT, ORDER BY, LIMIT; `NULL` is three-valued so you use `IS NULL`; the planner picks the physical plan, which I inspect with `EXPLAIN`."
**Structure a longer answer:** clauses and order → `NULL`/type pitfalls → PG-specific features → sargability and indexes.
**Likely follow-ups:** *`||` vs `CONCAT`?* → NULL handling. *`COUNT(*)` vs `COUNT(col)`?* → rows vs non-null. *`\dt` vs `\d t` vs `\l`?* → tables / one table's structure / databases. *`LIKE` vs `ILIKE`?* → case sensitivity.
**Signals interviewers look for:** correct `NULL` handling, ordered `LIMIT`, and that you think about indexes while writing predicates.

### 11. Check yourself
- [ ] State the logical order of clause evaluation and why aliases behave as they do.
- [ ] Explain three places `NULL` surprises you.
- [ ] Rewrite a non-sargable date filter so it can use an index.
- [ ] Use `RETURNING` to see changed rows and then roll back.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is the difference between the DISTINCT keyword and simply removing duplicates in application code?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36133_What_is_the_difference_between_the_DISTINCT_keyword_and_simp.md>)
- [Basic] [What does the ORDER BY clause do, and how do you sort by multiple columns with mixed ASC and DESC?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35827_What_does_the_ORDER_BY_clause_do_and_how_do_you_sort_by_mult.md>)
- [Intermediate] [What does a RIGHT JOIN do and how is it equivalent to a LEFT JOIN with the tables swapped?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36169_What_does_a_RIGHT_JOIN_do_and_how_is_it_equivalent_to_a_LEFT.md>)
- [Intermediate] [What is the difference between COUNT(DISTINCT column) and COUNT(column) and when would the results differ?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36198_What_is_the_difference_between_COUNT_DISTINCT_column_and_COU.md>)
- [Intermediate] [You need to unpivot a wide table with monthly columns into row-per-month form for reporting; how would you do this with a LATERAL join over a VALUES list versus a UNION ALL, and which scales better?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36259_You_need_to_unpivot_a_wide_table_with_monthly_columns_into_r.md>)
- [Intermediate] [What is the difference between a scalar subquery and a table subquery, with an example of each?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35842_What_is_the_difference_between_a_scalar_subquery_and_a_table.md>)
- [Intermediate] [How does DISTINCT ON work in Postgres, what ORDER BY does it require to be deterministic, and why do people get flaky results when they forget the tiebreaker column?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36057_How_does_DISTINCT_ON_work_in_Postgres_what_ORDER_BY_does_it_.md>)
- [Deep] [Your query needs the latest row per group and DISTINCT ON is fast but you're told window functions are more standard; compare DISTINCT ON versus ROW_NUMBER() = 1 in a subquery on both plan cost and index usage.](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36256_Your_query_needs_the_latest_row_per_group_and_DISTINCT_ON_is.md>)

## Related concepts

- [WHERE vs HAVING](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/WHERE_vs_HAVING.md>) *(DBMS Concepts)*
- [Scalar Subqueries](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Scalar_Subqueries.md>) *(SQL Fundamentals)*
- [GROUP BY vs PARTITION BY](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/GROUP_BY_vs_PARTITION_BY.md>) *(SQL Fundamentals)*
- [Joins vs Subqueries and Join Types](<../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Joins_vs_Subqueries_and_Join_Types.md>) *(Joins and Subqueries)*
- [CROSS APPLY and OUTER APPLY](<../../SQL_Server/T-SQL/CROSS_APPLY_and_OUTER_APPLY.md>) *(T-SQL)*
- [Join Types](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Join_Types.md>) *(SQL Fundamentals)*

