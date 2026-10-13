---
id: "TOOLS/PostgreSQL/PostgreSQL_Platform/RETURNING_Clause"
level: 4
type: "concept"
topic: "TOOLS/PostgreSQL/PostgreSQL_Platform"
tools: []
---

# RETURNING Clause

**L4 Advanced** · [↑ PostgreSQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** `RETURNING` makes `INSERT`, `UPDATE`, `DELETE` (and `MERGE` in PG 17+) hand back the rows they actually wrote — including server-generated values — in the same round trip and the same atomic statement, so you never need a second `SELECT` that could race with other sessions.

### 1. The big picture
A data-changing statement normally returns only a command tag such as `UPDATE 3`. That tells you *how many* rows changed but not *which* rows or *what values they ended up with*. Many values are decided by the server, not the client: `serial`/identity keys, `DEFAULT now()`, `gen_random_uuid()`, values rewritten by `BEFORE` triggers, generated columns. The classic workaround — insert, then `SELECT … WHERE` to read it back — costs an extra round trip and is racy, because another session may have modified the row between your two statements.

`RETURNING` closes that gap. It is a PostgreSQL extension to SQL (SQL Server's equivalent is the `OUTPUT` clause; MySQL has no real equivalent before MariaDB's). It sits at the end of a DML statement and behaves like a `SELECT` list evaluated over each affected row. It connects to several neighbouring topics: upsert (`ON CONFLICT`), queue-style workers (`DELETE … RETURNING` with `SKIP LOCKED`), data-modifying CTEs, and audit/archiving patterns.

### 2. Core concepts and vocabulary
- **Command tag** — the short status string (`INSERT 0 1`, `UPDATE 5`, `DELETE 2`) returned when no `RETURNING` is present.
- **`RETURNING` list** — comma-separated expressions or `*`, evaluated per affected row; can use aliases (`RETURNING id AS new_id`).
- **New vs. old row image** — for `INSERT` and `UPDATE`, `RETURNING` sees the *new* row as stored; for `DELETE`, the row as it was before deletion. (PostgreSQL 18 adds `old.` and `new.` qualifiers so `UPDATE … RETURNING old.price, new.price` is possible; before that you need a join or CTE trick.)
- **Data-modifying CTE** — a `WITH` clause containing `INSERT/UPDATE/DELETE … RETURNING`, whose output feeds the main statement.
- **`EXCLUDED` pseudo-table** — in `INSERT … ON CONFLICT DO UPDATE`, holds the row proposed for insertion; usable in `SET` and `WHERE`, but `RETURNING` reports the final stored row.
- **Identity / serial column** — an auto-numbered column whose value only exists after insert; the primary reason people reach for `RETURNING`.

### 3. How it works, step by step
1. The statement runs normally: the executor finds or builds each target row version, fires `BEFORE` row triggers, applies defaults and generated columns, writes the tuple.
2. For each row that was actually written (or deleted), the executor evaluates the `RETURNING` expressions against the stored row.
3. Those rows are streamed back to the client as a normal result set, with column names taken from the list or aliases.
4. `AFTER` triggers run at statement end; they do **not** change what `RETURNING` already produced. Only `BEFORE` triggers influence the returned values.
5. Because everything is one statement, it is atomic: if the transaction rolls back, the rows you were shown never existed. The result is also one snapshot — no race window.

Key behaviours to remember:

| Statement | What `RETURNING` shows | Rows missing from the output |
|---|---|---|
| `INSERT` | the new row(s) as stored (defaults applied) | rows skipped by `ON CONFLICT DO NOTHING` |
| `UPDATE` | the new version of each updated row | rows not matched by `WHERE` |
| `DELETE` | the deleted row's last contents | rows not matched |
| `MERGE` (PG 17+) | the affected rows; `merge_action()` tells which branch ran | rows hit by `DO NOTHING` |

### 4. Worked example
```sql
CREATE TABLE orders (
  id          bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  customer_id int  NOT NULL,
  status      text NOT NULL DEFAULT 'new',
  created_at  timestamptz NOT NULL DEFAULT now()
);

-- 1. Get the generated key and default back without a second query
INSERT INTO orders (customer_id) VALUES (42)
RETURNING id, status, created_at;
--  id | status |          created_at
-- ----+--------+-------------------------------
--   1 | new    | 2026-10-13 09:14:02.118+00

-- 2. Multi-row insert returns one row per input row, in input order
INSERT INTO orders (customer_id) VALUES (7), (8), (9) RETURNING id;

-- 3. Confirm exactly what an UPDATE touched
UPDATE orders SET status = 'paid'
WHERE customer_id = 42 AND status = 'new'
RETURNING id, status;

-- 4. Archive-and-delete in one atomic step with a CTE
WITH moved AS (
  DELETE FROM orders WHERE created_at < now() - interval '1 year'
  RETURNING *
)
INSERT INTO orders_archive SELECT * FROM moved;

-- 5. Upsert and learn whether it inserted or updated
INSERT INTO counters (k, n) VALUES ('hits', 1)
ON CONFLICT (k) DO UPDATE SET n = counters.n + EXCLUDED.n
RETURNING k, n, (xmax = 0) AS inserted;
```
Reading the results: in example 3, if the result set is empty, nothing matched — the application can treat "0 rows returned" as "someone else already changed it" (a cheap optimistic-lock check). In example 4, the delete and the insert are one statement, so a crash can never leave rows deleted but not archived. In example 5, `xmax = 0` is a well-known trick: a freshly inserted tuple has `xmax` of 0, while an updated one carries the updater's transaction id. It is an implementation detail — fine for diagnostics, not a contract.

### 5. Under the hood
- `RETURNING` is implemented in the `ModifyTable` plan node: after the node writes a tuple, it projects the `RETURNING` target list from the resulting slot and emits it upward. Hence a plan for `UPDATE … RETURNING` is simply the normal update plan with an extra projection.
- In a data-modifying CTE, the sub-statement runs **exactly once, to completion**, regardless of whether the outer query reads all of its output. All sub-statements in one `WITH` see the *same snapshot*, so they cannot see each other's changes — only the `RETURNING` output is the communication channel. Two CTEs updating the same row give unpredictable results.
- Views: `RETURNING` works through updatable views and, with `INSTEAD OF` triggers, returns whatever the trigger returns. If a `BEFORE` trigger returns `NULL`, the row is skipped and nothing is returned for it.
- Foreign keys cascading deletes do not appear in the top-level `RETURNING`; only rows from the named target table are reported.

### 6. Performance and design implications
- It removes one network round trip per write — significant over WAN or via a pooler, and lets drivers (`psycopg`, JDBC `getGeneratedKeys`, Npgsql) use a single message exchange.
- Cost on the server is negligible: no extra scan, just projection. The bigger cost is returning wide rows (`RETURNING *` on a table with large `jsonb`/`text` columns) — return only the columns you need.
- Under concurrency, `RETURNING` is what makes patterns like `UPDATE … WHERE id = (SELECT id … FOR UPDATE SKIP LOCKED LIMIT 1) RETURNING *` a correct, lock-free job queue claim.
- For bulk loads via `COPY` there is no `RETURNING`; use `INSERT … SELECT` or load a staging table first when you need generated keys.
- Measure with `EXPLAIN (ANALYZE)` — you will see the same `ModifyTable` node and a "rows" count equal to the number returned.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `INSERT … RETURNING id` | Return generated key(s) | Child inserts that need the parent id |
| `UPDATE … RETURNING *` | Show post-update row | Optimistic locking, audit responses |
| `DELETE … RETURNING *` | Show what was removed | Queues, archiving, safe confirmation |
| `WITH x AS (… RETURNING …) …` | Chain statements atomically | Parent+children insert in one round trip |
| `ON CONFLICT … DO UPDATE … RETURNING` | Upsert and read result | Idempotent writes |
| `MERGE … RETURNING merge_action()` (PG 17+) | Report which branch fired | Sync/ETL steps |
| `currval('seq')` / `lastval()` | Old way to read a sequence | Avoid when `RETURNING` is available |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| You need server-generated values (ids, timestamps) immediately | You only need the row count — the command tag is enough |
| You want "claim and read" semantics for queues | You return `*` from wide tables on a hot path |
| You want atomic move-between-tables via CTE | You need to see changes made by a sibling CTE (they share one snapshot) |
| Your code must be portable across DBs (use `OUTPUT`/`getGeneratedKeys` abstractions) | You rely on `ON CONFLICT DO NOTHING` returning the existing row — it returns nothing for skipped rows |

### 9. Common mistakes
- Using `SELECT lastval()` or `SELECT max(id)` after insert → wrong under concurrency or after another insert in the same session → use `RETURNING id`.
- Expecting `ON CONFLICT DO NOTHING … RETURNING id` to return the existing row → empty result → use `DO UPDATE SET col = col` (a no-op update) or a follow-up `SELECT`.
- Running `UPDATE` without `WHERE` and not noticing → every row changed → use `RETURNING` or a transaction (`BEGIN … check … COMMIT/ROLLBACK`) in manual sessions.
- Assuming an `AFTER` trigger's changes show up in `RETURNING` → they do not → read back with a separate query if you need them.
- Putting two data-modifying CTEs on the same row → unpredictable winner → restructure into one statement.

### 10. 🎯 Interview kit
**30-second answer:** `RETURNING` lets an `INSERT`, `UPDATE` or `DELETE` return the affected rows, including defaults and generated keys, in the same statement. It saves a round trip, avoids races from a follow-up `SELECT`, and is the basis for atomic patterns like archive-and-delete CTEs and queue claiming.

**Structure a longer answer:** clarify what is being returned (new row for insert/update, old for delete) → explain that it is projected inside the same statement and snapshot → show a CTE or upsert example → mention limits (skipped rows, `AFTER` triggers, shared CTE snapshot) → close with a job-queue scenario.

**Likely follow-ups:**
- *How do you get the id of a row you just inserted?* → `INSERT … RETURNING id`, not `currval`/`max`.
- *What does `ON CONFLICT DO UPDATE` return?* → the final stored row, for both inserted and updated rows; `DO NOTHING` returns nothing for conflicts.
- *How would you move rows between tables atomically?* → `WITH d AS (DELETE … RETURNING *) INSERT … SELECT * FROM d`.
- *Can `RETURNING` show the old value on update?* → PG 18 supports `old.col`; earlier, join a CTE that read the row `FOR UPDATE` first.
- *How does this compare to SQL Server?* → `OUTPUT inserted.* / deleted.*`, which can also write into a table variable.

**Signals interviewers look for:** you think in terms of atomic single statements and race-free patterns rather than "insert then select"; you know the empty-result cases; you can combine it with `SKIP LOCKED` and CTEs.

### 11. Check yourself
- [ ] Write an insert that returns the generated identity and a default timestamp.
- [ ] Explain why a follow-up `SELECT` after `INSERT` can return the wrong row under concurrency.
- [ ] Write an archive-and-delete in one statement and explain why it is crash-safe.
- [ ] Say what `RETURNING` outputs for rows skipped by `ON CONFLICT DO NOTHING`.
- [ ] Explain why two CTEs updating the same row is a problem (shared snapshot).
- [ ] Write a worker query that claims one job with `FOR UPDATE SKIP LOCKED … RETURNING`.
<!-- explainer:end -->

## Practice questions

- [Basic] [What does the RETURNING * clause give you back after a DELETE and why is it handy for confirming what was removed?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36352_What_does_the_RETURNING_clause_give_you_back_after_a_DELETE_.md>)
- [Basic] [How do you insert multiple rows into a table with a single INSERT statement?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36142_How_do_you_insert_multiple_rows_into_a_table_with_a_single_I.md>)
- [Intermediate] [How would you write an INSERT that copies rows from one table into another using INSERT INTO ... SELECT?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36195_How_would_you_write_an_INSERT_that_copies_rows_from_one_tabl.md>)
- [Intermediate] [How would you write a query to swap the values of two columns in every row?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36209_How_would_you_write_a_query_to_swap_the_values_of_two_column.md>)
- [Intermediate] [Your BEFORE trigger silently modified a row and downstream code broke; explain the difference between BEFORE and AFTER triggers, when returning NULL cancels the row, and why BEFORE triggers surprise people.](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36339_Your_BEFORE_trigger_silently_modified_a_row_and_downstream_c.md>)
- [Intermediate] [What does INSERT ... ON CONFLICT DO NOTHING do, and how is it different from DO UPDATE?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35859_What_does_INSERT_ON_CONFLICT_DO_NOTHING_do_and_how_is_it_dif.md>)
- [Intermediate] [What is a trigger and give a scenario where you would use one.](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36357_What_is_a_trigger_and_give_a_scenario_where_you_would_use_on.md>)
- [Intermediate] [What is SQL injection and how do parameterized queries prevent it in PostgreSQL?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36395_What_is_SQL_injection_and_how_do_parameterized_queries_preve.md>)

## Related concepts

- [Upserts](<../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Upserts.md>) *(SQL Command Types)*
- [Scalar Subqueries](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Scalar_Subqueries.md>) *(SQL Fundamentals)*
- [Dynamic SQL](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Dynamic_SQL.md>) *(DBMS Concepts)*

