---
type: "practice"
difficulty: "Basic"
---

# What is a constraint? Explain NOT NULL, CHECK, UNIQUE, and DEFAULT.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1 · [↑ Normal Forms](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Normal_Forms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What the question is really asking: Define what a constraint is in a relational database, then explain four specific column/table constraints — NOT NULL, CHECK, UNIQUE, and DEFAULT — including what each enforces, when it's applied, and how it behaves at edge cases. This is a foundational DBMS question for freshers/entry-level SDEs.

What the interviewer is probing for: Whether you understand that constraints are the database's mechanism for enforcing data integrity declaratively — pushing correctness rules into the schema rather than trusting every application to validate. They also listen for the tricky behaviors: DEFAULT is not a constraint that rejects rows, UNIQUE allows NULLs (usually multiple), and CHECK in standard SQL treats NULL specially (a check passes if it evaluates to unknown).

#### Assumptions
/clarifications: I'll answer in standard SQL, noting a couple of engine differences (MySQL vs PostgreSQL) where behavior genuinely differs, since that's where the real interview value is. The candidate profile is fullstack/backend SDE, so a practitioner-flavored but foundational answer fits.

Plan: Define "constraint" and why it matters → explain each of the four with a definition, purpose, and gotcha → show DDL + insert examples that pass/fail → summarize with the key traps.

What is a constraint?

A constraint is a rule attached to a table (or column) that the database engine automatically enforces on every insert, update, or delete. Its job is data integrity: it guarantees that the data stored always satisfies certain conditions, no matter which application or user writes it. If an operation would violate a constraint, the DBMS rejects the whole statement (and, inside a transaction, that failure can roll back the transaction).

#### Cons
traints are valuable because they are declarative and centralized. Instead of hoping every microservice, script, and analyst remembers to validate "price must be non-negative," you declare it once in the schema and the database refuses bad data for everyone, forever. Common constraint types are PRIMARY KEY, FOREIGN KEY, NOT NULL, UNIQUE, CHECK, and DEFAULT (technically a default is a clause that supplies a value rather than a rejecting rule, but it's grouped with constraints).

#### Cons
traints can be column-level (written inline on one column) or table-level (written separately, and required when the rule spans multiple columns).

The four constraints
1. NOT NULL

What it enforces: the column must always contain a value; it can never be NULL.

Why: many columns are mandatory — an order must have a customer, a user must have an email. NULL means "unknown/absent," which is meaningless for required fields. Marking them NOT NULL blocks incomplete rows at write time.

Behavior: INSERT that omits a NOT NULL column (with no DEFAULT) fails; UPDATE ... SET col = NULL fails. Note: NOT NULL only bans NULL — it does not ban empty strings '' or 0; those are real values.

2. CHECK

What it enforces: a boolean predicate that each row must satisfy — an arbitrary condition on one or more columns of that row.

Why: to encode domain rules: age >= 18, price >= 0, status IN ('NEW','SHIPPED','CANCELLED'), end_date >= start_date.

Behavior / gotcha: In standard SQL, a CHECK passes if the predicate evaluates to TRUE or UNKNOWN. So if a column involved is NULL, the comparison is UNKNOWN and the check does not fail. That means CHECK (age >= 18) will allow a NULL age. If you need to forbid it, combine with NOT NULL. Engine note: MySQL only began enforcing CHECK from version 8.0.16 (older versions parsed but silently ignored it) — a classic "why didn't my constraint work" bug.

3. UNIQUE

What it enforces: no two rows may have the same value in the column(s) — every value (or combination of values) must be distinct.

Why: for natural keys that aren't the primary key: email, username, phone, or a composite like (user_id, workspace_id).

Behavior / gotcha: UNIQUE differs from PRIMARY KEY in two ways: (a) a table can have many UNIQUE constraints but only one primary key, and (b) UNIQUE allows NULLs. In most engines (PostgreSQL, Oracle, MySQL) NULL is treated as "not equal to anything, including another NULL," so you can insert multiple NULL rows into a UNIQUE column. (SQL Server historically allowed only one NULL; PostgreSQL 15+ adds UNIQUE NULLS NOT DISTINCT to opt into treating NULLs as equal.) A UNIQUE constraint is usually backed by a unique index, which also speeds up lookups.

4. DEFAULT

What it does: supplies a value automatically when an INSERT does not specify one for that column.

Why: convenience and consistency — created_at DEFAULT CURRENT_TIMESTAMP, status DEFAULT 'ACTIVE', is_deleted DEFAULT FALSE.

Behavior / gotcha: DEFAULT is not a rejecting constraint — it never causes an insert to fail; it just fills in a value when one is omitted. If you explicitly insert NULL into a column that has a DEFAULT but no NOT NULL, you get NULL, not the default — the default only kicks in when the column is omitted. To force the default, use the keyword DEFAULT in the insert or omit the column entirely. A default value must be a constant or a permitted function; it must itself satisfy the other constraints.

Summary — what a strong answer emphasizes
A constraint is a declarative integrity rule the DB enforces on every write, rejecting violating statements — validation lives in the schema, not scattered across apps.
NOT NULL → value required (but allows ''/0).
CHECK → arbitrary boolean rule per row; passes on NULL (TRUE-or-UNKNOWN), so pair with NOT NULL when needed.
UNIQUE → all values distinct; allows (multiple) NULLs; many per table vs one PRIMARY KEY.
DEFAULT → auto-fills when column omitted; never rejects a row; explicit NULL bypasses it.

#### Common mistakes
to avoid: thinking CHECK blocks NULLs; thinking UNIQUE blocks NULLs; thinking DEFAULT is used when you insert NULL explicitly; forgetting MySQL <8.0.16 ignored CHECK. Nailing those gotchas is what separates a strong answer from a textbook recital.

**Comparison:**

```
Constraint	Enforces	Rejects a row?	Key gotcha
NOT NULL	Column must have a value	Yes, if NULL inserted	Allows '' and 0 (they are real values)
CHECK	A boolean predicate per row	Yes, if predicate is FALSE	Passes when predicate is UNKNOWN (NULL)
UNIQUE	All values distinct	Yes, on duplicate non-NULL	Allows multiple NULLs (in most engines)
DEFAULT	Auto-fill when column omitted	Never	Explicit NULL bypasses it, not omission
```

**Code:**

```sql
CREATE TABLE Employee (
    emp_id     INT PRIMARY KEY,
    name       VARCHAR(50)  NOT NULL,                         -- must have a value
    email      VARCHAR(100) UNIQUE,                           -- distinct; NULLs allowed
    age        INT          CHECK (age >= 18),                -- domain rule
    department VARCHAR(30)  DEFAULT 'Unassigned',             -- auto-filled if omitted
    status     VARCHAR(10)  NOT NULL DEFAULT 'ACTIVE'
                            CHECK (status IN ('ACTIVE','INACTIVE')),
    -- table-level CHECK spanning conditions:
    CONSTRAINT chk_age_named CHECK (age IS NULL OR age <= 65)
);
```

**Example input / output:**

```

Given the Employee table above:

SQL
Copy
-- (1) PASSES: department & status auto-filled by DEFAULT
INSERT INTO Employee (emp_id, name, email, age) VALUES (1, 'Asha', 'asha@co.com', 25);
--> row = (1, 'Asha', 'asha@co.com', 25, 'Unassigned', 'ACTIVE')

-- (2) FAILS: name is NOT NULL and omitted
INSERT INTO Employee (emp_id, email) VALUES (2, 'x@co.com');
--> ERROR: NULL not allowed in column "name"

-- (3) FAILS: CHECK (age >= 18) violated
INSERT INTO Employee (emp_id, name, age) VALUES (3, 'Ravi', 16);
--> ERROR: check constraint violated (age >= 18)

-- (4) PASSES: CHECK allows NULL age (predicate is UNKNOWN, not FALSE)
INSERT INTO Employee (emp_id, name, age) VALUES (4, 'Meera', NULL);
--> row stored, age = NULL

-- (5) PASSES twice: UNIQUE allows multiple NULL emails
INSERT INTO Employee (emp_id, name, email) VALUES (5, 'Sam',  NULL);
INSERT INTO Employee (emp_id, name, email) VALUES (6, 'Nina', NULL);
--> both stored; NULL is "distinct from" NULL

-- (6) FAILS: duplicate non-NULL email
INSERT INTO Employee (emp_id, name, email) VALUES (7, 'Dup', 'asha@co.com');
--> ERROR: UNIQUE constraint violated

-- (7) explicit NULL bypasses DEFAULT (department has no NOT NULL):
INSERT INTO Employee (emp_id, name, department) VALUES (8, 'Om', NULL);
--> department = NULL, NOT 'Unassigned'
```

**Flow:**

1. no
2. yes
3. yes
4. no
5. no (dup non-NULL)
6. yes / NULL
7. FALSE
8. TRUE/UNKNOWN
9. INSERT / UPDATE arrives
10. NOT NULL cols present?
11. REJECT statement
12. Column omitted & hasDEFAULT?
13. fill DEFAULT value
14. UNIQUE values distinct?
15. CHECK predicate TRUE orUNKNOWN?
16. COMMIT row

