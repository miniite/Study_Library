---
id: "TOOLS/MySQL/MySQL_Platform/MySQL_Query_Basics"
level: 4
type: "concept"
topic: "TOOLS/MySQL/MySQL_Platform"
tools: ["MySQL"]
---

# MySQL Query Basics

**L4 Advanced** · [↑ MySQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Everyday MySQL is `SELECT … FROM … WHERE`, `INSERT`, `UPDATE`, `DELETE` and `ALTER TABLE`, plus three-valued logic around `NULL`, pattern matching with `LIKE`, and small expression helpers (`CASE`, `COALESCE`, `CONCAT`) — and the habits that keep these safe are explicit column lists, a `WHERE` on every write, and treating `NULL` as "unknown", not as a value.

### 1. The big picture
Everything else in SQL builds on a handful of statements. If these are solid, joins, windows and tuning become extensions rather than new topics. This lesson covers the core vocabulary in MySQL's dialect: retrieving and filtering rows, handling `NULL`, creating and changing data, shaping output with expressions, and the client-side `DELIMITER` command you meet when writing stored programs.

The same ideas appear in the T-SQL fundamentals books (query logic, `NULL` handling, `CASE`, `CONCAT`) with small dialect differences: MySQL uses backticks for identifiers, `LIMIT` instead of `TOP`, `IFNULL`/`COALESCE`, and `DELIMITER` is a *client* command, not SQL.

### 2. Core concepts and vocabulary
- **Projection** — choosing columns in `SELECT`. **Selection** — choosing rows with `WHERE`.
- **Predicate** — a condition that evaluates to TRUE, FALSE or UNKNOWN.
- **Three-valued logic** — any comparison with `NULL` is UNKNOWN; `WHERE` keeps only TRUE.
- **Alias** — a temporary name: `price * qty AS line_total`, `orders AS o`.
- **Wildcards** — `%` (any length, including zero), `_` (exactly one character) in `LIKE`.
- **`DISTINCT`** — remove duplicate result rows (whole row, not one column).
- **`CASE`** — conditional expression returning one value per row.
- **`DEFAULT`** — value used when an `INSERT` omits the column.
- **DML / DDL** — data manipulation (`INSERT/UPDATE/DELETE`) vs definition (`CREATE/ALTER/DROP`).
- **`DELIMITER`** — mysql-client setting for the statement terminator.

### 3. How it works, step by step
**Logical order of evaluation of a `SELECT`** (not the written order): `FROM`/joins → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` (expressions, aliases) → `DISTINCT` → `ORDER BY` → `LIMIT`. That is why `WHERE` can't use a select alias but `ORDER BY` can.

**Writing safely**
1. Name the columns in `INSERT INTO t (a,b) VALUES …` — it survives column reordering/additions.
2. Test the `WHERE` with a `SELECT` before turning it into `UPDATE`/`DELETE`.
3. Wrap risky changes in `START TRANSACTION` and check `ROW_COUNT()` before `COMMIT`.
4. Keep `sql_safe_updates` on in interactive sessions: it refuses `UPDATE`/`DELETE` without a key-based `WHERE` or `LIMIT`.

### 4. Worked example
```sql
-- Select and filter
SELECT customer_id, first_name, last_name
FROM   customer
WHERE  country = 'IN' AND (status = 'active' OR vip = 1) AND NOT blocked;

-- IN is shorthand for OR-ed equalities
SELECT * FROM orders WHERE status IN ('PAID','SHIPPED','DONE');

-- LIKE and wildcards
SELECT * FROM product WHERE name LIKE 'Pro%';        -- starts with Pro
SELECT * FROM product WHERE code LIKE 'A_7';         -- A, any one char, 7
SELECT * FROM product WHERE name NOT LIKE '%refurb%';

-- NULL handling
SELECT * FROM customer WHERE phone IS NULL;          -- not "= NULL"
SELECT * FROM customer WHERE phone IS NOT NULL;
SELECT IFNULL(phone, 'n/a'), COALESCE(mobile, phone, 'n/a') FROM customer;

-- Aliases, CONCAT vs CONCAT_WS, CASE
SELECT CONCAT(first_name, ' ', last_name)              AS full_name,   -- NULL if any part is NULL
       CONCAT_WS(' ', first_name, middle_name, last_name) AS safe_name, -- skips NULL parts
       CASE WHEN amount >= 1000 THEN 'High' ELSE 'Low' END AS tier
FROM   orders o JOIN customer c ON c.customer_id = o.customer_id;

-- Normalize a display name
SELECT CONCAT(UPPER(LEFT(TRIM(name),1)), LOWER(SUBSTRING(TRIM(name),2))) AS display_name
FROM   users;

-- Division by zero guard
SELECT revenue / NULLIF(units, 0) AS price_per_unit FROM sales;

-- Insert (single, multi-row), update, delete
INSERT INTO customer (first_name, last_name, country) VALUES ('Asha','Nair','IN');
INSERT INTO tag (name) VALUES ('sql'),('mysql'),('index');
UPDATE customer SET status = 'inactive' WHERE last_login < NOW() - INTERVAL 1 YEAR;
DELETE FROM session WHERE expires_at < NOW();

-- Schema change
ALTER TABLE customer ADD COLUMN loyalty_points INT UNSIGNED NOT NULL DEFAULT 0;
ALTER TABLE customer RENAME COLUMN phone TO phone_number;   -- 8.0; CHANGE in 5.7
RENAME TABLE customer TO client;

-- Comments in scripts
-- single line     # also single line     /* block */
```
Notes on the results: `CONCAT('a', NULL, 'b')` is `NULL`; `CONCAT_WS('-', 'a', NULL, 'b')` is `a-b`. `AVG`, `SUM`, `MIN`, `MAX` ignore `NULL`s (`AVG` of `1,NULL,3` is 2, not 4/3); `COUNT(col)` counts non-NULL values while `COUNT(*)` counts rows; an aggregate over *no* rows returns `NULL` (except `COUNT`, which returns 0).

**`DELIMITER`** (client command, not SQL sent to the server):
```sql
DELIMITER //
CREATE PROCEDURE bump_points(IN cid INT)
BEGIN
  UPDATE customer SET loyalty_points = loyalty_points + 1 WHERE customer_id = cid;
  SELECT loyalty_points FROM customer WHERE customer_id = cid;
END//
DELIMITER ;
```
The procedure body contains `;`, which the client would otherwise treat as "end of statement" and send half a definition. Changing the delimiter lets the whole body go to the server as one statement. Drivers and GUI tools that send the statement as a single string don't need it.

### 5. Under the hood
- **`NULL` and indexes.** `IS NULL` can use an index (NULLs are indexed in InnoDB B-trees); `col = NULL` is always UNKNOWN and returns no rows.
- **`IN (list)`** with constants is sorted and binary-searched; a long list is still one range-ish probe set per element. `IN` with a `NULL` in `NOT IN (…, NULL)` yields UNKNOWN for every row — return nothing.
- **`LIKE` and collation.** Case sensitivity follows the column collation; `LIKE 'abc%'` can use an index range, `LIKE '%abc'` cannot. `=` also respects trailing-space rules of the collation (PAD SPACE), `LIKE` does not.
- **`DISTINCT`** requires deduplication: MySQL builds a temporary table or sorts, compares the *entire* select list, and can be as expensive as a `GROUP BY` over a large result. It is often masking a join that duplicates rows — fix the join instead. An index that covers the selected columns lets it stream.
- **`UPDATE` without `WHERE`** changes every row, locks them, and logs them; in a transaction you can `ROLLBACK`, in autocommit you can't. **`DELETE`** is transactional and row-by-row; `TRUNCATE` is DDL (implicit commit, resets `AUTO_INCREMENT`, can't be rolled back); `DROP` removes the table.
- **Omitted columns on `INSERT`.** The column gets its `DEFAULT`; if none exists and the column is `NOT NULL`, strict mode (default) raises error 1364 "Field doesn't have a default value"; with strict mode off it inserts an implicit zero/empty value silently.
- **`ALTER TABLE`** may be `INSTANT` (metadata only, e.g., adding a trailing column in 8.0), `INPLACE` (rebuild without blocking DML), or `COPY` (full copy, blocks writes) depending on the change; check with `ALGORITHM=INSTANT/INPLACE` — the statement errors if it can't comply, which is how you avoid surprises.
- **Reserved words/identifiers.** Quote with backticks; `ANSI_QUOTES` mode changes `"` to identifier quoting.

### 6. Performance and design implications
- Select only the columns you need — fewer bytes, enables covering indexes.
- Put filters on raw columns (no function wrapping) so indexes can be used.
- `IN` with a few values is fine; thousands of values → temp table/join.
- Prefer multi-row `INSERT` (or `LOAD DATA`) over many single-row statements.
- Use `COALESCE` for portability; `IFNULL` is MySQL-specific two-argument sugar.
- Don't use `DISTINCT` to hide a bad join; don't use `SELECT *` in application code.
- `UPDATE … WHERE` on an unindexed column locks a lot — index or batch by PK.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SELECT … WHERE … ORDER BY … LIMIT` | Query | Everything |
| `IS [NOT] NULL`, `<=>` (null-safe equal) | Null tests | Comparing possibly-null columns |
| `IFNULL`, `COALESCE`, `NULLIF` | Null substitution / creation | Defaults, division guards |
| `CONCAT`, `CONCAT_WS`, `TRIM`, `UPPER`, `LOWER`, `SUBSTRING`, `CHAR_LENGTH` | String helpers | Normalizing text |
| `CASE WHEN … THEN … ELSE … END` | Conditional value | Labels, pivots |
| `INSERT … VALUES (…),(…)`, `INSERT … SELECT` | Add rows | Seeding, copying |
| `UPDATE … SET … WHERE`, `DELETE … WHERE … LIMIT n` | Modify/remove rows | Maintenance |
| `ALTER TABLE … ADD/DROP/MODIFY/RENAME` | Schema change | Evolving schema |
| `DELIMITER //` | Change client terminator | Stored programs in the CLI |
| `SET sql_safe_updates=1` | Block keyless mass updates | Interactive sessions |
| `SELECT VERSION();` | Server version | Feature checks |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `COALESCE` — portable, multiple fallbacks | Need exactly MySQL semantics of `IFNULL` (same result in most cases) |
| `CONCAT_WS` — optional parts | You *want* the whole result NULL if any part is missing |
| `IN (…)` — short constant lists | Very long lists or lists from another table (join instead) |
| `CASE` in `SELECT` — presentation logic | Heavy business rules duplicated across queries (put in a view/column) |
| `DELETE` with `LIMIT`/chunks | Table-wide wipe (`TRUNCATE`) when you really want all rows gone |
| `INSERT IGNORE` | You need to know about errors (it hides them) |

### 9. Common mistakes
- **`= NULL` / `<> NULL`** → always UNKNOWN → `IS NULL` / `IS NOT NULL`.
- **`UPDATE`/`DELETE` without `WHERE`** → entire table changed → select first, use transactions and `sql_safe_updates`.
- **`NOT IN` with a nullable subquery column** → empty result → `NOT EXISTS`.
- **`OR` precedence** (`a = 1 AND b = 2 OR c = 3`) → unintended grouping → parenthesize.
- **`INSERT` without column list** → breaks on schema change → list columns.
- **Concatenating with `+`** → numeric addition in MySQL → `CONCAT`.
- **`COUNT(col)` vs `COUNT(*)`** confusion → wrong totals with NULLs → choose deliberately.
- **Alias in `WHERE`** → unknown column → repeat the expression or use a derived table.
- **Forgetting that `LIKE` is case-insensitive under `_ci` collations** → surprising matches → use a binary/case-sensitive collation when needed.

### 10. 🎯 Interview kit
**30-second answer:** `SELECT` chooses columns, `WHERE` filters rows with TRUE/FALSE/UNKNOWN logic, so test `NULL` with `IS NULL`. `INSERT` lists columns and omitted ones take their `DEFAULT`; `UPDATE` and `DELETE` need a `WHERE` or they hit every row. `ALTER TABLE` changes the schema; `CASE`, `COALESCE`, `CONCAT_WS` shape output.

**Structure a longer answer:** statement purpose → syntax → `NULL`/edge semantics → safety (transactions, `WHERE`, safe updates) → performance note (index use, batching).

**Likely follow-ups:**
- *`IN` vs multiple `OR`s?* → same semantics; `IN` is shorter and the optimizer treats constants as a range list.
- *`IFNULL` vs `COALESCE`?* → two args vs many; COALESCE is standard.
- *`CONCAT` vs `CONCAT_WS`?* → NULL poisons CONCAT; `CONCAT_WS` skips NULLs and adds separators.
- *`DELETE` vs `TRUNCATE` vs `DROP`?* → rows (transactional, logged) vs all rows (DDL, resets counter) vs the table itself.
- *What happens if you omit a column with no default?* → error 1364 in strict mode for `NOT NULL` columns.
- *Why `DELIMITER`?* → client parsing of `;` inside stored-program bodies.

**Signals interviewers look for:** correct `NULL` reasoning, safety habits, clean syntax, awareness of what `DISTINCT` and `SELECT *` cost.

### 11. Check yourself
- [ ] Write a query using `AND`, `OR`, `NOT`, `IN` and `LIKE` with correct parentheses.
- [ ] Explain why `WHERE col = NULL` returns nothing and give two correct alternatives.
- [ ] Show `CONCAT` vs `CONCAT_WS` with a `NULL` argument.
- [ ] Add a `NOT NULL DEFAULT` column and say what algorithm MySQL uses.
- [ ] Explain the logical order of `SELECT` clause evaluation.
- [ ] Describe what happens on `INSERT` when you omit a `NOT NULL` column without default.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../_index.md>)

## Practice questions

- [Basic] [What does the INSERT INTO statement do and what happens if you omit a column that has no default?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33338_What_does_the_INSERT_INTO_statement_do_and_what_happens_if_y.md>)
- [Basic] [What do the % and _ wildcards mean in a LIKE pattern and give an example of each?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33348_What_do_the_and_wildcards_mean_in_a_LIKE_pattern_and_give_an.md>)
- [Basic] [What does the ALTER TABLE statement let you do and give two examples of its use?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33361_What_does_the_ALTER_TABLE_statement_let_you_do_and_give_two_.md>)
- [Basic] [What is the result of an arithmetic operation involving a NULL value in MySQL?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33378_What_is_the_result_of_an_arithmetic_operation_involving_a_NU.md>)
- [Basic] [What is the difference between the AVG, SUM, MIN, and MAX aggregate functions when NULLs are present?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33017_What_is_the_difference_between_the_AVG_SUM_MIN_and_MAX_aggre.md>)
- [Intermediate] [What is the difference between a stored procedure being called with CALL versus a function used in a SELECT?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33415_What_is_the_difference_between_a_stored_procedure_being_call.md>)
- [Intermediate] [How does GROUP_CONCAT work and how would you produce a comma-separated list of a customer's product names?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33018_How_does_GROUP_CONCAT_work_and_how_would_you_produce_a_comma.md>)
- [Intermediate] [How do NULL values behave in comparisons and aggregate functions in MySQL?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33091_How_do_NULL_values_behave_in_comparisons_and_aggregate_funct.md>)

## Related concepts

- [WHERE vs HAVING](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/WHERE_vs_HAVING.md>) *(DBMS Concepts)*
- [Scalar Subqueries](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Scalar_Subqueries.md>) *(SQL Fundamentals)*
- [Joins vs Subqueries and Join Types](<../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Joins_vs_Subqueries_and_Join_Types.md>) *(Joins and Subqueries)*
- [Dynamic SQL](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Dynamic_SQL.md>) *(DBMS Concepts)*
- [MERGE and Upserts](<../../SQL_Server/SQL_Server_Platform/MERGE_and_Upserts.md>) *(SQL Server Platform)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Column Aliases
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Combining Aliases and Computed Columns
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › The IN Operator
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Single-table queries (cont.)
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Data ��difi�ati�n
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — 12. Understanding T-SQL Programming Logic › Using Variables in WHERE and HAVING Clauses

