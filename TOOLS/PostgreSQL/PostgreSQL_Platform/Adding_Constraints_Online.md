---
id: "TOOLS/PostgreSQL/PostgreSQL_Platform/Adding_Constraints_Online"
level: 4
type: "concept"
topic: "TOOLS/PostgreSQL/PostgreSQL_Platform"
tools: []
---

# Adding Constraints Online

**L4 Advanced** · [↑ PostgreSQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** On a big PostgreSQL table you add a `CHECK` or foreign key in two steps — `ADD CONSTRAINT … NOT VALID` (cheap lock, enforced for new writes) then `VALIDATE CONSTRAINT` (long scan under a weaker lock) — and you add unique/partial indexes with `CREATE UNIQUE INDEX CONCURRENTLY`.

### 1. The big picture
Constraints push data rules into the database, where no application bug or retry storm can bypass them: valid ranges (`CHECK`), referential integrity (foreign keys), uniqueness (including *partial* uniqueness), non-overlap (`EXCLUDE`). Defining them at table creation is trivial. Adding them later to a table with hundreds of millions of rows is a production risk: the naive `ALTER TABLE … ADD CONSTRAINT` takes an `ACCESS EXCLUSIVE` or `SHARE ROW EXCLUSIVE` lock **and** scans the whole table to validate, blocking reads and writes (and queueing everything behind it) for minutes.

PostgreSQL's answer is to split *enforcement of new data* from *validation of existing data*. This lesson covers the constraint kinds, the two-step online pattern with the exact locks, partial unique indexes and idempotency keys, deferrable constraints, exclusion constraints, and how to deal with existing violations.

### 2. Core concepts and vocabulary
- **`CHECK` constraint** — a boolean expression every row must satisfy (`amount > 0`); passes when the expression is `TRUE` **or `NULL`** (unknown).
- **Column vs. table constraint** — column constraint is written next to one column; a table constraint can involve several columns. Functionally the same; only multi-column rules require the table form.
- **Foreign key** — child columns must match a parent key (or be `NULL` with the default `MATCH SIMPLE`); actions `ON DELETE/UPDATE RESTRICT | NO ACTION | CASCADE | SET NULL | SET DEFAULT`.
- **Referential integrity** — no orphan child rows, guaranteed by foreign keys.
- **`NOT VALID`** — constraint is recorded and enforced for subsequent `INSERT`/`UPDATE`, but existing rows are not checked.
- **`VALIDATE CONSTRAINT`** — scans existing rows and marks the constraint valid.
- **Unique constraint / unique index** — at most one row per key; a unique constraint is implemented *by* a unique index.
- **Partial index** — index with a `WHERE` predicate; a *partial unique index* enforces uniqueness only on the subset.
- **`DEFERRABLE INITIALLY DEFERRED`** — checking postponed to commit (or `SET CONSTRAINTS … IMMEDIATE`).
- **Exclusion constraint** — generalization of uniqueness using operators (`EXCLUDE USING gist (room WITH =, during WITH &&)`).
- **`ON CONFLICT`** — `INSERT … ON CONFLICT (cols) DO NOTHING | DO UPDATE` (upsert), relies on a unique index.
- **Idempotency key** — client-supplied unique token that makes retried requests safe.

### 3. How it works, step by step
**A. CHECK constraint on a huge table**
1. Add as not valid (brief lock, no scan):
   ```sql
   ALTER TABLE orders ADD CONSTRAINT orders_amount_pos CHECK (amount > 0) NOT VALID;
   ```
   Lock: `ACCESS EXCLUSIVE` but held only long enough to change the catalog (milliseconds) — still has to wait for existing transactions touching the table, so set a lock timeout.
2. From now on, new/updated rows are checked.
3. Fix existing violations (`UPDATE … SET amount = … WHERE amount <= 0`), then:
   ```sql
   ALTER TABLE orders VALIDATE CONSTRAINT orders_amount_pos;
   ```
   Lock: `SHARE UPDATE EXCLUSIVE` — allows normal reads and writes (`SELECT`, `INSERT`, `UPDATE`, `DELETE`) concurrently; blocks only other schema changes and `VACUUM FULL`. It performs a full scan but doesn't stop traffic.

**B. Foreign key on a huge table**
1. `ALTER TABLE order_items ADD CONSTRAINT fk_items_order FOREIGN KEY (order_id) REFERENCES orders(id) NOT VALID;`
   Locks: `SHARE ROW EXCLUSIVE` on **both** the child and the parent — blocks writes to those tables during the (short) catalog change.
2. `ALTER TABLE order_items VALIDATE CONSTRAINT fk_items_order;`
   Locks: `SHARE UPDATE EXCLUSIVE` on the child and `ROW SHARE` on the parent; writes continue.
3. Create the supporting **index on the child's FK column** first (`CREATE INDEX CONCURRENTLY`), else validation and parent deletes scan the child.

**C. Unique constraint without blocking**
1. `CREATE UNIQUE INDEX CONCURRENTLY users_email_uq ON users (lower(email));` (two table scans, no write lock; can't run inside a transaction block; if it fails it leaves an `INVALID` index to drop).
2. Optionally attach: `ALTER TABLE users ADD CONSTRAINT users_email_key UNIQUE USING INDEX users_email_uq;` (fast, brief lock).

**D. `NOT NULL`**
`SET NOT NULL` scans the table under `ACCESS EXCLUSIVE`. Trick: first add `CHECK (col IS NOT NULL) NOT VALID`, `VALIDATE` it, then `ALTER COLUMN … SET NOT NULL` — PostgreSQL 12+ recognizes the validated check and skips the scan; then drop the redundant check.

### 4. Worked example
```sql
-- 1. Amount must be positive (table constraint, online)
SET lock_timeout = '3s';   -- don't queue behind long transactions and block everyone
ALTER TABLE payments ADD CONSTRAINT payments_amount_chk CHECK (amount > 0) NOT VALID;
-- find legacy violations
SELECT id, amount FROM payments WHERE NOT (amount > 0) LIMIT 100;
-- clean up in batches, then:
ALTER TABLE payments VALIDATE CONSTRAINT payments_amount_chk;

-- 2. Exactly one default address per user: partial unique index
CREATE UNIQUE INDEX CONCURRENTLY one_default_address_per_user
  ON addresses (user_id) WHERE is_default;
-- atomically flip the default inside one transaction:
BEGIN;
UPDATE addresses SET is_default = false WHERE user_id = 7 AND is_default;
UPDATE addresses SET is_default = true  WHERE id = 99 AND user_id = 7;
COMMIT;
-- the index guarantees two concurrent flips can't both end with a default; one fails with 23505 and retries
-- (statement order matters: a single UPDATE ... SET is_default = (id = 99) may fail transiently
--  because uniqueness is checked per row unless the constraint is DEFERRABLE; a unique *constraint* can be deferrable, a partial unique *index* cannot)

-- 3. Soft delete without blocking re-signup
CREATE UNIQUE INDEX CONCURRENTLY users_email_active_uq
  ON users (lower(email)) WHERE deleted_at IS NULL;

-- 4. Idempotent checkout: the unique key makes retries safe
CREATE TABLE checkouts (
  id               bigserial PRIMARY KEY,
  idempotency_key  uuid NOT NULL UNIQUE,
  order_id         bigint,
  created_at       timestamptz NOT NULL DEFAULT now()
);
INSERT INTO checkouts (idempotency_key) VALUES ($1)
ON CONFLICT (idempotency_key) DO NOTHING
RETURNING id;           -- no row returned = a retry; fetch the stored result instead of creating a second order

-- 5. No overlapping bookings per room
CREATE EXTENSION IF NOT EXISTS btree_gist;
ALTER TABLE bookings ADD CONSTRAINT no_overlap
  EXCLUDE USING gist (room_id WITH =, tstzrange(starts_at, ends_at) WITH &&);

-- 6. Tenant integrity through a composite foreign key
ALTER TABLE projects ADD CONSTRAINT projects_tenant_uq UNIQUE (tenant_id, id);
ALTER TABLE tasks ADD CONSTRAINT tasks_project_fk
  FOREIGN KEY (tenant_id, project_id) REFERENCES projects (tenant_id, id);
```
Reading state: `SELECT conname, convalidated FROM pg_constraint WHERE conrelid = 'payments'::regclass;` — `convalidated = false` means `NOT VALID` pending validation. Locks in flight: `SELECT * FROM pg_locks l JOIN pg_stat_activity a USING (pid) WHERE NOT granted;`.

### 5. Under the hood
- **Why NOT VALID is cheap.** The catalog row in `pg_constraint` is inserted with `convalidated = false`. The enforcement code (check expressions executed by the executor for each new tuple version; FK triggers fired as internal `RI_ConstraintTrigger_*` triggers) is active immediately, but nothing reads the existing heap.
- **Validation is just a scan.** `VALIDATE CONSTRAINT` for `CHECK` runs a sequential scan evaluating the expression; for FK it runs an anti-join query (`SELECT … FROM child LEFT JOIN parent … WHERE parent IS NULL`). Under `SHARE UPDATE EXCLUSIVE` concurrent writers continue, and because they are checked on write, no violation can appear behind the scan.
- **Foreign keys are triggers.** An FK is implemented by system triggers on both tables; deleting a parent row runs a lookup in the child (needs an index on the child columns to be fast, otherwise one sequential scan per deleted parent row) and takes `FOR KEY SHARE` locks on the parent row when inserting children. That locking is why FK-heavy schemas can show lock waits on hot parents.
- **Unique via index + MVCC.** A unique index may contain entries for dead or in-progress tuples; on conflict PostgreSQL waits for the other transaction to finish, then decides. `CREATE INDEX CONCURRENTLY` runs in multiple phases (build, wait for old snapshots, second pass, wait again), and *can fail on duplicates* mid-way leaving an invalid index.
- **Lock queue effect.** An `ACCESS EXCLUSIVE` request waits behind a long-running transaction, and every later query then waits behind it — the real production danger. Setting `lock_timeout` plus retry makes migrations safe.
- **Deferrable constraints** are stored with a flag; checks are queued in memory per transaction and run at commit; a large number of deferred checks costs memory. Only unique/PK/FK/EXCLUDE can be deferrable; `CHECK` cannot (it's always immediate).
- **`ON CONFLICT`** uses speculative insertion: attempt to insert the tuple, and on a conflict in the arbiter index abort the speculation and take the `DO NOTHING/UPDATE` path, all within one statement and safe under concurrency.

### 6. Performance and design implications
- Always index foreign key columns on the child side; consider `ON DELETE` actions' cost for big parents.
- Validation scans the whole table: schedule off-peak, and use `statement_timeout`/`lock_timeout` appropriately (don't set a statement timeout shorter than the scan). Monitor with `pg_stat_progress_create_index` and `pg_stat_activity`.
- Prefer database constraints as the **source of truth**; app validation is for UX. Map SQLSTATEs to messages: `23505` unique_violation, `23503` foreign_key_violation, `23514` check_violation, `23502` not_null_violation, `23P01` exclusion_violation.
- Pre-flight before adding uniqueness: `SELECT key, count(*) FROM t GROUP BY key HAVING count(*) > 1;` and decide a dedupe strategy (keep latest, merge) in a transaction or batch; the same query becomes a CI/migration guard.
- Choice of representation: `CHECK (country IN (…))` (cheap, list in DDL), lookup table + FK (changeable by data, with metadata), or `ENUM` (compact, awkward to remove values). For lists that change, use the lookup table.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ADD CONSTRAINT … NOT VALID` | Enforce new writes only | Step 1 on big tables |
| `VALIDATE CONSTRAINT name` | Check old rows under a light lock | Step 2 |
| `CREATE [UNIQUE] INDEX CONCURRENTLY` | Build without write lock | Unique/partial indexes |
| `ADD CONSTRAINT … UNIQUE USING INDEX idx` | Promote index to constraint | After concurrent build |
| `SET lock_timeout = '2s'` | Fail fast instead of queueing | Every DDL migration |
| `DEFERRABLE INITIALLY DEFERRED` / `SET CONSTRAINTS ALL DEFERRED` | Check at commit | Circular FKs, swaps |
| `EXCLUDE USING gist (…)` | Non-overlap rule | Bookings, schedules |
| `INSERT … ON CONFLICT` | Upsert/idempotency | Retry-safe writes |
| `pg_constraint.convalidated` | Validation status | Audit migrations |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `NOT VALID` + `VALIDATE` | Table is tiny (one-step is fine) |
| Partial unique index | Need `DEFERRABLE` or to be referenced by an FK (use a real unique constraint) |
| `CHECK` | Rule needs data from other rows/tables (use FK, trigger, exclusion) |
| Lookup table + FK | List is static and tiny (CHECK is simpler) |
| `ENUM` | Values change frequently |
| Deferred constraint | Immediate feedback matters or deferred checks would be huge |
| App-only validation | Always pair with DB constraints for integrity |

### 9. Common mistakes
- One-step `ADD CONSTRAINT` on a hot table → minutes of blocked traffic → `NOT VALID` then `VALIDATE`.
- No `lock_timeout` → migration queues behind a long transaction and everything stalls → set it and retry.
- Creating a unique index non-concurrently → blocks writes → `CONCURRENTLY`; and clean up `INVALID` indexes after failure.
- Assuming `CHECK` rejects NULL → passes on NULL → add `NOT NULL`.
- No index on the FK column → slow parent deletes, lock pile-ups → index it.
- Soft delete + plain `UNIQUE(email)` → can't re-register → partial unique index on `WHERE deleted_at IS NULL`.
- Surfacing raw DB errors to users → leaks schema → catch SQLSTATE and translate.
- Validating a constraint before cleaning data → validation fails after a long scan → pre-query violations.

### 10. 🎯 Interview kit
**30-second answer:** "To add a constraint to a large table I use `ADD CONSTRAINT … NOT VALID`, which only takes a brief lock and enforces new writes, then `VALIDATE CONSTRAINT`, which scans existing rows under `SHARE UPDATE EXCLUSIVE` so traffic continues. For uniqueness I build the index with `CREATE UNIQUE INDEX CONCURRENTLY`. I set a lock timeout, fix violations first, and keep constraints in the database as the source of truth."

**Structure a longer answer:** why one-step DDL is dangerous (lock + scan + queue) → two-step pattern with exact locks → unique/partial indexes → data cleanup → application error handling and retries.

**Likely follow-ups:**
- *What does a `NOT VALID` constraint still enforce?* → all new inserts and updates; only existing rows are unchecked.
- *Locks for FK step 1 vs. 2?* → `SHARE ROW EXCLUSIVE` on both tables (short) vs. `SHARE UPDATE EXCLUSIVE` on child (long scan, writes allowed).
- *Idempotency keys?* → unique key + `ON CONFLICT DO NOTHING`, return stored result.
- *One default per user?* → partial unique index `WHERE is_default`.
- *Prevent overlapping bookings?* → exclusion constraint with range type and `&&`.
- *DEFERRABLE?* → check at commit; for circular references and swaps.
- *Multi-tenant integrity?* → composite FK including `tenant_id`.

**Signals interviewers look for:** you know the exact lock modes, think about queueing behaviour, and treat the database as the final guard rather than relying on application checks.

### 11. Check yourself
- [ ] Add a CHECK constraint to a 200M-row table with minimal blocking; list each statement and its lock.
- [ ] Explain why `lock_timeout` matters for DDL.
- [ ] Build a partial unique index for soft-deleted rows and test with two concurrent sessions.
- [ ] Design an idempotent endpoint backed by a unique key and `ON CONFLICT`.
- [ ] Add `NOT NULL` online using a validated CHECK on PostgreSQL 12+.
- [ ] Explain what makes an exclusion constraint different from a unique constraint.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is a CHECK constraint and give an example of enforcing valid data with it.](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36263_What_is_a_CHECK_constraint_and_give_an_example_of_enforcing_.md>)
- [Intermediate] [What happens when you try to delete a parent row that still has child rows referencing it via a foreign key?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36180_What_happens_when_you_try_to_delete_a_parent_row_that_still_.md>)
- [Intermediate] [How would you enforce that an order amount is always greater than zero when creating the table?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36227_How_would_you_enforce_that_an_order_amount_is_always_greater.md>)
- [Intermediate] [Your form validation allows a value the DB rejects with a CHECK/unique violation at 3am; how do you surface a clean error to the user and keep DB constraints as the source of truth?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36031_Your_form_validation_allows_a_value_the_DB_rejects_with_a_CH.md>)
- [Deep] [Two rows with the same email got inserted despite you expecting uniqueness; how would you prevent this at the database level going forward?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36234_Two_rows_with_the_same_email_got_inserted_despite_you_expect.md>)
- [Deep] [How would you enforce that only one row can be marked as the default/primary per user using a partial unique index?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35931_How_would_you_enforce_that_only_one_row_can_be_marked_as_the.md>)
- [Deep] [You want to enforce that a child row's tenant_id always matches its parent's tenant_id across a foreign key; how would you model a composite foreign key to guarantee tenant integrity and why is this better than trusting the app?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-36050_You_want_to_enforce_that_a_child_row_s_tenant_id_always_matc.md>)
- [Expert] [Why should you add a NOT VALID foreign key then VALIDATE it separately when adding constraints to a big table?](<../../../PRACTICE/TOOLS/PostgreSQL/PostgreSQL_Platform/Q-35919_Why_should_you_add_a_NOT_VALID_foreign_key_then_VALIDATE_it_.md>)

## Related concepts

- [Check and NOT NULL Constraints](<../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Check_and_NOT_NULL_Constraints.md>) *(Constraints and Keys)*
- [Unique Constraints for Dedup](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Unique_Constraints_for_Dedup.md>) *(SQL Fundamentals)*
- [Unique and Deferred Constraints](<../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Unique_and_Deferred_Constraints.md>) *(Constraints and Keys)*

