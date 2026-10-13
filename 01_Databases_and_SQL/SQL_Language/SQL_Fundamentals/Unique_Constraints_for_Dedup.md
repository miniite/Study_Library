---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Unique_Constraints_for_Dedup"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: []
---

# Unique Constraints for Dedup

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A unique constraint makes the database itself refuse duplicates, which is the only dedup guarantee that survives concurrent writers, retries and bugs; query-time dedup (`ROW_NUMBER`, `DISTINCT`) cleans what already got in.

### 1. The big picture
Duplicates enter data through retries (a client re-sends a payment), at-least-once pipelines (a stream redelivers events), double clicks, overlapping loads, and bad imports. Application-level checks ("select, and insert if not found") have a race: two sessions both see "not found" and both insert (a *time-of-check to time-of-use* bug). Only a constraint enforced inside the engine, using an index and locking, closes that race.

Think in two layers:
1. **Prevent** with constraints and idempotent writes (`UNIQUE`, `PRIMARY KEY`, partial/filtered unique indexes, `INSERT … ON CONFLICT`, `MERGE`).
2. **Detect and remove** with set-based queries (`GROUP BY … HAVING COUNT(*) > 1`, `ROW_NUMBER() OVER (PARTITION BY …)`).

### 2. Core concepts and vocabulary
- **Natural key** — business identifier (email, SKU + warehouse, (host, metric, ts)). **Surrogate key** — generated id.
- **Unique constraint** — declarative rule that no two rows share the same values in the column set.
- **Unique index** — the physical structure that enforces it. In most engines a constraint is *implemented by* a unique index.
- **Primary key** — unique + not null; at most one per table.
- **Partial (filtered) unique index** — uniqueness only over rows matching a predicate.
- **Idempotency key** — client-supplied token identifying one logical operation, stored under a unique constraint.
- **Upsert** — insert, or update if the key exists (`ON CONFLICT DO UPDATE`, `MERGE`).
- **Deferrable constraint** — checked at commit instead of per statement.
- **Exclusion constraint** — generalizes uniqueness to "no overlap" (PostgreSQL).

### 3. How it works, step by step
**What happens on insert.**
1. The engine locates the key in the unique index (B-tree).
2. If an equal key exists and is visible/committed → error (SQLSTATE 23505 in PostgreSQL; error 2627/2601 in SQL Server).
3. If an equal key exists from an *uncommitted* transaction → the second inserter **waits** for the first to commit or roll back, then re-evaluates. That wait is what serializes concurrent duplicates correctly.
4. Otherwise the key entry is inserted atomically with the row.

**NULLs.** SQL says `NULL` is not equal to `NULL`, so by the standard a unique constraint permits many NULLs. Engines differ:
- PostgreSQL/Oracle/MySQL/SQLite: multiple NULLs allowed (PostgreSQL 15 adds `NULLS NOT DISTINCT`).
- SQL Server: a unique constraint/index treats NULLs as equal → only one NULL; use a filtered index `WHERE col IS NOT NULL` to allow many.

**Conditional uniqueness (partial index).** "At most one active subscription per user": the "inactive" state is encoded as an end date, so uniqueness should apply only to open rows.

### 4. Worked example
```sql
-- 1) Basic: one account per email, named so errors are readable
ALTER TABLE users
  ADD CONSTRAINT uq_users_email UNIQUE (email);

-- 2) At most one active subscription per user (PostgreSQL partial unique index)
CREATE UNIQUE INDEX uq_one_active_sub
  ON subscriptions (user_id)
  WHERE end_date IS NULL;

-- SQL Server equivalent: filtered unique index
-- CREATE UNIQUE INDEX uq_one_active_sub ON dbo.subscriptions(user_id) WHERE end_date IS NULL;

-- 3) Idempotent ingestion of events that may arrive twice
CREATE TABLE events (
  event_id   uuid PRIMARY KEY,           -- idempotency key from the producer
  payload    jsonb NOT NULL,
  received_at timestamptz NOT NULL DEFAULT now()
);

INSERT INTO events (event_id, payload)
VALUES ($1, $2)
ON CONFLICT (event_id) DO NOTHING;       -- duplicate delivery is a no-op
-- affected rows = 1 → new; 0 → duplicate (handler can skip downstream work)

-- 4) Upsert for retried payment posts
INSERT INTO payments (idempotency_key, account_id, amount)
VALUES ($1, $2, $3)
ON CONFLICT (idempotency_key) DO UPDATE
   SET amount = EXCLUDED.amount          -- or DO NOTHING if immutable
RETURNING payment_id, (xmax = 0) AS inserted;
```

Pitfall in a batch upsert: if the *source rows themselves* contain the same key twice, PostgreSQL raises "ON CONFLICT DO UPDATE command cannot affect row a second time". Deduplicate the batch first:

```sql
INSERT INTO target (k, v, updated_at)
SELECT DISTINCT ON (k) k, v, updated_at
FROM   staging
ORDER  BY k, updated_at DESC            -- keep the latest per key, deterministic
ON CONFLICT (k) DO UPDATE SET v = EXCLUDED.v, updated_at = EXCLUDED.updated_at;
```

Detect and remove existing duplicates:

```sql
-- Find
SELECT user_id, amount, merchant, date_trunc('minute', ts) AS minute, COUNT(*) AS n
FROM   transactions
GROUP  BY 1,2,3,4
HAVING COUNT(*) > 1;

-- Keep the earliest, delete the rest (works in PostgreSQL and SQL Server with small syntax tweaks)
WITH ranked AS (
  SELECT ctid,
         ROW_NUMBER() OVER (PARTITION BY user_id, amount, merchant, date_trunc('minute', ts)
                            ORDER BY ts, txn_id) AS rn
  FROM transactions
)
DELETE FROM transactions t USING ranked r
WHERE  t.ctid = r.ctid AND r.rn > 1;
```

Deferrable uniqueness (swap two rows' positions without a transient violation):

```sql
ALTER TABLE playlist_items
  ADD CONSTRAINT uq_pos UNIQUE (playlist_id, position) DEFERRABLE INITIALLY DEFERRED;
BEGIN;
UPDATE playlist_items SET position = 2 WHERE item_id = 10;  -- temporarily duplicates 2
UPDATE playlist_items SET position = 1 WHERE item_id = 11;
COMMIT;                                                      -- checked here, passes
```

### 5. Under the hood
- **Index-backed.** Each unique constraint is a B-tree with a uniqueness check. Insert cost = one more index probe and maintenance. The check uses the index's key order, so a lookup is O(log n).
- **Concurrency.** PostgreSQL uses *speculative insertion* for `ON CONFLICT`: it inserts the heap tuple optimistically, tries the index, and on conflict aborts the speculative tuple and takes the conflict path with a row lock, never a table lock. SQL Server's `MERGE` is historically bug-prone under concurrency; hint it with `HOLDLOCK` (serializable range lock) or use an explicit `UPDATE … ; IF @@ROWCOUNT = 0 INSERT …` inside a transaction with `UPDLOCK, HOLDLOCK`.
- **Unique constraint vs unique index.** Functionally equal in uniqueness enforcement; the constraint is metadata (can be referenced by foreign keys, appears in `information_schema.table_constraints`), while the index can additionally be partial, include columns (`INCLUDE`), or be an expression index (`UNIQUE (lower(email))`).
- **Hash/exclusion.** `EXCLUDE USING gist (room WITH =, during WITH &&)` prevents overlapping bookings, something no ordinary unique constraint can express.
- **Distributed warehouses.** Snowflake, BigQuery and Redshift accept `UNIQUE` as *informational* only (not enforced). There, dedup must be done in the load (`MERGE`, `QUALIFY ROW_NUMBER() = 1`) and verified with data tests.

### 6. Performance and design implications
- Each unique index adds write cost and storage; create them for real business rules, not "just in case".
- Wide keys make indexes big. Use a surrogate PK for foreign keys and a separate unique constraint on the natural key.
- Time-limited idempotency tables (24–72 h) can be pruned; partition by day to drop old partitions cheaply.
- Unique constraints on partitioned tables must include the partition key in most engines; plan keys accordingly.
- Batch dedup before upsert to avoid conflict errors and to make results deterministic.
- Use `lower(email)` or a case-insensitive collation/citext so `A@x.com` and `a@x.com` collide.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `UNIQUE (cols)` / `CREATE UNIQUE INDEX` | Enforce uniqueness | Business keys |
| `… WHERE predicate` (partial/filtered) | Conditional uniqueness | "One active per user" |
| `INSERT … ON CONFLICT DO NOTHING/UPDATE` | Idempotent insert/upsert (PostgreSQL, SQLite) | Retries, streams |
| `INSERT IGNORE`, `ON DUPLICATE KEY UPDATE` | MySQL equivalents | MySQL ingestion |
| `MERGE` | Standard upsert (use locking hints on SQL Server) | Batch loads |
| `IGNORE_DUP_KEY = ON` (SQL Server) | Silently discard duplicates in inserts | Simple dedup loads |
| `NULLS NOT DISTINCT` (PG 15+) | Treat NULLs as equal | Single-NULL semantics |
| `DEFERRABLE INITIALLY DEFERRED` | Check at commit | Reordering rows |
| `EXCLUDE USING gist` | No-overlap constraint | Bookings, schedules |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Constraint-based dedup — correct under concurrency | Warehouse engine doesn't enforce uniqueness |
| `ON CONFLICT DO NOTHING` — immutable events | Later versions of the same key should win (use DO UPDATE with version/timestamp guard) |
| Partial unique index for state-dependent rules | Trigger-based checks (race-prone, slower) |
| Query-time `ROW_NUMBER` dedup in analytics | You need to stop duplicates at write time |
| Idempotency table keyed by request id | Keys never expire and table grows unbounded |

### 9. Common mistakes
- "Check then insert" in application code → duplicates under concurrency → unique constraint + handle the error.
- Plain `UNIQUE (user_id, end_date)` for "one active row" → NULL end dates don't collide (PostgreSQL) → partial index on `end_date IS NULL`.
- One NULL allowed in SQL Server, many in PostgreSQL → surprise on migration → know the engine rule.
- Upserting a batch with internal duplicates → "cannot affect row a second time" → dedup the source first.
- Dedup by `SELECT DISTINCT *` → keeps rows that differ in a timestamp → define the business key and keep one per key.
- Assuming warehouse `UNIQUE` is enforced → duplicates silently load → add dedup + test.
- Unnamed constraints → cryptic names in logs → `CONSTRAINT uq_<table>_<cols>`.

### 10. 🎯 Interview kit
**30-second answer:** Enforce uniqueness in the database with a unique constraint or partial unique index, and make writers idempotent with an idempotency key and `ON CONFLICT DO NOTHING`/`MERGE`. Application "check then insert" races; the engine's index check plus locking doesn't. For existing duplicates, use `ROW_NUMBER() OVER (PARTITION BY business_key ORDER BY …)` and delete `rn > 1`.

**Structure a longer answer:** where duplicates come from → constraint design (key, NULLs, partial) → idempotent write path → concurrency behaviour → cleanup of historical duplicates → warehouse caveat.

**Likely follow-ups:**
- *Constraint vs index?* → same enforcement; index can be partial/expression.
- *Can NULLs violate uniqueness?* → by the standard no; SQL Server treats them as equal.
- *Why would a retried POST create two rows even with a check?* → race between check and insert.
- *How do you prevent overlapping reservations?* → exclusion constraint or serializable + range lock.

**Signals interviewers look for:** concurrency reasoning, NULL semantics, partial indexes, idempotency design, retention of keys.

### 11. Check yourself
- [ ] Write a partial unique index for "one primary address per user".
- [ ] Explain why two concurrent inserts of the same key end with one error, not two rows.
- [ ] Deduplicate a staging batch before an upsert, keeping the latest by timestamp.
- [ ] State how SQL Server and PostgreSQL differ on NULLs in unique indexes.
- [ ] Design an idempotency table with a retention policy.
<!-- explainer:end -->

## Practice questions

- [Basic] [How do you make a column store only unique email addresses when creating the table?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39805_How_do_you_make_a_column_store_only_unique_email_addresses_w.md>)
- [Intermediate] [You must guarantee at most one active subscription per user where inactive is a NULL end_date. Why does a plain UNIQUE constraint fail and how does a partial unique index solve it?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39883_You_must_guarantee_at_most_one_active_subscription_per_user_.md>)
- [Intermediate] [You need to enforce that at most one row per user has is_primary = true. Show the partial unique index approach and explain why a plain unique constraint or trigger is worse.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40325_You_need_to_enforce_that_at_most_one_row_per_user_has_is_pri.md>)
- [Deep] [Explain a DEFERRABLE INITIALLY DEFERRED constraint and give a real case, like swapping two rows' unique sort positions, where deferring the check until COMMIT is the only clean solution.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39903_Explain_a_DEFERRABLE_INITIALLY_DEFERRED_constraint_and_give_.md>)
- [Deep] [How does a UNIQUE constraint differ from a UNIQUE index, and can NULLs violate uniqueness?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40227_How_does_a_UNIQUE_constraint_differ_from_a_UNIQUE_index_and_.md>)
- [Deep] [You need to detect duplicate transactions that differ only by a retry-generated id but share (user, amount, merchant, minute). How would you write the dedup query and what unique constraint would prevent them at write time?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40471_You_need_to_detect_duplicate_transactions_that_differ_only_b.md>)
- [Deep] [Your agent writes structured results back to a business table. How would you make the write idempotent so a retried LLM tool-call doesn't create duplicate records?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40896_Your_agent_writes_structured_results_back_to_a_business_tabl.md>)
- [Expert] [How do you enforce a constraint that a value must be unique only when another column is true (partial/filtered index)?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-41108_How_do_you_enforce_a_constraint_that_a_value_must_be_unique_.md>)

## Related concepts

- [Unique and Deferred Constraints](<../../Database_Design/Constraints_and_Keys/Unique_and_Deferred_Constraints.md>) *(Constraints and Keys)*
- [Deadlocks in MySQL](<../../../TOOLS/MySQL/MySQL_Platform/Deadlocks_in_MySQL.md>) *(MySQL Platform)*
- [Adding Constraints Online](<../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Adding_Constraints_Online.md>) *(PostgreSQL Platform)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 10: Transactions, Concurrency, and Isolation Levels › Consistency
- *Designing Data-Intensive Applications* — Martin Kleppmann — Enforcing Constraints
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Data ��difi�ati�n
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — SQL Graph (cont.)
- *Grokking Relational Database Design* — Qiang Hao, Michail Tsikerdekis — Implementation
- *Grokking Relational Database Design* — Qiang Hao, Michail Tsikerdekis — Implementation (cont.)

