---
type: "practice"
difficulty: "Intermediate"
---

# What is a database trigger and give a practical use case for one.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ Dynamic SQL](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Dynamic_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What is really being asked: Define a database trigger and give a concrete, practical use case. The level is "intermediate experience," so the interviewer wants the definition plus the mechanics (event, timing, granularity, access to OLD/NEW row images), a real use case you'd actually build, and awareness of the trade-offs/pitfalls of putting logic in triggers.

What the interviewer is probing: Do you understand that a trigger is procedural code the database automatically fires in response to a data event — and can you justify when it's the right tool (auditing, enforcing invariants, maintaining derived data) versus when it becomes a hidden-logic liability? A strong candidate names the timing (BEFORE/AFTER/INSTEAD OF), the events (INSERT/UPDATE/DELETE), row-level vs statement-level, and the OLD/NEW pseudo-records.

#### Assumptions
/ clarifying questions: Syntax varies (Oracle PL/SQL, PostgreSQL trigger-function + CREATE TRIGGER, MySQL, SQL Server). I'll give the concepts generally and a runnable PostgreSQL + a MySQL example. The most convincing, universally-applicable use case is audit logging, so I'll build that, and also mention derived-column maintenance and validation.

Plan: Define trigger → the dimensions (event, timing, granularity, OLD/NEW) → a practical audit-log use case with code → other good use cases → pitfalls → summary.

What a database trigger is

A trigger is a named block of procedural code stored in the database that the DBMS executes automatically ("fires") in response to a specified data-modification event on a table (or view). You don't call it — the engine invokes it whenever the triggering event occurs. Triggers let the database enforce rules and side effects transparently, close to the data, so every application path that touches the table is subject to the same behavior — no matter which service, script, or console did the write.

A trigger is defined along four dimensions:

Triggering event: INSERT, UPDATE, DELETE (some engines also TRUNCATE, and DDL/logon triggers).
Timing: BEFORE the event (inspect/modify the incoming row, validate, or veto by raising an error), AFTER the event (react once the change is committed to the table — auditing, cascading writes), or INSTEAD OF (typically on views, to make a non-updatable view writable by redirecting the DML).
Granularity: FOR EACH ROW (row-level — fires once per affected row, with access to that row's data) vs. FOR EACH STATEMENT (fires once per SQL statement regardless of row count).
Row images: inside a row-level trigger you get pseudo-records — NEW (the incoming/updated row, available for INSERT/UPDATE) and OLD (the prior row, available for UPDATE/DELETE). In SQL Server these are the inserted and deleted pseudo-tables.
Practical use case: audit / change-history logging

The canonical production use case: capture who changed what and when, automatically, so no application code can forget to log. In fintech/payments (Razorpay, PhonePe) and enterprise systems, an immutable audit trail on sensitive tables (accounts, balances, KYC status) is often mandatory for compliance. An AFTER UPDATE/AFTER DELETE row-level trigger writes the OLD and NEW values into an audit_log table on every change — guaranteed, regardless of which service issued the SQL.

This is a great trigger use case precisely because:

It must fire for every write path (multiple microservices, admin tools, migration scripts) — enforcing it in the DB is airtight; enforcing it in each app is not.
It's a side effect the caller shouldn't have to think about.
It needs the OLD and NEW values, which triggers hand you for free.

(See the code and I/O sections for a full working audit trigger.)

Other strong use cases
Maintaining derived/denormalized data: keep a running account_balance, a comment_count on a post, or a search vector in sync when child rows change — an AFTER INSERT/UPDATE/DELETE trigger updates the rollup atomically in the same transaction.
Enforcing complex business invariants that a CHECK constraint can't express (cross-row or cross-table rules) — a BEFORE trigger validates and RAISEs an error to reject the write.
Auto-populating columns: set updated_at = now(), generate a slug, or normalize casing in a BEFORE INSERT/UPDATE trigger.
INSTEAD OF triggers on views: make a complex/joined view updatable by translating the DML into writes on the base tables.
Pitfalls & trade-offs (intermediate depth — this is what separates a good answer)
Hidden logic / "spooky action at a distance": triggers run invisibly, so a developer debugging a slow or surprising INSERT may not realize a trigger is doing extra work. Document them; keep them simple.
Performance: row-level triggers run per affected row — a bulk UPDATE of 1M rows fires the trigger 1M times. Prefer statement-level or set-based logic for bulk paths; keep trigger bodies lightweight.
Cascading / recursive triggers: a trigger that writes to a table with its own trigger can chain or recurse; guard against loops.
Transaction coupling: trigger work is part of the caller's transaction — if the trigger errors, the whole DML rolls back. Good for atomic auditing, but a slow/failing trigger blocks the write.
Testability & portability: logic in triggers is harder to unit-test and is engine-specific. Teams sometimes prefer application-layer or CDC-based approaches for auditing at scale. Know the trade-off.
Summary — what a strong answer emphasizes

A trigger is procedural code the database fires automatically on an INSERT/UPDATE/DELETE event, configurable by timing (BEFORE/AFTER/INSTEAD OF), granularity (row vs. statement), with access to the OLD/NEW row images. The strongest practical use case is automatic audit logging — writing every change to sensitive data into an audit table so no write path can bypass it — and closely related, maintaining derived data and enforcing invariants a CHECK can't. A senior-sounding answer pairs the use case with the caveats: hidden logic, per-row performance on bulk writes, recursion, and transaction coupling — and notes that at very large scale, change-data-capture is an alternative to trigger-based auditing.

#### Common mistakes
confusing BEFORE vs AFTER (BEFORE can modify/veto NEW; AFTER is for reacting/logging); forgetting OLD isn't available on INSERT and NEW isn't available on DELETE; ignoring the per-row performance cost on bulk operations.

#### Follow-up
probes: "How would you audit 10M-row bulk updates without a 10M-fire trigger?" (statement-level trigger or CDC/logical replication), and "BEFORE vs AFTER for validation vs logging — why?"

**Comparison:**

```
Dimension	Options	What it means
Event	INSERT / UPDATE / DELETE (also TRUNCATE, DDL)	Which data change fires the trigger
Timing	BEFORE / AFTER / INSTEAD OF	BEFORE: validate/modify/veto NEW; AFTER: react/log; INSTEAD OF: redirect (views)
Granularity	FOR EACH ROW / FOR EACH STATEMENT	Once per affected row vs. once per statement
Row images	NEW, OLD (SQL Server: inserted/deleted)	NEW: incoming row (INSERT/UPDATE); OLD: prior row (UPDATE/DELETE)
Use case	Timing/type	Why a trigger fits
Audit / change history	AFTER, row-level	Fires on every write path; needs OLD+NEW; atomic with the change
Auto-fill updated_at / slug	BEFORE, row-level	Can modify NEW before it's stored
Enforce complex invariant	BEFORE, row-level	RAISE error to veto writes a CHECK can't express
Maintain rollup/derived count	AFTER, row-level	Keep denormalized totals in sync in same transaction
Make a view updatable	INSTEAD OF, on view	Translate DML into base-table writes
```

**Code:**

```sql
-- ===== PostgreSQL: automatic AUDIT trigger on the accounts table =====
CREATE TABLE audit_log (
    id          BIGSERIAL PRIMARY KEY,
    table_name  TEXT,
    action      TEXT,            -- INSERT / UPDATE / DELETE
    row_pk      TEXT,
    old_data    JSONB,
    new_data    JSONB,
    changed_by  TEXT DEFAULT current_user,
    changed_at  TIMESTAMPTZ DEFAULT now()
);

-- The trigger FUNCTION holds the logic; NEW/OLD are the row images
CREATE OR REPLACE FUNCTION fn_audit_accounts()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        INSERT INTO audit_log(table_name, action, row_pk, new_data)
        VALUES ('accounts', 'INSERT', NEW.id::text, to_jsonb(NEW));
    ELSIF TG_OP = 'UPDATE' THEN
        INSERT INTO audit_log(table_name, action, row_pk, old_data, new_data)
        VALUES ('accounts', 'UPDATE', NEW.id::text, to_jsonb(OLD), to_jsonb(NEW));
    ELSIF TG_OP = 'DELETE' THEN
        INSERT INTO audit_log(table_name, action, row_pk, old_data)
        VALUES ('accounts', 'DELETE', OLD.id::text, to_jsonb(OLD));
    END IF;
    RETURN COALESCE(NEW, OLD);   -- AFTER trigger: return value is ignored, but be safe
END;
$$;

-- Fire AFTER the change, once per affected row
CREATE TRIGGER trg_audit_accounts
AFTER INSERT OR UPDATE OR DELETE ON accounts
FOR EACH ROW EXECUTE FUNCTION fn_audit_accounts();

-- ===== BEFORE trigger: auto-maintain updated_at and reject invalid change =====
CREATE OR REPLACE FUNCTION fn_accounts_biu()
RETURNS TRIGGER LANGUAGE plpgsql AS $$
BEGIN
    NEW.updated_at := now();                 -- auto-populate
    IF NEW.balance < 0 THEN
        RAISE EXCEPTION 'balance cannot be negative';  -- veto the write
    END IF;
    RETURN NEW;                              -- BEFORE trigger passes NEW forward
END; $$;

CREATE TRIGGER trg_accounts_biu
BEFORE INSERT OR UPDATE ON accounts
FOR EACH ROW EXECUTE FUNCTION fn_accounts_biu();

-- ===== MySQL equivalent audit trigger (no separate function object) =====
DELIMITER //
CREATE TRIGGER trg_audit_accounts_upd
AFTER UPDATE ON accounts
FOR EACH ROW
BEGIN
    INSERT INTO audit_log(table_name, action, row_pk, old_balance, new_balance, changed_at)
    VALUES ('accounts', 'UPDATE', OLD.id, OLD.balance, NEW.balance, NOW());
END//
DELIMITER ;
```

**Example input / output:**

```text
accounts (before):
  id | balance | updated_at
  7  | 500.00  | 2026-07-01 09:00

-- App runs (from ANY service, no logging code needed):
UPDATE accounts SET balance = 350.00 WHERE id = 7;

Effect:
  1. BEFORE trigger trg_accounts_biu fires:
       NEW.updated_at := now()  -> 2026-07-07 10:15
       NEW.balance (350) >= 0   -> passes, write proceeds
  2. Row updated -> balance=350.00, updated_at=2026-07-07 10:15
  3. AFTER trigger trg_audit_accounts fires (TG_OP='UPDATE'):
       INSERT INTO audit_log (old_data={id:7,balance:500}, new_data={id:7,balance:350})

audit_log (new row):
  action='UPDATE' row_pk='7'
  old_data={"id":7,"balance":500.00}
  new_data={"id":7,"balance":350.00}
  changed_by='app_user'  changed_at='2026-07-07 10:15'

-- Rejected write (invariant enforced by BEFORE trigger):
UPDATE accounts SET balance = -20 WHERE id = 7;
  -> ERROR: balance cannot be negative   (whole statement rolled back, nothing logged)
```

**Flow:**

1. No
2. Yes
3. DML issued:INSERT/UPDATE/DELETE onaccounts
4. BEFORE trigger fires
5. Valid? balance >= 0
6. RAISE EXCEPTION ->statement rolls back
7. Set NEW.updated_at, applyrow change
8. AFTER trigger fires
9. INSERT OLD+NEW intoaudit_log
10. Commit: data change + auditrow atomic

