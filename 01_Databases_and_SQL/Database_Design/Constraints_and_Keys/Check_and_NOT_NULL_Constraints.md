---
id: "01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Check_and_NOT_NULL_Constraints"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Design/Constraints_and_Keys"
tools: []
---

# Check and NOT NULL Constraints

**L4 Advanced** · [↑ Constraints and Keys](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** NOT NULL forbids missing values and CHECK forbids values that break a rule; together they push data-quality rules into the database where no application can bypass them — and the optimizer benefits too.

### 1. The big picture
Data types restrict *kind* (an INT is a number) but not *meaning* (a negative age, an end date before the start date, a status of `'maybe'`). Constraints close that gap. NOT NULL answers "must this fact always exist?"; CHECK answers "which values are acceptable?". Because the database enforces them on every INSERT and UPDATE from every client — app, ETL, ad-hoc script — they are the most reliable layer of validation.

They are called **domain integrity** constraints (values valid within a column or row), in contrast to entity integrity (primary key) and referential integrity (foreign key).

### 2. Core concepts and vocabulary
- **NULL** — "unknown / not applicable"; not zero, not empty string. Comparisons with NULL yield UNKNOWN.
- **NOT NULL** — column-level rule: a value is mandatory.
- **CHECK constraint** — boolean expression that must not evaluate to FALSE for a row.
- **Three-valued logic** — TRUE / FALSE / UNKNOWN. A CHECK passes on UNKNOWN (NULL) — the surprise.
- **Column vs table CHECK** — one column, or several columns of the same row (`end_date >= start_date`).
- **DEFAULT** — value used when none is supplied; pairs with NOT NULL.
- **Trusted constraint** — one the engine knows is validated for all rows (SQL Server `is_not_trusted = 0`).
- **WITH NOCHECK / NOT VALID** — add a constraint without validating existing rows.
- **Domain / lookup table** — alternative to CHECK for changeable value lists (via FK).

### 3. How it works, step by step
1. You declare the rule in CREATE/ALTER TABLE.
2. On each INSERT/UPDATE the engine evaluates it for the changed row (no lookup in other tables).
3. NOT NULL: reject if the value is NULL.
4. CHECK: evaluate the expression; reject only if it is FALSE. If it is UNKNOWN (because of a NULL operand), accept.
5. A violation aborts the statement with an error (SQL Server 547, PostgreSQL 23514, 23502 for NOT NULL).
6. When adding to existing data, the engine scans the table to validate unless told not to.

### 4. Worked example
```sql
CREATE TABLE dbo.Orders (
    order_id    INT           NOT NULL CONSTRAINT PK_Orders PRIMARY KEY,
    customer_id INT           NOT NULL,
    status      VARCHAR(10)   NOT NULL CONSTRAINT DF_Orders_status DEFAULT 'NEW',
    qty         INT           NOT NULL,
    price       DECIMAL(10,2) NOT NULL,
    placed_at   DATETIME2     NOT NULL,
    shipped_at  DATETIME2     NULL,
    CONSTRAINT CK_Orders_qty    CHECK (qty > 0),
    CONSTRAINT CK_Orders_price  CHECK (price >= 0),
    CONSTRAINT CK_Orders_status CHECK (status IN ('NEW','PAID','SHIPPED','CANCELLED')),
    CONSTRAINT CK_Orders_dates  CHECK (shipped_at IS NULL OR shipped_at >= placed_at)
);

INSERT INTO dbo.Orders VALUES (1, 10, 'NEW', 0, 5.00, SYSUTCDATETIME(), NULL);  -- fails CK_Orders_qty
```
The NULL trap:
```sql
CREATE TABLE t (age INT NULL CONSTRAINT CK_age CHECK (age >= 0));
INSERT INTO t VALUES (NULL);   -- succeeds: age >= 0 is UNKNOWN, not FALSE
```
To forbid NULL as well, add NOT NULL. Adding a rule to a large live table:
```sql
ALTER TABLE dbo.Orders WITH NOCHECK ADD CONSTRAINT CK_x CHECK (...);   -- no scan, but untrusted
ALTER TABLE dbo.Orders WITH CHECK CHECK CONSTRAINT CK_x;               -- validate, re-trust
```
PostgreSQL equivalents: `ADD CONSTRAINT ... CHECK (...) NOT VALID;` then `VALIDATE CONSTRAINT` (takes a weaker lock).

### 5. Under the hood
- NOT NULL is stored as column metadata; in many engines it is also a physical optimization (no null-bitmap entry needed; fixed-width rows).
- CHECK expressions are compiled and run in the DML path; they may only reference the same row (no subqueries in SQL Server/PostgreSQL), which is why cross-table rules need FKs or triggers.
- **Optimizer use:** a *trusted* CHECK lets the optimizer prove contradictions (`WHERE qty < 0` → constant scan, no I/O) and enables partition elimination in partitioned views. NOT NULL lets it drop `IS NULL` branches, choose anti-joins safely (`NOT IN` over a NOT NULL column is equivalent to `NOT EXISTS`) and improve cardinality estimates: for a nullable column, `col < 'B' OR col >= 'C'` excludes NULL rows, while for NOT NULL it covers the entire column.
- An *untrusted* constraint (added WITH NOCHECK, or disabled/re-enabled without CHECK) is still enforced for new data but is ignored for optimization — plans can silently get worse.
- Scalar UDFs in CHECK constraints are evaluated per row and prevent parallelism for DML in some engines.

### 6. Performance and design implications
- Per-row cost of a simple CHECK is negligible; the real cost is on ALTER (full scan, possibly a schema lock on a big table).
- NOT NULL + DEFAULT on a new column: in modern SQL Server, PostgreSQL 11+ and MySQL 8 it is a metadata-only operation for constants; in older versions it rewrites the table.
- Prefer many small named constraints over one monster expression: the error identifies the failing rule.
- Value lists that change often belong in a lookup table + FK, not a hard-coded CHECK (changing a CHECK needs DDL).
- Monitor untrusted constraints: `SELECT name FROM sys.check_constraints WHERE is_not_trusted = 1`.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `col type NOT NULL` | Mandatory value | Anything always known |
| `CHECK (expr)` | Row-level validity rule | Ranges, enumerations, date order |
| `DEFAULT value` | Value when omitted | With NOT NULL |
| `WITH NOCHECK` / `NOT VALID` | Skip validating old rows | Big-table rollout, legacy data |
| `WITH CHECK CHECK CONSTRAINT` / `VALIDATE CONSTRAINT` | Validate and re-trust | After cleanup |
| `NOCHECK CONSTRAINT` | Disable temporarily | Bulk loads (then re-enable WITH CHECK) |
| `sys.check_constraints` | Inspect | Audit trust and definitions |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Rule is stable and row-local | Rule depends on other tables → FK/trigger |
| Value list fixed (status codes) | List changes weekly → lookup table |
| NOT NULL on facts always known | Genuinely optional → allow NULL, don't invent sentinel values |
| Validate immediately | Table is huge and live → NOT VALID then validate off-peak |

### 9. Common mistakes
- Believing CHECK rejects NULL → NULLs slip through → add NOT NULL or `IS NOT NULL` in the check.
- Using sentinel values (`'N/A'`, `-1`, `1900-01-01`) instead of NULL → bogus aggregates → allow NULL, handle with COALESCE.
- Disabling constraints for a load and re-enabling without CHECK → untrusted constraints and worse plans → `WITH CHECK CHECK`.
- Putting validation only in the app → other writers bypass it → constrain in the DB too.
- Hard-coding a long enum in many CHECKs → painful changes → lookup table.
- Unnamed constraints → obscure error messages and drift between environments → name them.

### 10. 🎯 Interview kit
**30-second answer:** NOT NULL makes a column mandatory; CHECK enforces a boolean rule on each row. They enforce domain integrity inside the database. A CHECK only rejects when the expression is FALSE, so NULL passes unless the column is also NOT NULL.

**Structure a longer answer:** clarify → definitions and three-valued logic → examples → optimizer benefits (trusted constraints, contradiction detection) → rollout on large tables (NOCHECK/NOT VALID).

**Likely follow-ups:**
- *Does CHECK (x > 0) allow NULL?* → yes.
- *Can a CHECK reference another table?* → not directly; use FK or trigger.
- *Why does the optimizer care?* → trusted constraints let it prune and estimate better.
- *Disable constraints for a bulk load?* → possible, then revalidate WITH CHECK to re-trust.
- *CHECK vs lookup table?* → volatility of the value set.

**Signals interviewers look for:** NULL/UNKNOWN fluency, awareness of trusted/untrusted, practical rollout.

### 11. Check yourself
- [ ] Explain why `CHECK (age >= 0)` accepts NULL.
- [ ] Add a CHECK to a 500M-row table with minimal blocking, then validate it.
- [ ] Find untrusted constraints in a database and say why they matter.
- [ ] Decide CHECK vs lookup table for order status.
<!-- explainer:end -->

## Practice questions

- [Basic] [What does the NOT NULL constraint do and where is it applied?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10035_What_does_the_NOT_NULL_constraint_do_and_where_is_it_applied.md>)
- [Basic] [How do you view all constraints defined on a table in your chosen RDBMS?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10194_How_do_you_view_all_constraints_defined_on_a_table_in_your_c.md>)
- [Intermediate] [How would you ensure a column only accepts values from a fixed set such as 'ACTIVE', 'INACTIVE', 'PENDING'?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10177_How_would_you_ensure_a_column_only_accepts_values_from_a_fix.md>)
- [Intermediate] [Write a CHECK constraint ensuring end_date is greater than or equal to start_date in a bookings table.](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10189_Write_a_CHECK_constraint_ensuring_end_date_is_greater_than_o.md>)
- [Intermediate] [What happens if you add a NOT NULL constraint to a column that already contains NULL values?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10227_What_happens_if_you_add_a_NOT_NULL_constraint_to_a_column_th.md>)
- [Deep] [You need to add a NOT NULL column with a default to a 400M-row table with zero downtime. How do you avoid a full-table rewrite and long metadata lock in PostgreSQL versus MySQL?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10037_You_need_to_add_a_NOT_NULL_column_with_a_default_to_a_400M-r.md>)
- [Deep] [A NOT NULL constraint addition is rewriting a huge table in an old Postgres version. How did PG11+ change this for defaults, and what's your workaround pre-11?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10080_A_NOT_NULL_constraint_addition_is_rewriting_a_huge_table_in_.md>)
- [Deep] [How do you decide constraint placement — DB-enforced vs application-enforced vs both — for a system with multiple writer services? What's your rule of thumb?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10142_How_do_you_decide_constraint_placement_DB-enforced_vs_applic.md>)

## Related concepts

- [Adding Constraints Online](<../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Adding_Constraints_Online.md>) *(PostgreSQL Platform)*
- [Unique Constraints for Dedup](<../../SQL_Language/SQL_Fundamentals/Unique_Constraints_for_Dedup.md>) *(SQL Fundamentals)*
- [Normal Forms](<../../Database_Internals/DBMS_Concepts/Normal_Forms.md>) *(DBMS Concepts)*

## References

- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 14 Query Design Analysis
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Manipulating Data › Inserting Rows into Tables with Default Column Values
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — 13. Implementing Logic in the Database › Adding Check Constraints to a Table
- *Azure Data Fundamentals* — Michael John Pena — Relational Data Concepts › Exam Tip
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Quality Design Patterns › Solution
- *Data Engineering Design Patterns* — Bartosz Konieczny — Data Quality Design Patterns › Examples
- *Practical Graph Structures in SQL Server and Azure SQL* — Louis Davidson — SQL Graph Tables Extended Topics › Integrity Constraints and Indexes
- *Practical Graph Structures in SQL Server and Azure SQL* — Louis Davidson — SQL Graph Tables Extended Topics › Additional Constraints
- *SQL Server Tutorial* (YouTube series) — Part 4: Adding a default constraint ([watch](https://www.youtube.com/watch?v=dwSqHhMl32Y))
- *SQL Server Tutorial* (YouTube series) — Part 6: Adding a check constraint ([watch](https://www.youtube.com/watch?v=9Zj5ODhv0b0))

