---
id: "01_Databases_and_SQL/SQL_Language/SQL_Command_Types/DML_Statements"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Command_Types"
tools: []
---

# DML Statements

**L4 Advanced** · [↑ SQL Command Types](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** SQL statements fall into families — DDL defines objects, DML reads and changes data (`SELECT`, `INSERT`, `UPDATE`, `DELETE`, `MERGE`), DCL controls permissions, TCL controls transactions — and DML is set-based: each statement operates on a whole set of rows atomically.

### 1. The big picture
SQL is a declarative language built on set theory: you describe *which* rows you want or change and the engine decides *how*. The statement families tell you what a command touches:

- **DDL (Data Definition Language)** — structure: `CREATE`, `ALTER`, `DROP`, `TRUNCATE`, `RENAME`.
- **DML (Data Manipulation Language)** — data: `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `MERGE` (plus `COPY`/`BULK INSERT` loaders).
- **DCL (Data Control Language)** — permissions: `GRANT`, `REVOKE` (`DENY` in SQL Server).
- **TCL (Transaction Control Language)** — `BEGIN`/`COMMIT`/`ROLLBACK`/`SAVEPOINT`.

This lesson focuses on DML and clarifies the boundaries with DDL (a classic interview topic). Day-to-day work is almost entirely DML, which is also what introductory certifications emphasise.

### 2. Core concepts and vocabulary
- **Set-based operation** — a statement acts on all qualifying rows as one unit; a row-at-a-time loop (cursor) is usually slower and less declarative.
- **`SELECT`** — reads data; the only DML statement that doesn't change data; three core parts: columns, source table(s), conditions.
- **`INSERT`** — adds rows (`VALUES`, `INSERT … SELECT`, `SELECT … INTO`/CTAS).
- **`UPDATE`** — changes column values of existing rows, optionally with `FROM`/join.
- **`DELETE`** — removes rows; see *DELETE vs TRUNCATE*.
- **`MERGE`** — upsert: insert/update/delete in one statement based on a match condition.
- **`OUTPUT` / `RETURNING`** — return affected rows (inserted/deleted values) from DML.
- **Autocommit** — each statement is its own transaction unless you open one.
- **Affected rows (`@@ROWCOUNT`)** — how many rows the last statement touched.
- **Idempotency** — running the statement twice leaves the same result (important for pipelines; `MERGE`/upsert vs blind `INSERT`).

### 3. How it works, step by step
**Statement lifecycle**
1. Parse and bind names (objects, columns, permissions).
2. Optimizer builds a plan; for DML it also plans index maintenance, constraint checks, triggers.
3. Execution takes locks (or row versions) and writes log records *before* modifying pages (write-ahead logging).
4. Constraints (`NOT NULL`, `CHECK`, `FOREIGN KEY`, `UNIQUE`) are validated; violation fails the whole statement (statement atomicity).
5. Commit makes the changes durable; rollback undoes them.

**Category boundaries**
- DDL changes the catalog (metadata). In SQL Server and PostgreSQL DDL is transactional (can be rolled back); in Oracle and MySQL it implicitly commits.
- `TRUNCATE` is classified as DDL even though it removes data; `SELECT … INTO` creates a table (DDL-like) while moving data.
- DCL is metadata too (permissions), not data.

### 4. Worked example
```sql
-- DDL: define
CREATE TABLE dbo.Employees (
    EmployeeID INT IDENTITY PRIMARY KEY,
    Name       NVARCHAR(100) NOT NULL,
    Salary     DECIMAL(10,2) CHECK (Salary >= 0),
    DeptID     INT NULL
);

-- DML: INSERT (single, multi-row, from a query)
INSERT INTO dbo.Employees (Name, Salary, DeptID) VALUES (N'Alice', 9000, 1), (N'Bob', 7000, 1);
INSERT INTO dbo.Employees (Name, Salary, DeptID)
SELECT Name, Salary, DeptID FROM dbo.Candidates WHERE Accepted = 1;

-- UPDATE with a filter and a join
UPDATE e SET Salary = Salary * 1.05
FROM dbo.Employees e JOIN dbo.Departments d ON d.DeptID = e.DeptID
WHERE d.Name = N'Engineering';

-- DELETE with OUTPUT (archive what was removed)
DELETE FROM dbo.Employees
OUTPUT deleted.EmployeeID, deleted.Name INTO dbo.EmployeesArchive(EmployeeID, Name)
WHERE DeptID IS NULL;

-- MERGE (upsert) -- terminate with semicolon
MERGE dbo.Employees AS t
USING dbo.StagingEmployees AS s ON t.EmployeeID = s.EmployeeID
WHEN MATCHED THEN UPDATE SET t.Salary = s.Salary
WHEN NOT MATCHED THEN INSERT (Name, Salary) VALUES (s.Name, s.Salary);

-- DCL
GRANT SELECT, INSERT ON dbo.Employees TO reporting_role;
```
After each: `SELECT @@ROWCOUNT` (or check the messages) to confirm the number of rows affected; wrap risky ones in `BEGIN TRAN … ROLLBACK` while testing.

### 5. Under the hood   *(optional at L4)*
- DML on an indexed table maintains every index: one `INSERT` writes the clustered row plus a row in each nonindexed structure; that's why many indexes slow writes.
- `UPDATE` may be performed in place or as delete+insert (when a key changes or the row doesn't fit); it also affects the log and triggers.
- Triggers see the `inserted` / `deleted` pseudo-tables (SQL Server) and fire once per statement, not per row.
- `MERGE` in SQL Server has known historical bugs and concurrency issues without `HOLDLOCK`; many teams prefer explicit `UPDATE` then `INSERT` inside a transaction.
- `SELECT` itself obeys the isolation level: it takes shared locks or reads row versions, and can block or be blocked.

### 6. Performance and design implications
- Batch DML (multi-row insert, bulk load) over row-by-row.
- Keep `WHERE` sargable and indexed on large `UPDATE`/`DELETE`; check the plan before running.
- Large modifications: batch (e.g. 5–50k rows) to limit lock escalation and log growth.
- Always `SELECT` with the same `WHERE` before an `UPDATE`/`DELETE`; use a transaction to verify.
- Prefer `INSERT … SELECT` or `COPY`/`BULK INSERT` for loads; both can be minimally logged under conditions.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SELECT` | Read | Everything |
| `INSERT … VALUES / SELECT` | Add rows | Single, multi-row, load |
| `UPDATE … SET … WHERE` | Change rows | Corrections, derived values |
| `DELETE … WHERE` | Remove rows | Retention |
| `MERGE` | Upsert | Sync from staging |
| `OUTPUT` / `RETURNING` | Capture affected rows | Auditing, ID retrieval |
| `BEGIN TRAN / COMMIT / ROLLBACK` | Transaction control | Multi-statement atomicity |
| `GRANT / REVOKE` | Permissions | Access control |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Set-based DML | Row-by-row logic is truly required (rare) |
| `MERGE` for sync | Concurrency requirements are strict (add `HOLDLOCK`) or you need triggers + `OUTPUT` complexity |
| Explicit transaction | Long-running user interaction inside it (locks held) |
| `TRUNCATE` for full clears | Triggers/FKs needed |

### 9. Common mistakes
- `UPDATE`/`DELETE` without `WHERE` → whole table changed → write the `SELECT` first, use a transaction, enable safe-update settings in clients.
- Treating DDL as always non-rollbackable (or always rollbackable) → surprises → know your engine.
- Forgetting that `NULL` comparisons need `IS NULL` → rows not matched.
- Assuming trigger fires per row → logic breaks for multi-row statements.
- Misusing `INSERT` without column list → breaks when the table changes.

### 10. 🎯 Interview kit
**30-second answer:** "DDL defines structure (`CREATE`, `ALTER`, `DROP`, `TRUNCATE`), DML manipulates data (`SELECT`, `INSERT`, `UPDATE`, `DELETE`, `MERGE`), DCL manages permissions (`GRANT`, `REVOKE`), TCL controls transactions. DML is set-based and transactional; DDL changes metadata and in some engines auto-commits."

**Structure a longer answer:** families → what each touches (data vs metadata) → transaction behaviour → performance considerations → examples.

**Likely follow-ups:**
- *DCL commands?* → `GRANT` and `REVOKE` (plus `DENY` in SQL Server).
- *Rename a table/column?* → `sp_rename` (SQL Server), `ALTER TABLE … RENAME` (PostgreSQL/MySQL).
- *Difference between `CREATE INDEX` and `DROP INDEX`?* → build vs remove an access structure; DDL, affects write cost and read plans.
- *Is `TRUNCATE` DML or DDL?* → DDL.

**Signals interviewers look for:** correct taxonomy, engine-specific awareness (transactional DDL), safe-change habits.

### 11. Check yourself
- [ ] Classify `TRUNCATE`, `SELECT INTO`, `GRANT`, `MERGE`, `COMMIT`.
- [ ] Write an `UPDATE` with a join and an `OUTPUT` clause.
- [ ] Explain which engines roll back DDL.
- [ ] Convert a cursor loop into one set-based statement.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is the difference between a DDL statement and a DQL statement, and where does SELECT fit?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22960_What_is_the_difference_between_a_DDL_statement_and_a_DQL_sta.md>)
- [Basic] [How do you rename a column and a table in SQL?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22965_How_do_you_rename_a_column_and_a_table_in_SQL.md>)
- [Basic] [What is the difference between DDL commands CREATE INDEX and DROP INDEX?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22995_What_is_the_difference_between_DDL_commands_CREATE_INDEX_and.md>)
- [Basic] [How would you update a column value based on values in the same row?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-23029_How_would_you_update_a_column_value_based_on_values_in_the_s.md>)
- [Intermediate] [What is the difference between the RENAME command and ALTER TABLE ... RENAME?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22967_What_is_the_difference_between_the_RENAME_command_and_ALTER_.md>)
- [Intermediate] [Is TRUNCATE a DDL or DML command and why?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-23014_Is_TRUNCATE_a_DDL_or_DML_command_and_why.md>)
- [Deep] [Can you ROLLBACK a TRUNCATE statement? Explain your answer.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22936_Can_you_ROLLBACK_a_TRUNCATE_statement_Explain_your_answer.md>)
- [Deep] [Is CREATE USER a DDL or DCL command? Explain the classification.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-23009_Is_CREATE_USER_a_DDL_or_DCL_command_Explain_the_classificati.md>)

## Related concepts

- [SQL vs NoSQL](<../SQL_Fundamentals/SQL_vs_NoSQL.md>) *(SQL Fundamentals)*
- [Schema Migration Risks](<../../Database_Internals/DBMS_Concepts/Schema_Migration_Risks.md>) *(DBMS Concepts)*
- [Dynamic SQL](<../../Database_Internals/DBMS_Concepts/Dynamic_SQL.md>) *(DBMS Concepts)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › Structure
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › Batch Processing and the GO Command
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › Data Definition Language (DDL)
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › RENAME
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Background to T-SQL querying and programming
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Data ��difi�ati�n
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Chapter 8. Queries, Modeling, and Transformation › Data definition language
- *Azure Data Fundamentals* — Michael John Pena — Relational Data Concepts › Basic SQL Queries
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Capacity Reservations › DDL Capabilities
- *Building Medallion Architectures* — Piethein Strengholt — Construct the Bronze Layer › SQL DDL Statements

