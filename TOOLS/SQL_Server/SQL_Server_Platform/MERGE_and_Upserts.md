---
id: "TOOLS/SQL_Server/SQL_Server_Platform/MERGE_and_Upserts"
level: 4
type: "concept"
topic: "TOOLS/SQL_Server/SQL_Server_Platform"
tools: ["SQL_Server"]
---

# MERGE and Upserts

**L4 Advanced** · [↑ SQL Server Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** An upsert means "update if the row exists, insert if not"; `MERGE` does it in one statement but has well-known concurrency and bug pitfalls, so many teams use an explicit `UPDATE` + `INSERT` inside a transaction with the right locking hints instead.

### 1. The big picture
Loading changes into a target table — from staging, an API batch or a CDC feed — requires deciding per row whether it is new, changed or (optionally) deleted. T-SQL offers `MERGE`, which joins a source to a target and applies `WHEN MATCHED` / `WHEN NOT MATCHED` actions, and the older two-statement pattern. Identity and sequence objects supply surrogate keys for inserted rows, and the `OUTPUT` clause lets you capture what happened.

### 2. Core concepts and vocabulary
- **Upsert** — update-or-insert semantics keyed on a business key.
- **`MERGE`** — single statement combining `INSERT`, `UPDATE`, `DELETE` against a target based on a join to a source.
- **`WHEN MATCHED`** — source and target rows join; typically `UPDATE` (or `DELETE`).
- **`WHEN NOT MATCHED [BY TARGET]`** — source row has no target row; `INSERT`.
- **`WHEN NOT MATCHED BY SOURCE`** — target row has no source row; `UPDATE`/`DELETE` (sync scenario).
- **`OUTPUT` / `$action`** — returns inserted/deleted images and which action ran.
- **`IDENTITY` / `SEQUENCE`** — auto-generated numbers for surrogate keys.
- **`HOLDLOCK` / `SERIALIZABLE`** — hint that holds key-range locks to prevent races.
- **Idempotent load** — re-running the same batch gives the same end state.

### 3. How it works, step by step
1. The engine joins source to target using the `ON` predicate (a full outer join in effect).
2. For each joined row it picks a `WHEN` branch, evaluating any extra `AND` condition on the branch.
3. It applies changes in a single statement, in one transaction, firing triggers once per action type.
4. If more than one source row matches the same target row in a `WHEN MATCHED ... UPDATE`, the statement fails (error 8672) — the source must be unique on the join key.
5. `OUTPUT` returns rows at the end, with `$action` = `INSERT`/`UPDATE`/`DELETE`.
6. The statement must end with a semicolon.

### 4. Worked example
```sql
-- Staging -> dimension upsert, capturing what changed
CREATE TABLE #changes (Action nvarchar(10), CustomerKey int, Email nvarchar(200));

MERGE dbo.Customer WITH (HOLDLOCK) AS t
USING (SELECT CustomerKey, Email, City FROM stg.Customer) AS s
   ON t.CustomerKey = s.CustomerKey
WHEN MATCHED AND (t.Email <> s.Email OR t.City <> s.City) THEN
     UPDATE SET Email = s.Email, City = s.City
WHEN NOT MATCHED BY TARGET THEN
     INSERT (CustomerKey, Email, City) VALUES (s.CustomerKey, s.Email, s.City)
OUTPUT $action, inserted.CustomerKey, inserted.Email INTO #changes;

-- Equivalent explicit pattern
BEGIN TRAN;
UPDATE t SET Email = s.Email, City = s.City
FROM dbo.Customer t WITH (UPDLOCK, SERIALIZABLE)
JOIN stg.Customer s ON s.CustomerKey = t.CustomerKey;

INSERT dbo.Customer (CustomerKey, Email, City)
SELECT s.CustomerKey, s.Email, s.City
FROM stg.Customer s
WHERE NOT EXISTS (SELECT 1 FROM dbo.Customer t WHERE t.CustomerKey = s.CustomerKey);
COMMIT;
```
Rows whose attributes are unchanged are skipped by the extra `AND` condition, so no useless writes or log. Note `NULL` columns need `IS DISTINCT FROM` (2022+) or explicit `ISNULL` handling for the change test.

### 5. Under the hood
- The plan shows a join (often merge or hash) feeding a `Table Merge` operator, which dispatches insert/update/delete per row; an index on the target's join key is essential.
- `MERGE` takes update locks on matched rows and, without a hint, only protects keys it actually read; two concurrent sessions can both see "not matched" and both insert, causing a duplicate-key error. `HOLDLOCK` (serializable) takes key-range locks to close the gap.
- Because a single statement wraps all branches, a failed row rolls back the entire statement.
- Triggers see combined `inserted`/`deleted` tables for each action type separately; `OUTPUT ... INTO` is incompatible with some trigger situations.
- Documented and long-running community reports describe `MERGE` bugs (e.g. filtered indexes, indexed views, ordering of `OUTPUT`), which is why some shops ban it for critical paths.

### 6. Performance and design implications
- Index the join key on both sides; stage and dedupe the source first.
- Keep batches moderate (10k–100k rows) to bound log growth and lock escalation.
- Skip no-op updates with a change predicate; this reduces log, replication and CDC noise.
- Measure with `SET STATISTICS IO`, `sys.dm_exec_requests` waits (`LCK_M_*`) and the actual plan.
- For high-concurrency single-row upserts, `UPDATE ... ; IF @@ROWCOUNT = 0 INSERT` under `UPDLOCK, SERIALIZABLE` is lighter than `MERGE`.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `MERGE ... WHEN MATCHED/NOT MATCHED` | One-statement upsert/sync | Batch loads from staging |
| `WITH (HOLDLOCK)` | Serializable key-range locking on target | Concurrent upserts |
| `OUTPUT $action, inserted.*` | Capture changes | Auditing, SCD logs |
| `UPDATE ... WHERE @@ROWCOUNT = 0` pattern | Explicit upsert | Single-row, hot paths |
| `IDENTITY` / `SEQUENCE` + `NEXT VALUE FOR` | Surrogate keys | Inserted rows |
| `SET XACT_ABORT ON` | Roll back on error | Multi-step upsert procs |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `MERGE` for set-based dimension/staging loads with unique sources | High concurrency without `HOLDLOCK`; heavy triggers |
| Explicit `UPDATE`+`INSERT` for transparent locking and simpler plans | Large sync including deletes (more code) |
| `INSERT ... WHERE NOT EXISTS` for insert-only | Rows may change |
| `DELETE`+`INSERT` swap | FK dependencies or history needed |

### 9. Common mistakes
- No `HOLDLOCK` under concurrency → duplicate key or lost update → add hint and unique constraint.
- Duplicate keys in the source → error 8672 → dedupe with `ROW_NUMBER()` first.
- Forgetting trailing `;` → syntax error.
- Using `NOT MATCHED BY SOURCE ... DELETE` without a filter → wipes unrelated target rows → constrain the target.
- Comparing nullable columns with `<>` → missed updates → null-safe comparison.
- Relying on `@@IDENTITY` for new keys → wrong in triggers → use `OUTPUT` or `SCOPE_IDENTITY()`.

### 10. 🎯 Interview kit
**30-second answer:** An upsert updates existing rows and inserts missing ones. `MERGE` expresses this in one statement via source/target join and `WHEN` branches; because it has concurrency traps, you pair it with `HOLDLOCK` or use an explicit update-then-insert in a transaction.

**Structure a longer answer:** clarify (batch or single row? concurrency?) → mechanism (join, branches, OUTPUT) → trade-offs → scenario (staging to dimension load).

**Likely follow-ups:**
- *Why does MERGE fail with duplicates?* → one target row matched by many source rows (8672).
- *How do you prevent races?* → `HOLDLOCK`/serializable, unique index.
- *How do you capture inserted identities?* → `OUTPUT inserted.ID`.
- *Identity vs sequence?* → sequence is object-level, shareable, can be pre-fetched.

**Signals interviewers look for:** concurrency awareness, idempotence, null-safe change detection.

### 11. Check yourself
- [ ] Write a `MERGE` that inserts, updates only changed rows, and outputs `$action`.
- [ ] Explain how two sessions can both insert the same key without `HOLDLOCK`.
- [ ] Rewrite the upsert as `UPDATE` + `INSERT` in a transaction.
- [ ] Describe how you dedupe the source before merging.
<!-- explainer:end -->

## Implementations (tools)

- [SQL Server](<../_index.md>)

## Practice questions

- [Basic] [What are the four system databases in SQL Server (master, model, msdb, tempdb) and what is each one used for?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41461_What_are_the_four_system_databases_in_SQL_Server_master_mode.md>)
- [Basic] [What is a view in SQL Server and what problems does it help solve?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41482_What_is_a_view_in_SQL_Server_and_what_problems_does_it_help_.md>)
- [Basic] [What does the GRANT statement do and how do you control permissions in SQL Server?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41531_What_does_the_GRANT_statement_do_and_how_do_you_control_perm.md>)
- [Basic] [How do you add, modify, and drop a column on an existing table using ALTER TABLE?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41544_How_do_you_add_modify_and_drop_a_column_on_an_existing_table.md>)
- [Intermediate] [How do you create a stored procedure with input parameters in SQL Server?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41478_How_do_you_create_a_stored_procedure_with_input_parameters_i.md>)
- [Intermediate] [What is a trigger in SQL Server and when would you use an AFTER trigger?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41498_What_is_a_trigger_in_SQL_Server_and_when_would_you_use_an_AF.md>)
- [Intermediate] [What is a composite key and when would you use one instead of a single-column key?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41505_What_is_a_composite_key_and_when_would_you_use_one_instead_o.md>)
- [Intermediate] [What is a cursor in SQL Server and why is set-based logic usually preferred over cursors?](<../../../PRACTICE/TOOLS/SQL_Server/SQL_Server_Platform/Q-41540_What_is_a_cursor_in_SQL_Server_and_why_is_set-based_logic_us.md>)

## Related concepts

- [T-SQL Basics](<../T-SQL/T-SQL_Basics.md>) *(T-SQL)*
- [Dynamic SQL](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Dynamic_SQL.md>) *(DBMS Concepts)*
- [Scalar Subqueries](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Scalar_Subqueries.md>) *(SQL Fundamentals)*

## References

- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore metadata › CHAPTER 7
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — BatCh exeCution H XE UTION › How Does Batch Mode Work?
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — sys.fulltextindexcolumns › Storage Basics
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Pages
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — System Columns
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › CRUD Operations
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 2: Data Types, NULLs and Conversions › Identity and Sequence Objects
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 2: Data Types, NULLs and Conversions › Sequences
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Execution Plan
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Background to T-SQL querying and programming
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Introducing SQL Server › 1.2 An overview of SQL Server
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — SQL Server installation › 8.9 #41 Thinking configuration management doesn't apply to SQL Server
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Expanding on Data Type Concepts › FileTables
- *SQL Server Tutorial* (YouTube series) — Part 10: Select statement in sql server ([watch](https://www.youtube.com/watch?v=R9pXnHIFj_8))
- *SQL Server Tutorial* (YouTube series) — Part 54: Pivot in sql server 2008 ([watch](https://www.youtube.com/watch?v=h3BtudZehuo))
- *SQL Server Tutorial* (YouTube series) — Part 65: List all tables in a sql server database using a query ([watch](https://www.youtube.com/watch?v=z1HFiXt6KKQ))
- *SQL Server Tutorial* (YouTube series) — Part 97: Select into in sql server ([watch](https://www.youtube.com/watch?v=1XQHULQcjG0))

