---
id: "01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Upserts"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Command_Types"
tools: ["MySQL"]
---

# Upserts

**L4 Advanced** · [↑ SQL Command Types](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** An *upsert* means "update the row if it exists, insert it if it doesn't" in a single atomic statement — `INSERT ... ON CONFLICT`, `ON DUPLICATE KEY UPDATE`, or `MERGE` — so concurrent writers can't create duplicates or lose updates.

### 1. The big picture
Loading data is rarely "insert only". A nightly feed contains new customers *and* changed ones; a retrying job might resend rows already written. The naive pattern is: `SELECT` to check existence, then `INSERT` or `UPDATE`. That is two statements with a gap between them — two sessions can both see "not there" and both insert, producing a duplicate-key error or (without a unique constraint) duplicate rows.

Upsert collapses check-and-write into one statement the engine can make atomic. It is the basic building block of **idempotent** writes: running the same load twice yields the same final state. It is also the DML side of keys and constraints: upserts depend on a unique index/primary key to detect "already exists".

Related topics: *INSERT/UPDATE/DELETE basics* (below it), *Primary and unique keys* and *constraints* (it needs them), *Transactions and isolation* (it is where concurrency bugs live), and *slowly changing dimensions / CDC loading* (it is the main tool there).

### 2. Core concepts and vocabulary
- **Upsert** — update-or-insert; the name merges UPDATE and INSERT. The ANSI SQL form is `MERGE`.
- **Conflict target / match key** — the unique column set that decides "exists": a primary key, a unique constraint, or the `ON` condition of a `MERGE`.
- **Source and target** — in `MERGE`, the incoming rows (source) are compared against the destination table (target).
- **`WHEN MATCHED` / `WHEN NOT MATCHED [BY TARGET|BY SOURCE]`** — `MERGE` branches: row exists in both / only in source / only in target.
- **`RETURNING` / `OUTPUT`** — clause that gives back the written rows (e.g. generated keys, or which action fired).
- **Idempotent** — repeating the operation has no additional effect.
- **Write skew / lost update** — concurrency anomalies a naive read-then-write produces.
- **Join multiplicity** — when a source row matches several target rows (or vice versa), causing an operation to touch a row more than once.

### 3. How it works, step by step
**What the engine does when you upsert on a unique key (conceptually):**

1. Parse the statement and determine the conflict target (index to probe).
2. For each incoming row, probe the unique index.
3. **Found** → lock the existing row and apply the `UPDATE` branch (or do nothing for `DO NOTHING` / `INSERT IGNORE`).
4. **Not found** → insert the row and index entry; if a concurrent session inserted the same key in the meantime, the index probe sees the conflict and the engine falls back to the update path rather than erroring.
5. Commit; the log records the net change.

**Syntax by engine**

| Engine | Form |
|---|---|
| PostgreSQL, SQLite | `INSERT ... ON CONFLICT (key) DO UPDATE SET ... / DO NOTHING` |
| MySQL / MariaDB | `INSERT ... ON DUPLICATE KEY UPDATE ...`; `INSERT IGNORE`; `REPLACE INTO` |
| SQL Server, Oracle, DB2, Snowflake, Databricks/Delta | `MERGE INTO target USING source ON ... WHEN MATCHED ... WHEN NOT MATCHED ...` |
| Cassandra | `INSERT` *is* an upsert (last write wins) |

The *Ultimate T-SQL* book frames `MERGE` as the way to replace the traditional "separate `INSERT`, `UPDATE`, `DELETE` based on conditions" with one statement that "simplifies code, and can improve performance and maintainability" for data synchronization.

### 4. Worked example
Maintain a `customers` dimension from a staging feed.

```sql
-- PostgreSQL
INSERT INTO customers (customer_id, name, email, updated_at)
SELECT customer_id, name, email, now()
FROM   staging_customers
ON CONFLICT (customer_id)
DO UPDATE SET name       = EXCLUDED.name,
              email      = EXCLUDED.email,
              updated_at = EXCLUDED.updated_at
WHERE  customers.name  IS DISTINCT FROM EXCLUDED.name
   OR  customers.email IS DISTINCT FROM EXCLUDED.email;   -- skip no-op updates
```

`EXCLUDED` is the row that *would* have been inserted. The `WHERE` on the update avoids rewriting identical rows (less WAL/bloat, fewer triggers).

```sql
-- MySQL
INSERT INTO customers (customer_id, name, email)
VALUES (101, 'Ana', 'ana@x.com')
ON DUPLICATE KEY UPDATE name = VALUES(name), email = VALUES(email);
```

```sql
-- SQL Server
MERGE dbo.customers AS t
USING dbo.staging_customers AS s
   ON t.customer_id = s.customer_id
WHEN MATCHED AND (t.name <> s.name OR t.email <> s.email) THEN
     UPDATE SET t.name = s.name, t.email = s.email
WHEN NOT MATCHED BY TARGET THEN
     INSERT (customer_id, name, email) VALUES (s.customer_id, s.name, s.email)
OUTPUT $action, inserted.customer_id;
```

Reading the output: `OUTPUT $action` lists `INSERT` or `UPDATE` per row, so you can count how many rows were new vs changed.

**The "cannot update the same row twice" error** (`MERGE`, PostgreSQL 15+/SQL Server/Oracle: *"The MERGE statement attempted to UPDATE or DELETE the same row more than once"*) appears when the source has two rows for one key. Fix the source: deduplicate first.

```sql
WITH dedup AS (
  SELECT *, ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY updated_at DESC) rn
  FROM staging_customers
)
MERGE dbo.customers t USING (SELECT * FROM dedup WHERE rn = 1) s
  ON t.customer_id = s.customer_id
...
```

### 5. Under the hood
- **Why not "check then insert"?** Even inside a transaction, under the default `READ COMMITTED` level two sessions can both read "no row" and both attempt insert; one then fails on the unique index. Either catch and retry, or use a single-statement upsert that handles the race for you.
- **`MERGE` is not a magic atomic lock in every engine.** SQL Server's `MERGE` can still race on the insert branch unless you add `WITH (HOLDLOCK)` (serializable range lock) on the target; this plus its historical bugs is why many teams prefer explicit `UPDATE` then `INSERT ... WHERE NOT EXISTS` inside a transaction there. PostgreSQL's `ON CONFLICT` is purpose-built around the unique index and handles the race by design.
- **Locking.** The upsert takes a row lock on the existing row (update path) or an insertion lock on the index entry. Two sessions upserting overlapping key sets in different order can deadlock — sort the batch by key.
- **Identity/sequence gaps.** In PostgreSQL, a failed `ON CONFLICT` attempt still consumed a sequence value, and rollbacks never return numbers, so identities have gaps. That is expected: sequences are non-transactional for concurrency. Gaps only matter if you *must* have gap-free numbering (invoice numbers), where you use a counter-table pattern — T-SQL Fundamentals shows `UPDATE dbo.MySequences SET @nextval = val += 1 WHERE id='SEQ1'` to increment and read in one atomic access.
- **Generated columns** (e.g. Delta Lake `GENERATED ALWAYS`) can't be set by an upsert's insert/update branches — supplying a value raises an error.
- **Lakehouse tables.** `MERGE INTO` on Delta Lake rewrites whole data files that contain matched rows, so cost depends on how scattered matches are; partition or cluster on the match key.

### 6. Performance and design implications
- A **unique index on the conflict key** is mandatory (and is what makes it fast). Without it `ON CONFLICT` errors, and `MERGE` scans.
- Prefer **set-based** upserts from a staging table over row-by-row calls; thousands of single-row upserts mean thousands of round-trips and log flushes.
- Avoid **no-op updates**: they create new row versions (bloat, WAL, trigger firing, replication traffic).
- **Deduplicate source** deterministically (latest by timestamp) before merging.
- Bulk path: load into staging with `COPY`/bulk insert, then one upsert; faster than multi-row `INSERT` batches, which in turn beat single-row prepared statements.
- Use `RETURNING`/`OUTPUT` to get generated keys atomically — safer under concurrency than `SELECT LAST_INSERT_ID()` or `@@IDENTITY` (which can pick up another session's/trigger's value; use `SCOPE_IDENTITY()` or `OUTPUT`).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `INSERT ... ON CONFLICT DO UPDATE` | Atomic upsert in PostgreSQL | Default choice in PG |
| `ON CONFLICT DO NOTHING` | Insert if new, ignore otherwise | Idempotent event ingestion |
| `ON DUPLICATE KEY UPDATE` | MySQL upsert | MySQL tables with a unique key |
| `INSERT IGNORE` | Skips rows that violate *any* error (dupes and others) | Rarely; hides real problems |
| `REPLACE INTO` | Delete + insert | Almost never: resets other columns, fires delete triggers |
| `MERGE ... USING ... ON` | ANSI synchronization statement | SQL Server/Oracle/warehouses; SCD loads |
| `WITH (HOLDLOCK)` | Serializable lock hint on `MERGE` target (SQL Server) | When concurrent upserts are expected |
| `RETURNING` / `OUTPUT` | Returns affected rows | Capture keys, audit, count inserts vs updates |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `ON CONFLICT` / `ON DUPLICATE KEY` for a single unique key | Multiple unique indexes: MySQL picks whichever conflicts first, ambiguity |
| `MERGE` for set-based sync (insert + update + delete) | Highly concurrent OLTP writes on SQL Server without `HOLDLOCK` |
| `DO NOTHING` for append-only/idempotent loads | You actually need updated values |
| Delete+insert (or `REPLACE`) for simple full refreshes | Foreign keys, triggers, or audit tables — it's a different semantics |
| Staging + merge for batch pipelines | Tiny OLTP writes: just use the native single-row upsert |

### 9. Common mistakes
- Using `SELECT`-then-`INSERT` → race → duplicate-key errors or duplicates → single-statement upsert or retry loop.
- Omitting the unique constraint → upsert silently inserts duplicates (MySQL) or errors (PostgreSQL) → create the key first.
- Wrong conflict target (e.g. `ON CONFLICT (email)` when the key is `customer_id`) → rows skipped or raised instead of updated → match the real unique index.
- Source contains duplicate keys → "same row twice" error or nondeterministic winner → dedupe with `ROW_NUMBER()`.
- UPDATE with a join that fans out → three times the rows changed → ensure the join is 1:1, aggregate or dedupe the joined set.
- `UPDATE` without `WHERE` → whole table changed (Beginning T-SQL warns this is the most common restore-from-backup reason) → test with `SELECT` first and wrap in a transaction.
- `INSERT IGNORE` used to "make errors go away" → NULL-truncation and FK failures silently swallowed.

### 10. 🎯 Interview kit
**30-second answer:** "An upsert is insert-or-update in one atomic statement keyed on a unique constraint: `ON CONFLICT DO UPDATE` in PostgreSQL, `ON DUPLICATE KEY UPDATE` in MySQL, `MERGE` in SQL Server/Oracle. It avoids the check-then-write race and makes loads idempotent."

**Structure a longer answer:** clarify engine and key → core mechanism (probe unique index, lock, update or insert) → trade-offs (MERGE quirks, no-op updates, bulk vs row) → scenario (dimension load from staging with dedupe).

**Likely follow-ups:**
- *Why does MERGE say "cannot update the same row twice"?* → source has duplicate keys; deduplicate.
- *Conditionally insert only if not exists?* → `INSERT ... ON CONFLICT DO NOTHING` or `INSERT ... WHERE NOT EXISTS` with a unique constraint as the true guard.
- *Are `MERGE` and `ON CONFLICT` equivalent?* → no; different concurrency guarantees and features (multiple branches, deletes).
- *How would you design a concurrency-safe idempotent upsert without deadlocks?* → unique key, deterministic ordering of batch, short transactions, retry on serialization/deadlock.
- *Why identity gaps?* → sequence values aren't rolled back; only matters for gap-free requirements.
- *Fast bulk insert?* → `COPY`/bulk load > multi-row insert > single-row prepared.

**Signals interviewers look for:** mention of the race, unique key, idempotency, set-based loading, and engine-specific caveats (`HOLDLOCK`, `INSERT IGNORE`).

### 11. Check yourself
- [ ] Write an upsert in PostgreSQL, MySQL and SQL Server for the same table.
- [ ] Explain the race in "select then insert" and how upsert removes it.
- [ ] Fix a `MERGE` that fails because of duplicate source keys.
- [ ] Explain why you'd add a `WHERE` to the `DO UPDATE` branch.
- [ ] Say why `REPLACE INTO` differs from an upsert.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../../../TOOLS/MySQL/_index.md>)

## Practice questions

- [Basic] [How do you update multiple columns in a single UPDATE statement?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22976_How_do_you_update_multiple_columns_in_a_single_UPDATE_statem.md>)
- [Basic] [What does the INSERT statement do and what is its basic syntax?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-23036_What_does_the_INSERT_statement_do_and_what_is_its_basic_synt.md>)
- [Intermediate] [How do you conditionally insert a row only if it does not already exist?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-23033_How_do_you_conditionally_insert_a_row_only_if_it_does_not_al.md>)
- [Intermediate] [A colleague wrote an UPDATE with a join that changed 3x more rows than intended. How does the join multiplicity cause this and how do you prevent it?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22858_A_colleague_wrote_an_UPDATE_with_a_join_that_changed_3x_more.md>)
- [Intermediate] [Your bulk INSERT is slow. Compare multi-row INSERT, COPY/bulk load, and prepared statements, and explain when each wins.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22908_Your_bulk_INSERT_is_slow_Compare_multi-row_INSERT_COPY_bulk_.md>)
- [Deep] [What is the difference between UPDATE and MERGE for changing existing rows?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22999_What_is_the_difference_between_UPDATE_and_MERGE_for_changing.md>)
- [Deep] [Describe how you'd add a CHECK constraint to a live table when existing rows may violate it. What's the NOT VALID / VALIDATE strategy?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22874_Describe_how_you_d_add_a_CHECK_constraint_to_a_live_table_wh.md>)
- [Deep] [An UPDATE ... FROM in Postgres behaves differently from UPDATE ... JOIN in MySQL. Explain the semantic difference and a bug it can cause.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22924_An_UPDATE_FROM_in_Postgres_behaves_differently_from_UPDATE_J.md>)

## Related concepts

- [Lock Granularity](<../../Database_Internals/DBMS_Concepts/Lock_Granularity.md>) *(DBMS Concepts)*
- [Online Schema Changes](<../../../TOOLS/MySQL/MySQL_Platform/Online_Schema_Changes.md>) *(MySQL Platform)*
- [Unique Constraints for Dedup](<../SQL_Fundamentals/Unique_Constraints_for_Dedup.md>) *(SQL Fundamentals)*

## References

- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Data ��difi�ati�n
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — SQL Graph
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Manipulating Data › Inserting New Rows
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Manipulating Data › Adding One Row with Literal Values
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Manipulating Data › Avoiding Common Insert Errors
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Manipulating Data › Inserting Rows from Another Table
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › CREATE (INSERT)
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › UPDATE
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › MERGE Statement for UPSERT Operations
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › MERGE Statement Syntax
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Essential Delta Lake Operations › CREATE TABLE AS SELECT
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Essential Delta Lake Operations › Python
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Diving into the Delta Lake Ecosystem › Using INSERT
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Diving into the Delta Lake Ecosystem › Updating rows
- *Mapping Data Flows in Azure Data Factory* — Mark Kromer — Slowly Changing Dimensions › Prepare Data for Writing to Database
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Modeling Streaming Data › Insert only
- *SQL Server Tutorial* (YouTube series) — Part 69: Part 69   Merge in SQL Server ([watch](https://www.youtube.com/watch?v=5dk33HN8BX8))

