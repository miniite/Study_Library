---
type: "practice"
difficulty: "Basic"
---

# Explain the difference between DDL, DML, DCL, and TCL commands.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ SQL vs NoSQL](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/SQL_vs_NoSQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What's really being asked: A classic fundamentals question that IT-services and SaaS interviewers use as a warm-up filter: can I cleanly categorize SQL commands into the four sub-languages, name the commands in each, and explain the behavioral differences that actually matter — especially the transaction and rollback behavior that separates DDL from DML? A weak answer just lists commands; a strong answer explains why the categories exist and one or two consequences (auto-commit of DDL, what TCL controls).

#### Assumptions
Standard SQL across common engines. I'll note the one big engine difference (whether DDL is transactional) since it's the most interview-worthy nuance.

Plan: define each category, list its commands, give the defining property, then a comparison table and the key gotchas.

SQL is not one flat language; its statements are grouped into sub-languages by what they act on and how they interact with transactions. The four groups:

DDL — Data Definition Language

Defines and changes the structure (schema) of the database — the tables, columns, indexes, constraints themselves, not the rows inside them.

Commands: CREATE, ALTER, DROP, TRUNCATE, RENAME. (COMMENT too, in some engines.)
Defining property: in most databases (Oracle, MySQL, and by default SQL Server) DDL is auto-committing — issuing a CREATE/ALTER/DROP implicitly commits the current transaction and cannot be rolled back with ROLLBACK. (PostgreSQL is the notable exception: DDL is transactional there and can be rolled back inside a transaction block.)
TRUNCATE is DDL, not DML: it deallocates the table's data pages wholesale rather than deleting rows one by one, so it's fast, resets identity counters, and generally can't be rolled back or fire row triggers — unlike DELETE.
DML — Data Manipulation Language

Reads and modifies the rows (the data) inside existing objects.

Commands: INSERT, UPDATE, DELETE, MERGE (upsert), and — depending on the textbook's convention — SELECT. (Purists split SELECT into a separate DQL / Data Query Language; many interviewers accept SELECT under DML. I'd say: "SELECT is often called DQL, but if we lump querying in, it's DML.")
Defining property: DML changes are part of a transaction — they are not permanent until you COMMIT, and they can be rolled back. This is the crucial contrast with DDL.
DCL — Data Control Language

Controls permissions / access rights — who is allowed to do what.

Commands: GRANT (give a privilege), REVOKE (take it away).

#### Example
GRANT SELECT, INSERT ON orders TO analyst_role;
These are about security/authorization, not data or schema shape.
TCL — Transaction Control Language

Manages transactions — grouping DML statements into atomic units of work.

Commands: COMMIT (make changes permanent), ROLLBACK (undo since last commit), SAVEPOINT (mark a point to partially roll back to), ROLLBACK TO SAVEPOINT, and SET TRANSACTION (set isolation level, read-only, etc.).
TCL is what makes DML's "not permanent until committed" promise usable: you wrap several INSERT/UPDATE/DELETEs, and either all succeed (COMMIT) or you back out (ROLLBACK). This is the "A" (atomicity) in ACID.
Why the categorization matters

The line that shows real understanding: DML operations are transactional and reversible (via TCL); DDL operations typically auto-commit and are not reversible on most engines. So if you run a migration script that mixes a DROP TABLE in the middle of INSERTs, the DROP may commit everything before it — a real production footgun. Likewise TRUNCATE (DDL) can't be undone with ROLLBACK the way DELETE (DML) can. And DCL (GRANT/REVOKE) is a third concern entirely — security — that people sometimes wrongly lump with DDL.

Summary
DDL = structure: CREATE, ALTER, DROP, TRUNCATE, RENAME — usually auto-commit, not rollback-able.
DML = data/rows: INSERT, UPDATE, DELETE, MERGE (+ SELECT/DQL) — transactional, rollback-able.
DCL = permissions: GRANT, REVOKE.
TCL = transactions: COMMIT, ROLLBACK, SAVEPOINT, SET TRANSACTION.

#### Common mistakes
calling TRUNCATE a DML command (it's DDL); thinking you can ROLLBACK a DROP/TRUNCATE on Oracle/MySQL; forgetting MERGE/SAVEPOINT; and confusing DCL with DDL. Follow-up probes: "Is SELECT DML or DQL?", "Why is TRUNCATE faster than DELETE?" (bulk page deallocation, minimal logging, no per-row triggers), and "Can DDL be rolled back?" (engine-dependent — yes in PostgreSQL, no by default in Oracle/MySQL).

**Comparison:**

```
Category	Full name	Acts on	Key commands	Transactional / rollback?
DDL	Data Definition Language	Schema/structure	CREATE, ALTER, DROP, TRUNCATE, RENAME	Usually auto-commit, not rollback-able (PostgreSQL: yes)
DML	Data Manipulation Language	Rows/data	INSERT, UPDATE, DELETE, MERGE (+SELECT)	Yes — reversible until COMMIT
DCL	Data Control Language	Permissions	GRANT, REVOKE	Security-scoped
TCL	Transaction Control Language	Transactions	COMMIT, ROLLBACK, SAVEPOINT, SET TRANSACTION	It is the control mechanism
```

**Code:**

```sql
-- DDL: define/modify structure (typically auto-commits)
CREATE TABLE orders (id INT PRIMARY KEY, amount DECIMAL(10,2), status CHAR(1));
ALTER  TABLE orders ADD COLUMN created_at TIMESTAMP;
TRUNCATE TABLE orders;          -- DDL: fast wipe, resets identity, not rollback-able
DROP   TABLE orders;

-- DML: manipulate rows (part of a transaction, reversible)
INSERT INTO orders (id, amount, status) VALUES (1, 500.00, 'N');
UPDATE orders SET status = 'P' WHERE id = 1;
DELETE FROM orders WHERE id = 1;
SELECT * FROM orders;           -- query (DQL, often grouped under DML)

-- DCL: permissions
GRANT SELECT, INSERT ON orders TO analyst_role;
REVOKE INSERT ON orders FROM analyst_role;

-- TCL: transaction control wrapping DML
BEGIN;                                  -- start transaction
  INSERT INTO orders VALUES (2, 250.00, 'N');
  SAVEPOINT after_insert;
  UPDATE orders SET status = 'P' WHERE id = 2;
  ROLLBACK TO SAVEPOINT after_insert;   -- undo just the UPDATE
COMMIT;                                 -- make the INSERT permanent
```

**Example input / output:**

```

Rollback behavior contrast (why DDL vs DML matters):

CODE
Copy
-- DML is reversible:
BEGIN;
DELETE FROM orders;      -- rows gone (uncommitted)
ROLLBACK;                -- rows RESTORED  ✅

-- DDL (TRUNCATE) on Oracle/MySQL is NOT:
BEGIN;
TRUNCATE TABLE orders;   -- auto-commits immediately; rows gone
ROLLBACK;                -- has no effect — rows are NOT restored  ❌

Input: a table with 100 rows. DELETE + ROLLBACK → 100 rows remain. TRUNCATE + ROLLBACK (Oracle/MySQL) → 0 rows remain.
```

**Flow:**

1. controlled by
2. usually auto-commits
3. SQL commands
4. DDL: structureCREATE ALTER DROPTRUNCATE
5. DML: rowsINSERT UPDATE DELETEMERGE
6. DCL: accessGRANT REVOKE
7. TCL: transactionsCOMMIT ROLLBACKSAVEPOINT

