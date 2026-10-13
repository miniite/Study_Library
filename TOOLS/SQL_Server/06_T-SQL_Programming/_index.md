---
id: "TOOLS/SQL_Server/06_T-SQL_Programming"
level: 3
type: "curriculum"
curriculum: "sql_server"
---

# T-SQL Programming

**L3 Practitioner** · [↑ SQL Server](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** T-SQL is SQL Server's dialect of SQL. It is a set-based, declarative language built on set theory and predicate logic, extended with procedural features (variables, flow control, error handling). Writing good T-SQL means thinking in sets, respecting three-valued logic, and letting the optimizer do the "how".

### 1. The big picture
The earlier blocks covered how SQL Server stores data, finds it with indexes, estimates with statistics, builds plans and isolates concurrent sessions. This block is about the language you use to drive all of that: **Transact-SQL (T-SQL)**. Every query, data change, stored procedure and admin script you write is T-SQL.

T-SQL looks easy. You can write a working `SELECT` within minutes. The trap is that it looks like an imperative language but is not. In C# you tell the computer *how* to do things, step by step, in loops. In SQL you describe *what* result you want, as a set, and the query optimizer chooses the steps. Most bad T-SQL comes from developers who bring loop-by-loop thinking into a set-based language: cursors where one `UPDATE` would do, scalar functions called per row, `NOT IN` against nullable columns, or functions wrapped around indexed columns.

The child lessons go deep on each area:
- **Logical Query Processing**: the order in which clauses are *conceptually* evaluated (FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY).
- **Joins and Set Operators**: combining tables horizontally (joins) and vertically (UNION, INTERSECT, EXCEPT).
- **Table Expressions**: derived tables, CTEs, views, inline TVFs, APPLY.
- **Window Functions**: ranking, aggregates and offsets over a window of rows without collapsing them.
- **Data Modification**: INSERT, UPDATE, DELETE, MERGE, OUTPUT, and large batch changes.
- **Programmable Objects**: variables, batches, flow control, cursors, temp tables, dynamic SQL, error handling, procedures, functions, triggers.
- **JSON and XML**, **Temporal and Graph Tables**: modern data shapes.

### 2. Core concepts and vocabulary
- **SQL**: Structured Query Language, an ANSI/ISO standard for querying and managing data in relational databases.
- **T-SQL (Transact-SQL)**: Microsoft's dialect of SQL, used in SQL Server, Azure SQL Database, Azure SQL Managed Instance and Azure Synapse. It follows the standard closely and adds extensions: `TOP`, variables, `IF`/`WHILE`, `TRY…CATCH`, `OUTPUT`, `APPLY`, and many built-in functions.
- **Relational model**: a way to represent data proposed by E. F. Codd. It is based on two branches of mathematics: **set theory** and **predicate logic**.
- **Relation**: the mathematical term behind a table. It has a **heading** (a set of attributes, i.e. columns, each with a name and type) and a **body** (a set of tuples, i.e. rows).
- **Set**: a collection of distinct elements with no order. This is why a table has no guaranteed row order, and why a query without `ORDER BY` has no guaranteed order either.
- **Predicate**: an expression that is true or false for a given row, such as `orderdate >= '20260101'`. Predicates are used to filter rows, define constraints and join tables.
- **Three-valued logic**: SQL's predicates can be TRUE, FALSE or **UNKNOWN**. UNKNOWN appears whenever a `NULL` (missing value) is compared.
- **NULL**: a marker for a missing value. It is not a value, not zero and not an empty string.
- **Declarative data integrity**: rules enforced by the table definition itself: data types, nullability, primary key, unique, foreign key, check and default constraints.
- **Procedural data integrity**: rules enforced by code, such as triggers or stored procedures.
- **Batch**: a group of statements sent to the server together. In SSMS and sqlcmd, `GO` separates batches. `GO` is a client tool command, not T-SQL.
- **DDL / DML / DCL**: Data Definition Language (`CREATE`, `ALTER`, `DROP`), Data Manipulation Language (`SELECT`, `INSERT`, `UPDATE`, `DELETE`, `MERGE`), Data Control Language (`GRANT`, `REVOKE`, `DENY`).
- **Set-based vs. row-based (RBAR)**: set-based code describes an operation on a whole set in one statement. Row-based code ("row by agonizing row") loops over rows one at a time with cursors or `WHILE` loops.

### 3. How it works, step by step
**From text to result:**
1. The client sends a batch (all text up to the next `GO`).
2. The batch is parsed (syntax check) and bound (names resolved to objects and columns, types checked).
3. The optimizer turns each statement into a physical plan, or reuses a cached one.
4. The execution engine runs the plan and streams rows back.

Logically, the query is defined by set operations in a fixed clause order (covered in the Logical Query Processing lesson). Physically, the optimizer may do things in a completely different order, as long as the result is the same. This separation is the key idea: you describe the *result*, the engine chooses the *method*.

**Thinking in T-SQL, the four habits the books keep returning to:**

| Habit | Why | Example |
|---|---|---|
| Think in sets, not loops | One statement lets the optimizer choose joins, parallelism and batch mode; a loop forces thousands of tiny plans | One `UPDATE … FROM … JOIN` instead of a cursor updating row by row |
| Respect NULL | Comparisons with NULL yield UNKNOWN, and filters keep only TRUE | `WHERE col <> 'X'` silently drops rows where `col` is NULL |
| Never rely on order without ORDER BY | Tables are sets; order depends on the plan | A view with `TOP 100 PERCENT … ORDER BY` returns rows in any order |
| Keep predicates "sargable" | An index seek needs the column alone on one side of the predicate | `WHERE OrderDate >= '20260101'` can seek; `WHERE YEAR(OrderDate) = 2026` scans |

**Indexes and queries.** Your T-SQL doesn't name indexes, but how you write predicates decides whether the optimizer can use them. A composite index on `(LastName, FirstName, MiddleName)` works like a phone book. A filter on `LastName` can seek straight to the right section. A filter on `FirstName` alone forces a read of every entry, because the index is not ordered by first name. A filter on a column with no index at all forces a scan of the whole table.

### 4. Worked example
Create a small table with declarative integrity, then compare a row-based and a set-based way to apply a price change.

```sql
CREATE TABLE dbo.Products
(
    ProductID   int           NOT NULL CONSTRAINT PK_Products PRIMARY KEY,
    Name        nvarchar(100) NOT NULL CONSTRAINT UQ_Products_Name UNIQUE,
    CategoryID  int           NOT NULL,
    Price       decimal(10,2) NOT NULL CONSTRAINT CK_Products_Price CHECK (Price >= 0),
    Discontinued bit          NOT NULL CONSTRAINT DF_Products_Disc DEFAULT (0)
);

-- 100,000 rows of test data
INSERT dbo.Products (ProductID, Name, CategoryID, Price)
SELECT n, CONCAT(N'Product ', n), n % 20, (n % 500) + 0.99
FROM (SELECT TOP (100000) ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) AS n
      FROM sys.all_objects a CROSS JOIN sys.all_objects b) AS t;

SET STATISTICS TIME, IO ON;

-- Row-based: a cursor that raises category 5 prices by 10%
DECLARE @id int;
DECLARE c CURSOR LOCAL FAST_FORWARD FOR
    SELECT ProductID FROM dbo.Products WHERE CategoryID = 5;
OPEN c;
FETCH NEXT FROM c INTO @id;
WHILE @@FETCH_STATUS = 0
BEGIN
    UPDATE dbo.Products SET Price = Price * 1.10 WHERE ProductID = @id;
    FETCH NEXT FROM c INTO @id;
END;
CLOSE c; DEALLOCATE c;

-- Set-based: the same change in one statement
UPDATE dbo.Products
SET Price = Price * 1.10
WHERE CategoryID = 5;
```
**What you see:** the cursor runs 5,000 separate `UPDATE` statements, each with its own seek, its own log records and its own lock requests. In the messages tab there are thousands of `Table 'Products'` lines. The set-based `UPDATE` produces one plan, one scan and one line of statistics. It usually runs one to two orders of magnitude faster. Run both inside an explicit transaction and roll back if you want to repeat the test.

**NULL trap, in the same database:**
```sql
CREATE TABLE #T (Code char(1) NULL);
INSERT #T VALUES ('A'), ('B'), (NULL);
SELECT COUNT(*) FROM #T WHERE Code <> 'A';                    -- 1, not 2
SELECT COUNT(*) FROM #T WHERE Code <> 'A' OR Code IS NULL;    -- 2
```
The NULL row is neither equal nor unequal to 'A'. The predicate is UNKNOWN, and `WHERE` keeps only TRUE.

### 5. Under the hood
- **Parsing and binding happen per batch.** A batch with a syntax error doesn't run at all. A batch that references a missing table is compiled with *deferred name resolution*: the error only appears when that statement runs. Some statements must be first in their batch (`CREATE PROCEDURE`, `CREATE VIEW`, `CREATE FUNCTION`, `CREATE TRIGGER`). Variables exist only inside their batch, so a variable declared before `GO` is gone after it.
- **Semicolons.** T-SQL only *requires* a semicolon in a few places: before a `WITH` that starts a CTE, at the end of `MERGE`, and before `THROW` in some positions. Leaving them out elsewhere is deprecated. Always terminating statements avoids confusing parser errors.
- **Plan caching.** Each statement's plan is cached and reused. Parameterized code (procedures, `sp_executesql`) reuses plans. Ad hoc strings with literals compile separately. That ties T-SQL style directly to CPU use and plan cache bloat (see the Query Processing block).
- **Declarative constraints help the optimizer.** A trusted foreign key lets the optimizer remove a join that isn't needed (join elimination). A trusted check constraint lets it skip partitions or return an empty result without reading data. `NOT NULL` lets `NOT IN` plans skip expensive NULL handling.

### 6. Performance and design implications
- Set-based statements let the optimizer use hash and merge joins, parallelism, batch mode and minimal logging. Loops can't use any of these.
- A predicate that wraps a column in a function or implicit conversion is not sargable. It turns seeks into scans. Watch for `CONVERT_IMPLICIT` warnings in plans.
- Return only the columns you need. `SELECT *` defeats covering indexes, sends more data over the network and breaks code when columns are added.
- Measure with `SET STATISTICS IO, TIME ON`, the actual execution plan, and `sys.dm_exec_query_stats` for cached workloads.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `GO` (client) | Ends a batch in SSMS/sqlcmd; `GO 10` repeats it | Script separation, test loops |
| `SET STATISTICS IO, TIME ON` | Logical reads and CPU/elapsed time per statement | Comparing query versions |
| `SET NOCOUNT ON` | Stops "n rows affected" messages | Start of every procedure |
| `SET XACT_ABORT ON` | Any runtime error rolls back the whole transaction | Procedures with transactions |
| `SET ANSI_NULLS ON` | Standard NULL comparison (required for indexed views, filtered indexes) | Always (default) |
| `SET QUOTED_IDENTIFIER ON` | `"name"` is an identifier, not a string | Always (default) |
| `ALTER DATABASE … SET COMPATIBILITY_LEVEL` | Controls optimizer and language behavior | After upgrades, after testing |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Set-based statement: almost always | A single huge statement fills the log or blocks for minutes → batch it (Data Modification lesson) |
| Cursor/loop: per-row calls to a procedure you can't change, admin scripts over databases, running totals before SQL 2012 | Business data changes: replace with a set-based statement |
| Declarative constraints | Rules that span tables in complex ways may need triggers; keep them small |
| Procedural logic in T-SQL | Heavy string manipulation or complex algorithms may belong in the application |

### 9. Common mistakes
- Using cursors for data changes → thousands of plans, locks and log records → one set-based statement with a join.
- `WHERE col <> value` on a nullable column → NULL rows silently excluded → add `OR col IS NULL` or make the column `NOT NULL`.
- Expecting order without `ORDER BY` → order changes after an index or parallel plan appears → always add `ORDER BY` to the outermost query.
- `SELECT *` in production code → wider reads, broken inserts after schema change → list columns.
- Wrapping indexed columns in functions → scans → rewrite as a range (`col >= @start AND col < @end`).
- Forgetting the semicolon before `WITH` → confusing syntax errors → terminate every statement.

### 10. 🎯 Interview kit
**30-second answer:** T-SQL is Microsoft's SQL dialect. It's declarative and set-based, rooted in set theory and predicate logic. You describe the result and the optimizer chooses the method. It adds procedural features like variables, flow control and TRY/CATCH. Good T-SQL means thinking in sets, handling NULL's three-valued logic correctly, never relying on row order without ORDER BY, and writing sargable predicates.

**Structure a longer answer:** clarify (querying vs. programming) → set-based core and the relational model → three-valued logic and NULL → procedural extensions and when to use them → a cursor you replaced with one `UPDATE`, and the speedup you measured.

**Likely follow-ups:**
- *Why avoid cursors?* → per-row overhead: plans, locks, log, round trips. No parallelism or batch mode.
- *What is three-valued logic?* → TRUE/FALSE/UNKNOWN. `WHERE` and `ON` keep only TRUE; `CHECK` constraints reject only FALSE.
- *Is `GO` a T-SQL statement?* → No, it's a client batch separator.
- *What does sargable mean?* → the predicate can use an index seek because the column stands alone.

**Signals interviewers look for:** you think in sets; you mention NULL pitfalls unprompted; you connect query style to plans and indexes; you know `GO` is not T-SQL.

### 11. Check yourself
- [ ] Explain why a table has no order and what that means for `TOP` without `ORDER BY`.
- [ ] Show a predicate that returns a different count because of NULL, and fix it two ways.
- [ ] Rewrite a cursor-based update as a single statement and compare `STATISTICS IO`.
- [ ] Name three declarative constraints and one way each helps the optimizer.
- [ ] Explain why a variable declared before `GO` is unknown after it.
<!-- explainer:end -->

## In this section (study in order)

1. [Logical Query Processing](<Logical_Query_Processing/_index.md>) (L4 Advanced)
2. [Joins Subqueries and Set Operators](<Joins_and_Set_Operators/_index.md>) (L4 Advanced)
3. [Table Expressions](<Table_Expressions/_index.md>) (L4 Advanced)
4. [Window Functions](<Window_Functions/_index.md>) (L4 Advanced)
5. [Data Modification and MERGE](<Data_Modification/_index.md>) (L4 Advanced)
6. [Procedures Functions and Triggers](<Programmable_Objects/_index.md>) (L4 Advanced)
7. [Temporal and Graph Tables](<Temporal_and_Graph_Tables/_index.md>) (L4 Advanced)
8. [JSON and XML](<JSON_and_XML/_index.md>) (L4 Advanced)

## References

- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Background to T-SQL querying and programming
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › Introduction
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — 12. Understanding T-SQL Programming Logic
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › Conclusion
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — T-SQL development › 5 T-SQL development
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Writing Simple SELECT Queries › Thinking About Performance
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Ultimate T-SQL for SQL Server Professionals › Preface
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 3: SELECT Statements and Filtering Logic › Introduction
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Writing Advanced Queries › Writing a Recursive Query
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › Key Takeaways

