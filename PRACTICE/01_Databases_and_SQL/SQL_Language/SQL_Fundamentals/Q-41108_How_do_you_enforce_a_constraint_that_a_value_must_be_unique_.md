---
type: "practice"
difficulty: "Expert"
---

# How do you enforce a constraint that a value must be unique only when another column is true (partial/filtered index)?

**Difficulty:** Expert · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Unique Constraints for Dedup](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Unique_Constraints_for_Dedup.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question: how do you enforce uniqueness on a column only when another column has a particular value? Classic example: a users table where the same email can appear many times across soft-deleted rows, but there must be at most one active row per email. Or: a subscriptions table where a user may have many historical subscriptions but only one with status = 'active' at a time. Or a default_address flag where exactly one address per user is the default.

At "expertise" level the interviewer is probing whether I know that a plain UNIQUE constraint can't express a conditional rule, that the right tool is a partial (a.k.a. filtered) unique index, and — crucially — the gotchas: NULL semantics in unique indexes, portability differences between Postgres/SQL Server/MySQL, why this beats application-level checks under concurrency, and the fallback when partial indexes aren't available. I'll assume Postgres as the primary dialect (most common in this profile) and note the others.

Why a plain UNIQUE constraint fails here

A UNIQUE (email) constraint enforces uniqueness across every row. But my rule is "unique among active rows only." If I made the whole column unique I couldn't keep the historical/soft-deleted duplicates I need. So I need to index a subset of rows.

The core answer: a partial unique index

A partial index is an index built over only the rows that satisfy a WHERE predicate. Make it UNIQUE and you get "uniqueness, but only over the filtered rows." In Postgres:

SQL
CREATE UNIQUE INDEX uq_users_active_email
ON users (email)
WHERE is_active = true;

Now the database physically refuses a second is_active = true row with the same email, while allowing unlimited inactive duplicates. Because it's enforced in the storage engine, it holds atomically under concurrency — two simultaneous transactions inserting the same active email will have one succeed and the other fail with a unique violation. That's the whole point: the invariant can never be violated, no matter how many app servers race.

The important NULL subtlety (soft-delete pattern)

A very common variant uses a nullable deleted_at instead of a boolean. People try WHERE deleted_at IS NULL — good. But some people instead try to fold "active" into the key itself, e.g. index (email, deleted_at) hoping NULL de-duplicates. That breaks, because in SQL two NULLs are not equal, so a standard unique index treats (a@x.com, NULL) and (a@x.com, NULL) as distinct — allowing duplicate active rows. The partial-index form (WHERE deleted_at IS NULL) avoids that trap entirely and is the idiomatic fix. (Postgres 15+ also adds UNIQUE NULLS NOT DISTINCT, but the partial index is clearer and older-version-safe.)

Dialect portability (expertise signal)
PostgreSQL / SQLite: CREATE UNIQUE INDEX ... WHERE predicate; — first-class.
SQL Server: filtered index, same idea: CREATE UNIQUE INDEX ... WHERE is_active = 1;.
MySQL/MariaDB (InnoDB): no partial/filtered indexes. Two common workarounds: (1) a generated column that is email when active and NULL when inactive, then a plain unique index on it (NULLs are allowed to repeat, so inactive rows don't collide); (2) enforce via a BEFORE INSERT/UPDATE trigger. Knowing MySQL lacks this natively is a strong differentiator.
SQL
-- MySQL workaround via generated column
ALTER TABLE users
  ADD active_email VARCHAR(255)
  GENERATED ALWAYS AS (IF(is_active, email, NULL)) STORED,
  ADD UNIQUE KEY uq_active_email (active_email);
Why this beats application-level enforcement

An app-level "SELECT then INSERT if none active" check has a race window: two requests both read "no active row," both insert, and now you have two active rows — the check-then-act TOCTOU bug. You'd have to serialize with SELECT ... FOR UPDATE or a transaction-level advisory lock, adding contention and complexity. The partial unique index pushes the invariant into the engine where it's enforced atomically and correctly for free, and it also self-documents the business rule in the schema. App validation is still valuable for friendly error messages, but the database index is the source of truth that can never be bypassed by a rogue script, a bulk import, or a second service writing the same table.

Operational notes
Partial indexes are smaller and faster than full ones (fewer rows), and the planner can use them for reads matching the predicate — a nice bonus (WHERE is_active AND email = ? gets an index-only style lookup).
Building it on a large live table: use CREATE UNIQUE INDEX CONCURRENTLY in Postgres to avoid a long write lock; be ready for it to fail if existing data already violates the rule (you must clean duplicates first).
Catch the unique-violation error code (23505 in Postgres) in the app and map it to a clean 409 Conflict.
Summary

Use a partial/filtered UNIQUE index with the condition in its WHERE clause — that's the correct, concurrency-safe, engine-enforced way to say "unique only when this other column is true." Watch the NULL-inequality trap, prefer it over app-level check-then-insert (which races), and know the MySQL fallback (generated column or trigger) since MySQL lacks native partial indexes.

#### Follow-up
probes: "What if two columns must jointly be conditionally unique?" (list both in the index column list) and "How do you migrate an existing dirty table?" (dedupe first, then CREATE ... CONCURRENTLY).

**Comparison:**

```
Approach	Conditional uniqueness?	Concurrency-safe?	Engines
Plain UNIQUE(col)	No (all rows)	Yes	All
Partial/filtered UNIQUE index	Yes	Yes (engine-enforced)	Postgres, SQLite, SQL Server
UNIQUE(col, flag) relying on NULLs	No — two NULLs aren't equal, so it leaks	N/A	pitfall to avoid
Generated NULLable column + UNIQUE	Yes	Yes	MySQL/MariaDB fallback
BEFORE-INSERT/UPDATE trigger	Yes	Yes but heavier	MySQL, others
App-level check-then-insert	Yes	No (TOCTOU race) unless locked	Any (not recommended alone)
```

**Code:**

```

-- PostgreSQL / SQLite: at most one ACTIVE row per email CREATE UNIQUE INDEX uq_users_active_email ON users (email) WHERE is_active = true;

-- Soft-delete variant: at most one non-deleted row per email CREATE UNIQUE INDEX uq_users_live_email ON users (email) WHERE deleted_at IS NULL;

-- "Exactly one default address per user" CREATE UNIQUE INDEX uq_one_default_addr_per_user ON addresses (user_id) WHERE is_default = true;

-- SQL Server (filtered index) CREATE UNIQUE INDEX uq_users_active_email ON users (email) WHERE is_active = 1;

CODE
Copy
```sql
-- MySQL / MariaDB fallback (no partial indexes): generated NULLable column
ALTER TABLE users
  ADD active_email VARCHAR(255)
    GENERATED ALWAYS AS (IF(is_active, email, NULL)) STORED,
  ADD UNIQUE KEY uq_active_email (active_email);
-- inactive rows -> active_email = NULL, and repeated NULLs are allowed,
-- so only active rows are constrained.
```

**Example input / output:**

```

Table users with the Postgres partial index uq_users_active_email in place:

CODE
Copy
-- Existing rows (allowed):
(1, 'a@x.com', is_active=false)   -- old soft-deleted account
(2, 'a@x.com', is_active=false)   -- another old one, duplicates OK
(3, 'a@x.com', is_active=true)    -- the one active account  ✅

-- Attempt to insert a SECOND active row:
INSERT INTO users(email, is_active) VALUES ('a@x.com', true);
-- => ERROR: duplicate key value violates unique constraint
--    "uq_users_active_email"  (SQLSTATE 23505)   ❌ blocked

-- Reactivating requires deactivating the other first (in one txn):
BEGIN;
  UPDATE users SET is_active=false WHERE id=3;
  UPDATE users SET is_active=true  WHERE id=1;   -- now the sole active row
COMMIT;                                          -- ✅ succeeds
```

