---
type: "practice"
difficulty: "Basic"
---

# What are the different types of SQL commands (DDL, DML, DCL, TCL)?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1 · [↑ Dynamic SQL](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Dynamic_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question asks me to classify SQL statements into the four (sometimes five) standard command categories — DDL, DML, DCL, TCL (and often DQL) — and explain what each does. This is a foundational IT-services / SDE screening question. The interviewer is checking whether I can (a) name the categories, (b) list the statements in each, (c) explain the key behavioral difference between them — especially which are auto-committed (DDL/DCL) versus which participate in a transaction (DML/TCL) — and (d) not confuse SELECT/query commands with data manipulation.

#### Assumptions
/clarifications: I'll use the widely taught five-way split (DDL, DML, DCL, TCL, DQL). Some textbooks fold SELECT into DML; I'll mention SELECT explicitly as DQL because that's the cleaner, modern breakdown and interviewers accept either as long as I'm consistent. Plan: define each category, list its commands with a one-line purpose, note the commit behavior, then give a summary table and a common mistakes list.

The categories from first principles

SQL statements are grouped by what they operate on: the schema/structure (DDL), the data inside tables (DML), access/permissions (DCL), transaction boundaries (TCL), and reading data (DQL).

1. DDL — Data Definition Language. Defines and changes the structure of database objects (tables, indexes, views, schemas). Commands: CREATE, ALTER, DROP, TRUNCATE, RENAME. Key property: DDL is auto-committed — the change is permanent as soon as it runs and (in engines like Oracle/MySQL) forces an implicit COMMIT of any pending transaction. You use DDL to build the shape of the database: CREATE TABLE, ALTER TABLE ... ADD COLUMN, DROP INDEX, TRUNCATE TABLE.

2. DML — Data Manipulation Language. Manipulates the rows/data inside existing tables. Commands: INSERT, UPDATE, DELETE (and MERGE). Key property: DML changes are transactional — they happen inside a transaction and can be COMMITted or ROLLBACKed. This is the everyday CRUD-write layer: INSERT INTO, UPDATE ... SET, DELETE FROM.

3. DCL — Data Control Language. Controls access rights and permissions. Commands: GRANT (give a privilege to a user/role) and REVOKE (take it away). Example: GRANT SELECT ON orders TO analyst_role;. Like DDL, DCL is typically auto-committed.

4. TCL — Transaction Control Language. Manages transaction boundaries — the atomic units of work formed by DML. Commands: COMMIT (make DML changes permanent), ROLLBACK (undo them), SAVEPOINT (mark a partial rollback point), SET TRANSACTION (set isolation/read-only). These are the levers that give DML its all-or-nothing (ACID atomicity) behavior. TCL only affects DML — you cannot ROLLBACK a completed DDL statement in engines with auto-commit DDL.

5. DQL — Data Query Language. Retrieves/reads data. The command is SELECT (with WHERE, JOIN, GROUP BY, ORDER BY, etc.). It does not modify data. Many older syllabi lump SELECT under DML; the cleaner classification calls it DQL. Either is acceptable if stated clearly.

The one idea that ties it together

The single most important behavioral distinction — and the follow-up interviewers love — is commit behavior:

DDL and DCL are auto-committed: run them and they're permanent; they cannot be rolled back with ROLLBACK (in auto-commit-DDL engines like Oracle/MySQL). PostgreSQL and SQL Server support transactional DDL, so there DDL can be rolled back inside an explicit transaction — a nice nuance to add.
DML is not auto-committed: it lives inside a transaction and is governed by TCL (COMMIT/ROLLBACK/SAVEPOINT).

So DML and TCL are a pair: DML makes the changes, TCL decides whether they stick.

Worked feel

CREATE TABLE emp(...) (DDL, committed) → INSERT INTO emp VALUES(...) (DML) → SAVEPOINT s1 (TCL) → UPDATE emp SET sal=sal*1.1 (DML) → ROLLBACK TO s1 (TCL, undoes the update but keeps the insert) → COMMIT (TCL, insert is now permanent) → GRANT SELECT ON emp TO reporting (DCL, committed) → SELECT * FROM emp (DQL).

#### Common mistakes
/ follow-up probes
Calling TRUNCATE a DML command — it's DDL (removes all rows via page deallocation, auto-committed, resets identity), unlike DELETE which is DML.
Putting SELECT under "control" commands — it's a query (DQL/DML-read), never DCL/TCL.
Confusing DCL (GRANT/REVOKE, permissions) with TCL (COMMIT/ROLLBACK, transactions) — both start with "control," but one controls access, the other controls transactions.
Saying DDL can always be rolled back — only in transactional-DDL engines (PostgreSQL, SQL Server), not Oracle/MySQL.

#### Follow-up
"Where does MERGE go?" → DML. "Where does COMMENT/RENAME go?" → DDL.

Summary — what a strong answer emphasizes: name all four/five categories with their commands, state each category's purpose in one line, and nail the behavioral split — DDL/DCL auto-commit, DML is transactional and governed by TCL, DQL just reads. Add the TRUNCATE-is-DDL and transactional-DDL nuances to stand out.

**Comparison:**

```
Category	Full name	Commands	Purpose	Commit behavior
DDL	Data Definition Language	CREATE, ALTER, DROP, TRUNCATE, RENAME	Define/modify schema & object structure	Auto-commit (transactional in PG/SQL Server)
DML	Data Manipulation Language	INSERT, UPDATE, DELETE, MERGE	Add/modify/remove rows (data)	Transactional (needs COMMIT)
DCL	Data Control Language	GRANT, REVOKE	Manage privileges/access	Auto-commit
TCL	Transaction Control Language	COMMIT, ROLLBACK, SAVEPOINT, SET TRANSACTION	Manage transaction boundaries for DML	N/A (they are the controls)
DQL	Data Query Language	SELECT	Retrieve/read data	Read-only
```

**Code:**

```sql
-- DDL: defines structure (auto-committed)
CREATE TABLE emp (id INT PRIMARY KEY, name VARCHAR(50), sal DECIMAL(10,2));
ALTER TABLE emp ADD COLUMN dept VARCHAR(30);
TRUNCATE TABLE emp;                 -- DDL, not DML
DROP TABLE emp;

-- DML: manipulates rows (transactional)
INSERT INTO emp (id, name, sal) VALUES (1, 'Asha', 50000);
UPDATE emp SET sal = sal * 1.10 WHERE id = 1;
DELETE FROM emp WHERE id = 1;

-- DQL: reads data
SELECT id, name FROM emp WHERE sal > 40000 ORDER BY sal DESC;

-- TCL: transaction boundaries (governs DML)
SAVEPOINT s1;
ROLLBACK TO s1;                     -- undo DML back to the savepoint
COMMIT;                            -- make DML permanent

-- DCL: access control (auto-committed)
GRANT SELECT, INSERT ON emp TO analyst_role;
REVOKE INSERT ON emp FROM analyst_role;
```

**Example input / output:**

```text
Sequence and effect:

CREATE TABLE emp(...)          [DDL]  -> committed immediately
INSERT INTO emp VALUES(1,...)  [DML]  -> pending in transaction
SAVEPOINT s1                   [TCL]  -> marker set
UPDATE emp SET sal=sal*1.1     [DML]  -> pending
ROLLBACK TO s1                 [TCL]  -> UPDATE undone; INSERT still pending
COMMIT                         [TCL]  -> INSERT now permanent
GRANT SELECT ON emp TO rpt     [DCL]  -> committed immediately
SELECT * FROM emp              [DQL]  -> returns row id=1 (sal unchanged, 50000)
```

**Flow:**

1. controlled by
2. SQL commands
3. DDL:CREATE/ALTER/DROP/TRUNCATEstructure - auto commit
4. DML: INSERT/UPDATE/DELETErows - transactional
5. DCL: GRANT/REVOKEpermissions - auto commit
6. TCL:COMMIT/ROLLBACK/SAVEPOINTtransaction boundaries
7. DQL: SELECTread data

