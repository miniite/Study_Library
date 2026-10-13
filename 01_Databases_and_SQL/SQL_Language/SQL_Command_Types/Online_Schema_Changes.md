---
id: "01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Online_Schema_Changes"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Command_Types"
tools: ["MySQL"]
---

# Online Schema Changes

**L4 Advanced** · [↑ SQL Command Types](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** An online schema change alters a live table without long blocking — by choosing operations that are metadata-only or concurrent, and otherwise by building a shadow copy, syncing changes and swapping, usually in expand → migrate → contract steps so old and new application versions both keep working.

### 1. The big picture
`ALTER TABLE` looks simple but on a big, busy table it can rewrite every row while holding a schema lock that blocks all readers and writers. The goal of "online" schema change is zero (or seconds of) downtime. Three families of technique exist: (1) *engine-native online operations* (metadata-only changes, `ONLINE = ON` index builds, concurrent index creation); (2) *shadow-table tools* that copy the table in the background and cut over (pt-online-schema-change, gh-ost, `pg_repack`, Spark/Delta rewrites); (3) *application-level patterns* (expand/contract) that make each step backward compatible. This lesson explains the vocabulary, the safe patterns and the DDL statements involved (`ALTER`, constraints, defaults/sequences, `CTAS`).

### 2. Core concepts and vocabulary
- **Metadata-only change** — only the catalog changes; no data is touched (instant). Example: adding a nullable column in SQL Server; adding a column with a constant default in PostgreSQL 11+ / SQL Server 2012+ Enterprise.
- **Size-of-data operation** — touches every row (changing a type, adding a `NOT NULL` column with a computed value in older engines).
- **Schema-modification (`Sch-M`) lock** — exclusive lock needed by most `ALTER`s; queued behind running queries and blocks everything behind it.
- **Online index build** — `CREATE INDEX … WITH (ONLINE = ON)` (SQL Server Enterprise/Azure) or `CREATE INDEX CONCURRENTLY` (PostgreSQL).
- **Shadow table / ghost table** — a new table with the target schema populated in the background.
- **Backfill** — copying/updating existing rows in batches after adding a column.
- **Expand / migrate / contract** — add new structure, move data/traffic, remove the old structure in a later release.
- **Constraint validation** — checking existing rows when a constraint is added (`WITH CHECK` vs `WITH NOCHECK`; PostgreSQL `NOT VALID` + `VALIDATE`).
- **`CTAS` (`CREATE TABLE AS SELECT`) / `SELECT … INTO`** — create-and-load in one statement; copies column definitions but *not* indexes, constraints or defaults (SQL Server's `SELECT INTO` keeps identity and NULL-ability only).
- **Sequence / default constraint** — `ALTER TABLE … ADD CONSTRAINT DF DEFAULT (NEXT VALUE FOR seq) FOR col` lets you add or drop a default later, unlike identity.

### 3. How it works, step by step
**A. Adding a column safely**
1. `ALTER TABLE Employees ADD Email NVARCHAR(100) NULL;` — metadata-only.
2. Deploy code that writes the column (tolerates NULL).
3. Backfill in batches (`UPDATE TOP (10000) … WHERE Email IS NULL`).
4. Add the constraint last: `ALTER TABLE … ALTER COLUMN Email NVARCHAR(100) NOT NULL` (scans the table; do off-peak) or a `CHECK … NOT VALID` then `VALIDATE`.

**B. Adding constraints without long blocks**
1. `ALTER TABLE child WITH NOCHECK ADD CONSTRAINT fk … FOREIGN KEY …` (SQL Server) or `NOT VALID` (PostgreSQL): enforced for new rows immediately, existing rows unchecked.
2. Validate later: `ALTER TABLE child WITH CHECK CHECK CONSTRAINT fk` / `VALIDATE CONSTRAINT` — lighter lock level in PostgreSQL.
3. Note: a `NOCHECK` constraint is *untrusted*; the optimizer can't use it until validated.

**C. Shadow table cutover (type change, big rewrite)**
1. Create `T_new` with the target schema.
2. Install triggers (or log reader) so inserts/updates/deletes on `T` are mirrored to `T_new`.
3. Copy existing rows in chunks, throttled by replica lag/IO.
4. Verify counts/checksums.
5. Atomically swap names (`sp_rename` or `RENAME`) in a short exclusive window.
6. Drop the old table later.

**D. Expand/contract for renames and splits**
1. Expand: add new column/table; write to both.
2. Migrate: backfill; switch reads.
3. Contract: stop writing the old; drop it a release later.

### 4. Worked example
```sql
-- SQL Server: add column, backfill, enforce
ALTER TABLE dbo.Employees ADD Email NVARCHAR(100) NULL;          -- instant

DECLARE @n INT = 1;
WHILE @n > 0
BEGIN
    UPDATE TOP (10000) dbo.Employees
    SET Email = LOWER(Name) + N'@corp.example'
    WHERE Email IS NULL;
    SET @n = @@ROWCOUNT;
    WAITFOR DELAY '00:00:00.2';             -- throttle
END

ALTER TABLE dbo.Employees ADD CONSTRAINT UQ_Employees_Email UNIQUE (Email);   -- builds an index

-- Online index
CREATE INDEX IX_Orders_Cust ON dbo.Orders (CustID) WITH (ONLINE = ON, MAXDOP = 4);

-- PostgreSQL: no long locks
SET lock_timeout = '3s';                                  -- fail fast instead of queueing
ALTER TABLE orders ADD COLUMN status text;                -- metadata-only
CREATE INDEX CONCURRENTLY ix_orders_status ON orders(status);
ALTER TABLE orders ADD CONSTRAINT chk_status CHECK (status IN ('N','P','S')) NOT VALID;
ALTER TABLE orders VALIDATE CONSTRAINT chk_status;        -- scans, but allows writes

-- Sequence as default (can be added/removed later, unlike identity)
CREATE SEQUENCE dbo.SeqOrderIDs START WITH 1;
ALTER TABLE dbo.T1 ADD CONSTRAINT DFT_T1_keycol DEFAULT (NEXT VALUE FOR dbo.SeqOrderIDs) FOR keycol;

-- CTAS: rebuild a table with new layout, then add what CTAS doesn't copy
SELECT * INTO dbo.Sales_new FROM dbo.Sales;      -- (SQL Server)  | CREATE TABLE sales_new AS SELECT * FROM sales; (PG)
CREATE CLUSTERED INDEX CX_Sales_new ON dbo.Sales_new(SaleDate);
```
Observe: during the backfill, `sys.dm_tran_locks` should show short row/page locks, not a table lock; the log should be backed up between batches in full recovery.

### 5. Under the hood   *(optional at L4)*
- **Why `ALTER` blocks everything**: it needs `Sch-M`; if a long-running query holds `Sch-S`, the `ALTER` waits, and *all new queries queue behind the waiting `ALTER`* (lock queue), producing an outage even though the `ALTER` itself is instant. Hence `lock_timeout` / retry loops.
- **Metadata-only column add**: the engine stores the default (or NULL) in the catalog and materializes values lazily when rows are read or rewritten; PostgreSQL 11+ keeps `attmissingval`.
- **Online index build** (SQL Server): builds from a snapshot (row versioning), records concurrent changes in a side structure, applies them, then takes a brief lock to switch; requires extra space and tempdb/log. Resumable builds exist in newer versions.
- **`CONCURRENTLY` (PostgreSQL)**: two table scans and waits for old transactions; can leave an `INVALID` index on failure — must drop and retry.
- **Trigger-based tools** (pt-osc) vs **binlog-based** (gh-ost): triggers add write overhead and locks on the original table; binlog readers avoid it and are throttle-able.
- **Delta Lake / lakehouse**: schema evolution (`mergeSchema`, `ALTER TABLE ADD COLUMNS`) is metadata in the transaction log; changing a type usually means rewriting files (`CTAS` / `REPLACE TABLE`).

### 6. Performance and design implications
- Time of the change ≈ table size / IO throughput for size-of-data operations; schedule off-peak and monitor.
- Batched backfills protect the log, replicas (lag) and lock escalation; throttle based on replication delay.
- Extra space: shadow copies and online index builds need up to 1–2× the object's size.
- Every added index or constraint adds write cost; do not leave NOT VALID/NOCHECK forever.
- Plan for rollback: with expand/contract each step is reversible; with a rename swap keep the old table until verified.
- Test on a production-size copy; measure duration and lock behaviour.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ALTER TABLE … ADD col` | Add column | Expand step |
| `ALTER COLUMN` | Change type/nullability (often size-of-data) | Contract step; test cost |
| `WITH (ONLINE = ON)` | Online index/ALTER (SQL Server editions) | Index builds/rebuilds |
| `CREATE INDEX CONCURRENTLY` | Non-blocking build (PG) | Production indexes |
| `NOT VALID` / `VALIDATE CONSTRAINT` | Defer validation (PG) | Large tables |
| `WITH NOCHECK` / `WITH CHECK CHECK` | Same idea (SQL Server) | Large tables |
| `lock_timeout`, `SET LOCK_TIMEOUT` | Fail instead of queueing | Every DDL on a hot table |
| `SELECT INTO` / `CTAS` | Create + load | Table rebuilds |
| `sp_rename` / `RENAME TO` | Swap names | Shadow-table cutover |
| pt-online-schema-change, gh-ost, `pg_repack` | External online tools | MySQL/PG rewrites |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Native online DDL | Edition lacks it (SQL Server Standard), or it still takes a final blocking lock |
| Expand/contract | Teams can't coordinate multiple releases |
| Shadow-table tool | Table has triggers, FKs, or very high write rate (tool lag) |
| Maintenance window + plain `ALTER` | Table small or window available — simpler is safer |
| `NOT VALID` constraints | You forget to validate and leave the constraint untrusted |

### 9. Common mistakes
- `ALTER COLUMN` type change on a huge table at peak → long `Sch-M`, blocked traffic → shadow table or window.
- No `lock_timeout` → lock queue pile-up → fail fast and retry.
- Backfill in one transaction → huge log, lock escalation → batches.
- Dropping the old column in the same release as the code change → rollback impossible → separate release.
- Assuming CTAS copies indexes/constraints → missing PK/indexes → recreate them explicitly.
- Forgetting replicas/CDC consumers that need the new schema first.

### 10. 🎯 Interview kit
**30-second answer:** "I classify the change: metadata-only (instant), concurrent/online (index builds, NOT VALID constraints) or size-of-data. For the last I do expand/contract with batched backfill or a shadow table with sync triggers and a quick rename swap. I always set a lock timeout, because the DDL queues behind long queries and blocks everyone."

**Structure a longer answer:** classify → lock behaviour → technique → backfill/validation → cutover and rollback → monitoring.

**Likely follow-ups:**
- *CTAS vs CREATE + INSERT SELECT?* → CTAS infers columns from the query and doesn't carry indexes, constraints, defaults (and may be minimally logged); `CREATE` + `INSERT` gives full control of the definition up front.
- *Add a NOT NULL column to a 1B-row table?* → add nullable, backfill in batches, add a validated check/`NOT NULL` last (or default with metadata-only support).
- *Identity vs sequence for flexibility?* → sequence + default constraint can be added/removed and range-allocated.
- *How do you add a foreign key online?* → `NOT VALID`/`NOCHECK` then validate, with indexes on the child column.

**Signals interviewers look for:** you reason about lock queues, mention backward compatibility across deploys, and know your engine's online capabilities.

### 11. Check yourself
- [ ] Classify five `ALTER` operations as metadata-only or size-of-data for your engine.
- [ ] Write the steps to rename a column with zero downtime.
- [ ] Explain why a fast `ALTER` can still cause an outage.
- [ ] Design a shadow-table migration including verification and rollback.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../../../TOOLS/MySQL/_index.md>)

## Practice questions

- [Basic] [How do you add a new column to an existing table?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22963_How_do_you_add_a_new_column_to_an_existing_table.md>)
- [Basic] [What is the difference between a UNIQUE constraint and a PRIMARY KEY defined in DDL?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-23012_What_is_the_difference_between_a_UNIQUE_constraint_and_a_PRI.md>)
- [Intermediate] [How do you add a NOT NULL constraint to an existing column using ALTER?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22969_How_do_you_add_a_NOT_NULL_constraint_to_an_existing_column_u.md>)
- [Intermediate] [How do you ensure new tables created by a migration automatically inherit the correct GRANTs without manual steps each time?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22913_How_do_you_ensure_new_tables_created_by_a_migration_automati.md>)
- [Deep] [You ran an ALTER TABLE ADD COLUMN on a 200M-row table in production and it locked writes for minutes. How would you make that change online with minimal disruption?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22849_You_ran_an_ALTER_TABLE_ADD_COLUMN_on_a_200M-row_table_in_pro.md>)
- [Deep] [Explain why an UPDATE that flips an indexed column can be far slower than one that doesn't, and how you'd measure the write amplification.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22879_Explain_why_an_UPDATE_that_flips_an_indexed_column_can_be_fa.md>)
- [Deep] [Explain how ALTER TABLE ... RENAME combined with a view or synonym can give you a blue-green table swap. What are the gotchas?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22902_Explain_how_ALTER_TABLE_RENAME_combined_with_a_view_or_synon.md>)
- [Deep] [How would you script a repeatable, forward-only migration for adding a column with a computed default backfill for 100M rows?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22919_How_would_you_script_a_repeatable_forward-only_migration_for.md>)

## Related concepts

- [Schema Evolution in Lakehouse Tables](<../SQL_Fundamentals/Schema_Evolution_in_Lakehouse_Tables.md>) *(SQL Fundamentals)*
- [Safe Migrations on Rolling Deploys](<../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Safe_Migrations_on_Rolling_Deploys.md>) *(PostgreSQL Platform)*
- [Online Schema Changes](<../../../TOOLS/MySQL/MySQL_Platform/Online_Schema_Changes.md>) *(MySQL Platform)*

## References

- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Data ��difi�ati�n
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Essential Delta Lake Operations › Append
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — sys.dmdbstatsproperties › Statistics DDL
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › CREATE
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › ALTER
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Mastering Relational Entities in Databricks › Creating tables in the new database
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Schema Handling › Adding Comments to a Column
- *Delta Lake: Up and Running* — Bennie Haelen, Dan Davis — Schema Handling › Renaming a Column

