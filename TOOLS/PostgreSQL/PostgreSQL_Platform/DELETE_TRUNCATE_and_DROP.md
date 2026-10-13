---
id: "TOOLS/PostgreSQL/PostgreSQL_Platform/DELETE_TRUNCATE_and_DROP"
level: 4
type: "concept"
topic: "TOOLS/PostgreSQL/PostgreSQL_Platform"
tools: []
---

# DELETE TRUNCATE and DROP

**L4 Advanced** · [↑ PostgreSQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** `DELETE` removes chosen rows (slowly, transactionally, leaving dead tuples), `TRUNCATE` empties a whole table almost instantly by swapping its storage, and `DROP` removes the table object itself — and all three are transactional in PostgreSQL.

### 1. The big picture
Removing data in PostgreSQL is never "just erase the bytes". The engine is built on MVCC (multi-version concurrency control): a `DELETE` marks row versions as dead, and a background process (`VACUUM`) reclaims the space later. `TRUNCATE` and `DROP` are DDL-style operations that work at the file level. Choosing the right one is a decision about *scope* (some rows, all rows, the object), *speed*, *locking*, *triggers*, and *whether you may need to roll back*.

Related topics: foreign keys, VACUUM and bloat, soft deletes, partitioning (where `DROP PARTITION` is the best retention tool).

### 2. Core concepts and vocabulary
- **MVCC** — every row version carries `xmin` (creating transaction) and `xmax` (deleting transaction); readers see a snapshot, not locks.
- **Dead tuple** — a row version no snapshot can see any more; it still occupies space until vacuumed.
- **VACUUM** — marks dead tuple space reusable; **`VACUUM FULL`** rewrites the table and returns space to the OS (takes `ACCESS EXCLUSIVE`).
- **Table bloat** — space held by dead tuples/free space inside the table file.
- **`ON DELETE CASCADE`** — foreign-key action that deletes child rows when the parent row goes.
- **Soft delete** — marking a row deleted (`deleted_at`) instead of removing it.
- **Relfilenode** — the on-disk file(s) backing a table; `TRUNCATE` assigns a new one.

### 3. How it works, step by step
**`DELETE FROM t WHERE …`**
1. Takes `ROW EXCLUSIVE` on the table (readers and other writers continue).
2. Finds matching rows, sets `xmax` on each version, fires row/statement triggers, writes WAL for each row.
3. On commit the rows become invisible; space is *not* returned until `VACUUM` (autovacuum) runs.

**`TRUNCATE t`**
1. Takes `ACCESS EXCLUSIVE` (blocks even `SELECT`).
2. Creates a fresh empty file for the table and indexes, and swaps it in; old file is deleted at commit.
3. Does **not** scan rows, does not fire `ON DELETE` row triggers (only `TRUNCATE` triggers), leaves no dead tuples; it is transactional and can be rolled back.
4. Refuses if other tables reference it by foreign key unless you add `CASCADE` (which truncates those too) or truncate them together.

**`DROP TABLE t`**
1. Takes `ACCESS EXCLUSIVE`, removes the catalog entries, indexes, triggers, and data files.
2. Fails if views/foreign keys depend on it unless `CASCADE` (drops the dependents' constraints/views, not the dependent tables).

### 4. Worked example
```sql
BEGIN;
DELETE FROM events WHERE created_at < now() - interval '90 days';
-- how many dead tuples did we create?
SELECT n_live_tup, n_dead_tup FROM pg_stat_user_tables WHERE relname = 'events';
ROLLBACK;                          -- rows are back; nothing was lost

BEGIN;
TRUNCATE events RESTART IDENTITY;  -- reset the serial/identity counters
ROLLBACK;                          -- also reversible, unlike in MySQL/Oracle

-- delete using another table
DELETE FROM order_items oi
USING orders o
WHERE oi.order_id = o.order_id AND o.status = 'cancelled'
RETURNING oi.order_id, oi.item_id;
```
`RETURNING` lets you capture what was deleted (useful for audit/archival in one statement). `DELETE … USING` is PostgreSQL's join-delete syntax.

### 5. Under the hood   *(optional at this level)*
- A `DELETE` writes `xmax` into the tuple header and WAL; indexes are *not* cleaned immediately — index entries point to dead heap tuples until vacuum removes both.
- Deleting 90% of a table leaves the file the same size. Plain `VACUUM` only makes space reusable inside the file (it can truncate empty pages at the very end). To shrink, use `VACUUM FULL`, `CLUSTER`, or `pg_repack` (online).
- `TRUNCATE` is MVCC-unsafe in a narrow case: a concurrent older snapshot can see the table as empty after commit, because the data files are replaced rather than versioned.
- Dropping a column (`ALTER TABLE … DROP COLUMN`) is only a catalog flag; storage is reclaimed lazily as rows are rewritten.

### 6. Performance and design implications
- Large deletes: do them in batches (`DELETE … WHERE id IN (SELECT id … LIMIT 10000)`) with commits in between, so locks, WAL bursts and autovacuum lag stay bounded.
- Retention by time: **partition** by date and `DROP`/`DETACH` old partitions — instant and no bloat.
- Monitor `n_dead_tup`, `last_autovacuum` in `pg_stat_user_tables`; tune per-table `autovacuum_vacuum_scale_factor` for very hot tables.
- `ON DELETE CASCADE` over millions of children is a hidden large delete; index the foreign-key column on the child.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `DELETE … WHERE … RETURNING *` | remove rows and see them | archival, auditing |
| `TRUNCATE t RESTART IDENTITY CASCADE` | empty table(s), reset sequences | test data, staging reloads |
| `DROP TABLE IF EXISTS t` | remove table | migrations |
| `VACUUM (VERBOSE, ANALYZE) t` | reclaim space/update stats | after big deletes |
| `pg_repack` / `VACUUM FULL` | physically shrink | heavily bloated tables |
| `ALTER TABLE … DETACH PARTITION … CONCURRENTLY` | online partition removal | retention jobs |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `DELETE` — subset of rows, triggers/cascades must run | deleting most of a huge table in one transaction |
| `TRUNCATE` — wipe all rows fast, reload | other sessions need to read concurrently; FK-referenced tables |
| `DROP` — object no longer exists | there are dependents; you only wanted the data gone |
| Soft delete — need history/undo | unique constraints (need a partial unique index `WHERE deleted_at IS NULL`) and every query must filter |

### 9. Common mistakes
- `DELETE FROM t;` without `WHERE` → whole table logically deleted, huge WAL → use `TRUNCATE` or wrap in a transaction and check the row count.
- Expecting disk space back after `DELETE` → file unchanged → `VACUUM FULL`/`pg_repack` or partition.
- `TRUNCATE … CASCADE` on a parent → silently empties child tables → list dependents first.
- Soft-deleted rows break uniqueness → duplicates or false conflicts → partial unique index.
- Running `VACUUM FULL` in business hours → exclusive lock stalls everything → use `pg_repack`.

### 10. 🎯 Interview kit
**30-second answer:** "`DELETE` is row-by-row DML: transactional, fires triggers, leaves dead tuples for `VACUUM`. `TRUNCATE` empties the table by replacing its files — fast, takes an `ACCESS EXCLUSIVE` lock, no row triggers. `DROP` removes the table itself. In PostgreSQL all three are transactional."
**Structure a longer answer:** scope → mechanism (MVCC, files) → locking/triggers/FKs → space reclamation → pick by scenario.
**Likely follow-ups:** *Can you roll back a `TRUNCATE`?* → yes in PG. *Why doesn't `DELETE` shrink the table?* → dead tuples/vacuum. *Best way to age out old data?* → partitions. *Soft vs hard delete?* → history and FK safety vs complexity and bloat.
**Signals interviewers look for:** mention of MVCC and vacuum, batching, partition drops, and the lock levels.

### 11. Check yourself
- [ ] Explain what physically happens to a row on `DELETE` and when its space is reusable.
- [ ] Say which statements take `ACCESS EXCLUSIVE` and what that blocks.
- [ ] Design a 90-day retention job for a 2 TB events table.
- [ ] Write the partial unique index that makes soft delete safe for an `email` column.
<!-- explainer:end -->

## Practice questions

- [Basic] [How do you comment out a line and a block of SQL in a PostgreSQL script and why does it matter in migration files?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36353_How_do_you_comment_out_a_line_and_a_block_of_SQL_in_a_Postgr.md>)
- [Basic] [What is the difference between deleting a row and updating a status column to mark it inactive?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36162_What_is_the_difference_between_deleting_a_row_and_updating_a.md>)
- [Basic] [What is a transaction? Explain BEGIN, COMMIT, and ROLLBACK.](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36107_What_is_a_transaction_Explain_BEGIN_COMMIT_and_ROLLBACK.md>)
- [Basic] [What is the difference between DELETE, TRUNCATE, and DROP?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36366_What_is_the_difference_between_DELETE_TRUNCATE_and_DROP.md>)
- [Intermediate] [You need to model soft-deletable rows but a foreign key from another table still points at them; how do you keep referential integrity working while hiding deleted rows from normal queries?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36298_You_need_to_model_soft-deletable_rows_but_a_foreign_key_from.md>)
- [Intermediate] [Your app stores enums as a Postgres ENUM type and now product wants to remove a value that's still referenced; explain why removing enum values is hard, and how you'd migrate to a lookup table to gain flexibility.](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36088_Your_app_stores_enums_as_a_Postgres_ENUM_type_and_now_produc.md>)
- [Deep] [What is autovacuum, how is it different from running VACUUM manually, and when might you tune its thresholds?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35898_What_is_autovacuum_how_is_it_different_from_running_VACUUM_m.md>)
- [Deep] [How do you restore a specific table from a full pg_dump, and what does the custom (-Fc) format let you do?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35915_How_do_you_restore_a_specific_table_from_a_full_pg_dump_and_.md>)

## Related concepts

- [Online Schema Changes](<../../MySQL/MySQL_Platform/Online_Schema_Changes.md>) *(MySQL Platform)*
- [DELETE vs TRUNCATE](<../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/DELETE_vs_TRUNCATE.md>) *(SQL Command Types)*
- [Schema Migration Risks](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Schema_Migration_Risks.md>) *(DBMS Concepts)*

## References

- *Data Pipelines Pocket Reference* — James Densmore — Data Ingestion Extracting Data › Full or Incremental Postgres Table Extraction

