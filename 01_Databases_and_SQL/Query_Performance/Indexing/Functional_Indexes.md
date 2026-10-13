---
id: "01_Databases_and_SQL/Query_Performance/Indexing/Functional_Indexes"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Query_Performance/Indexing"
tools: []
---

# Functional Indexes

**L4 Advanced** · [↑ Indexing](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A functional (expression) index stores the *result* of an expression such as `lower(email)`, so queries that filter on exactly that expression can seek instead of scanning.

### 1. The big picture
A normal B-tree index is sorted by the raw column value. The moment a query wraps the column in a function — `WHERE lower(email) = ?`, `WHERE DATE(created_at) = ?`, `WHERE YEAR(OrderDate) = 2025` — the engine can't use the column's sort order to navigate, because the function output isn't sorted the same way. The predicate is **non-sargable** and the plan degrades to a scan.

There are three ways out:
1. **Rewrite the predicate** so the column is bare (`created_at >= '2025-01-01' AND created_at < '2025-01-02'`).
2. **Index the expression** — a functional/expression index (PostgreSQL, Oracle, MySQL 8, SQLite).
3. **Materialize the expression as a column** and index that — computed/generated column (SQL Server persisted computed columns, MySQL generated columns, PostgreSQL stored generated columns) or a type that already behaves the way you want (`citext`, case-insensitive collation).

### 2. Core concepts and vocabulary
- **Sargable** — a predicate form that an index can be used for: `column <op> constant`.
- **Expression / functional index** — an index whose key is computed from one or more columns by an expression.
- **Immutable function** — always returns the same output for the same input; required to index an expression because stored index entries must never change meaning.
- **Computed (generated) column** — a column whose value is derived from others; may be *virtual* (computed on read) or *stored/persisted*.
- **Collation** — rules for comparing/sorting strings (case, accent). An index is only valid for the collation it was built with.
- **Operator class (opclass)** — in PostgreSQL, defines which comparison operators an index supports for a type (e.g. `text_pattern_ops` for `LIKE 'abc%'` under non-C collations).
- **Trigram index** — GIN/GiST index over 3-character substrings (`pg_trgm`) that makes `LIKE '%abc%'` searchable.

### 3. How it works, step by step
1. You declare `CREATE INDEX ix_users_email_lower ON users (lower(email));`.
2. On every insert/update of `email`, the engine evaluates `lower(email)` and stores that value (plus row locator) in the B-tree.
3. At plan time, the optimizer matches the query's expression against index expressions **textually/structurally**. `WHERE lower(email) = lower($1)` matches; `WHERE email ILIKE $1` does not; `WHERE lower(trim(email)) = …` does not.
4. If matched, it seeks on the stored value exactly like any B-tree.

Consequences of step 3: the expression in the query has to be *the same expression* (same function, same argument, same collation/type) as in the index definition. That's why an ORM generating `lower(email) = lower(?)` works but `LOWER(email COLLATE "tr_TR")` may not.

### 4. Worked example
```sql
-- PostgreSQL: case-insensitive unique email, fast lookups
CREATE UNIQUE INDEX ux_users_email_lower ON users (lower(email));

-- fast: expression matches the index
EXPLAIN SELECT * FROM users WHERE lower(email) = lower('Alice@Example.com');
--  Index Scan using ux_users_email_lower ...

-- slow: different expression -> seq scan
EXPLAIN SELECT * FROM users WHERE email ILIKE 'alice@example.com';

-- prefix autocomplete, case-insensitive
CREATE INDEX ix_users_name_prefix ON users (lower(username) text_pattern_ops);
SELECT username FROM users WHERE lower(username) LIKE lower('mar') || '%';

-- SQL Server equivalent: persisted computed column + index
ALTER TABLE dbo.Orders ADD OrderYear AS YEAR(OrderDate) PERSISTED;
CREATE INDEX IX_Orders_OrderYear ON dbo.Orders (OrderYear);
```
What to look at in the plan: *Index Scan/Seek using* the expression index vs *Seq Scan*; `Filter:` vs `Index Cond:` lines in PostgreSQL. In MySQL 8, `EXPLAIN` shows `key: functional_index_name`. In SQL Server the optimizer matches a query expression to a computed column definition and uses the index even if you don't reference the column name.

### 5. Under the hood
- **Immutability requirement.** If the function result could change (depends on time zone, locale setting, or current date), stored index entries would be wrong. PostgreSQL refuses non-`IMMUTABLE` functions (`now()`, `to_char(timestamptz,…)` with session TZ). Wrap the logic in an immutable function only if it is truly deterministic. `date_trunc('day', ts AT TIME ZONE 'UTC')` is fine; `date_trunc('day', ts)` on `timestamptz` depends on session time zone and is rejected.
- **Collation and opclass mismatches silently defeat the index.** A B-tree built under `en_US.utf8` can't serve `LIKE 'abc%'` — the sort order isn't byte order — so you need `text_pattern_ops` (or a `C` collation). Conversely, a query that says `COLLATE "C"` won't use an index built in the default collation. Diagnose via `\d+ table` / `pg_indexes.indexdef`, and compare against the query's `EXPLAIN` + `pg_collation`.
- **Planning statistics.** PostgreSQL gathers statistics on the index expression itself at `ANALYZE`, so estimates are good only after an analyze following index creation. SQL Server creates statistics on the computed column.
- **Uniqueness.** A unique functional index enforces uniqueness on the *expression*: `UNIQUE (lower(email))` rejects `A@x.com` if `a@x.com` exists. This is the standard solution for case-insensitive unique constraints.
- **`citext` and case-insensitive collations** move case folding into the type/collation: no query rewrite, but a different type everywhere, extension dependency, and sometimes slower comparisons. A **generated column** keeps the logic explicit and visible to every tool.
- **Substring and fuzzy matching** (`LIKE '%abc%'`, typo tolerance) can't be handled by any B-tree, even functional; use trigram (GIN/GiST), full-text, or a search service.

### 6. Performance and design implications
- Index maintenance now includes evaluating the function on every row write — negligible for `lower()`, significant for heavy expressions.
- Size: the index stores the computed value; at 100 M emails, a `lower(email)` index is roughly the same size as the plain one. A generated column adds a table column *and* an index.
- Prefer rewriting the predicate when possible — it needs no new index and works on all engines (`created_at >= :d AND created_at < :d + 1`).
- Verify after deploy: `pg_stat_user_indexes.idx_scan` for the new index; ensure the ORM's emitted SQL matches the expression byte for byte (mind `lower()` vs `upper()`, `::text` casts, `COALESCE`).
- Adding a functional index on a 100 M-row table: `CREATE INDEX CONCURRENTLY`, off-peak, check disk and WAL growth.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE INDEX ix ON t (lower(col))` | Expression index (PG, Oracle, MySQL 8 with extra parentheses `((lower(col)))`) | Case-insensitive lookups |
| `CREATE UNIQUE INDEX … (lower(col))` | Case-insensitive uniqueness | Emails, usernames |
| `text_pattern_ops` / `varchar_pattern_ops` | B-tree opclass for `LIKE 'x%'` | Prefix search under non-C collation |
| `ALTER TABLE … ADD col … GENERATED ALWAYS AS (expr) STORED` | Materialized derived column | Expression reused in many places |
| `ADD col AS (expr) PERSISTED` (SQL Server) | Persisted computed column | Index on an expression |
| `CREATE EXTENSION citext` / `pg_trgm` | Case-insensitive text type / trigram search | Simplest CI equality / substring search |
| `ANALYZE t` | Refresh stats (incl. expression stats) | After creating an expression index |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Query can't be rewritten (ORM, legacy) and expression is fixed | The predicate can simply be rewritten to a range |
| Need case-insensitive *uniqueness* | You want substring or fuzzy search → trigram/FTS |
| One expression, one table | Many queries use various transformations → generated column(s) or normalized column |
| Function is cheap and immutable | Function is slow, non-deterministic, or depends on session settings |
| Generated column when other tools (BI, ETL) need to see the value | `citext`: extension-free managed environments, strict portability |

### 9. Common mistakes
- **Index on `lower(email)` but query on `email = ?`** → seq scan → query the same expression or normalise on write.
- **Writing `WHERE lower(email) = ?` and passing a mixed-case value** → no matches → `lower($1)` on the parameter too.
- **Forgetting `text_pattern_ops` for prefix `LIKE`** → index unused under non-C collation → add pattern ops or use a `C` collated column.
- **Expecting `YEAR(col) = 2025` to use an index on `col`** → full scan → rewrite as half-open range.
- **Wrapping the indexed column in an implicit cast** (`WHERE varchar_col = 5`) → non-sargable conversion → match types.
- **Using `ILIKE` for equality** → no B-tree support → use `lower()` expression or `citext`.
- **Not analysing after creation** → bad estimates → run `ANALYZE`.

### 10. 🎯 Interview kit
**30-second answer:** Wrapping a column in a function makes the predicate non-sargable, so the index on the raw column can't be used. You either rewrite the predicate to leave the column bare, or create a functional index on the expression (`lower(email)`) so the stored index values match what the query computes.

**Structure a longer answer:** why B-tree order breaks under a function → options (rewrite, expression index, generated column, `citext`/collation) → matching rule (expression must match) → immutability and collation/opclass gotchas → unique case-insensitive constraint → verification with EXPLAIN.

**Likely follow-ups:**
- *Functional index vs generated column vs `citext` at 100 M rows?* → Functional index: no schema change, smallest footprint, but the query must repeat the expression. Generated column: explicit, reusable, extra storage. `citext`: transparent to queries but a type migration and extension. Ship the functional unique index unless many consumers need the value.
- *Why must the query expression match exactly?* → Planner matches expressions structurally; any difference is a different function.
- *Case-insensitive prefix search at 200 M rows?* → `lower(name) text_pattern_ops` B-tree for prefix; trigram for infix/fuzzy; search service when ranking, typo tolerance or faceting is required.
- *Index exists but isn't used — diagnose?* → Compare expression text, collation, opclass, parameter types; `EXPLAIN`; stats freshness; selectivity.

**Signals interviewers look for:** the sargability concept, knowledge of immutability/collation traps, a preference for simple rewrites first.

### 11. Check yourself
- [ ] Rewrite `WHERE DATE(created_at) = '2025-03-01'` to be sargable.
- [ ] Create a case-insensitive unique constraint on email in PostgreSQL and in SQL Server.
- [ ] Explain why `now()` can't appear in an index expression.
- [ ] Name two reasons an expression index exists but isn't chosen.
- [ ] Choose between trigram index and `text_pattern_ops` for `LIKE 'ab%'` and `LIKE '%ab%'`.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [You need email lookups to be case-insensitive and fast. How would you approach indexing for WHERE lower(email) = ?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20035_You_need_email_lookups_to_be_case-insensitive_and_fast_How_w.md>)
- [Deep] [You need a functional index on lower(email) to dedup case-insensitive emails during a load. Why must the query expression match the index exactly?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20342_You_need_a_functional_index_on_lower_email_to_dedup_case-ins.md>)
- [Deep] [Your ORM emits WHERE lower(email) = lower(?). What index makes this fast?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19647_Your_ORM_emits_WHERE_lower_email_lower_What_index_makes_this.md>)
- [Deep] [You need to make WHERE lower(username) LIKE ? || '%' fast for a prefix autocomplete. What index type and operator class would you choose and why?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20095_You_need_to_make_WHERE_lower_username_LIKE_fast_for_a_prefix.md>)
- [Deep] [You want to support case-insensitive uniqueness on username at scale. Compare a functional unique index on lower(username) versus a citext column for your team.](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20121_You_want_to_support_case-insensitive_uniqueness_on_username_.md>)
- [Deep] [You want to add a case-insensitive unique constraint on email at 100M rows. Compare a functional index on lower(email), a generated column, and CITEXT. Which do you ship and why?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19713_You_want_to_add_a_case-insensitive_unique_constraint_on_emai.md>)
- [Deep] [You need case-insensitive prefix search on names at 200M rows. Compare a functional index on lower(name) with text_pattern_ops, a trigram index, and a dedicated search service.](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19793_You_need_case-insensitive_prefix_search_on_names_at_200M_row.md>)
- [Deep] [A query with WHERE lower(email) = ? scans instead of using your lower(email) index because collation differs. How do you diagnose collation/opclass mismatches that silently defeat expression indexes?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19860_A_query_with_WHERE_lower_email_scans_instead_of_using_your_l.md>)

## Related concepts

- [Data Validation Queries](<../../SQL_Language/SQL_Fundamentals/Data_Validation_Queries.md>) *(SQL Fundamentals)*
- [Leftmost Prefix Rule](<../../SQL_Language/SQL_Fundamentals/Leftmost_Prefix_Rule.md>) *(SQL Fundamentals)*
- [Index Access Paths](<../../Database_Internals/DBMS_Concepts/Index_Access_Paths.md>) *(DBMS Concepts)*

## References

- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Chapter 6 › Chapter 9
- *Fuzzy Data Matching with SQL* — Jim Lehmer — Function Junction › LOWER and UPPER
- *Fuzzy Data Matching with SQL* — Jim Lehmer — Names, Names, Names › Punctuation
- *Fuzzy Data Matching with SQL* — Jim Lehmer — Names, Names, Names › Suffixes
- *Fuzzy Data Matching with SQL* — Jim Lehmer — Orthogonal Data › A Common Problem, A Common Solution, A New Common Problem
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Single-table queries (cont.)

