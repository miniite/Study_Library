---
type: "practice"
difficulty: "Basic"
---

# What is a primary key and how is it different from a unique key?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Foreign Keys and Referential Integrity](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Foreign_Keys_and_Referential_Integrity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question — "What is a primary key and how is it different from a unique key?" — tests whether you understand entity integrity in relational databases and the subtle-but-important distinctions between two constraints that both enforce uniqueness. The interviewer is checking that you know:

What a primary key is (the canonical row identifier) and its two guarantees: unique + not null.
That a unique key also enforces uniqueness but permits NULLs and there can be many of them per table.
The practical/physical differences: number allowed per table, NULL handling, default index type (clustered vs non-clustered), and their role in relationships (foreign keys).

My plan: define both from first principles (what "key" means, why we need uniqueness), then contrast them across the axes that matter, with a concrete CREATE TABLE example, and finish with when to use each.

First principles: what is a key?

A relational table is a set of rows, and we need a reliable way to identify each row uniquely and to enforce business rules about uniqueness. A "key" is a column (or set of columns) whose values distinguish rows.

Primary key (PK). The primary key is the column (or combination of columns) chosen as the table's official, canonical identifier for each row. It enforces entity integrity with two guarantees:

Uniqueness — no two rows may share the same primary-key value.
NOT NULL — a primary-key column can never be null; every row must have an identity.

A table can have at most one primary key (though that key may span multiple columns — a composite primary key). The primary key is what foreign keys in other tables typically reference to form relationships. Example: user_id in a users table.

Unique key (unique constraint). A unique key also enforces that values in the column(s) are not duplicated, but with two key relaxations:

It allows NULL values. (Standard SQL treats each NULL as distinct, so a nullable unique column can hold multiple NULLs — engine behaviour varies; SQL Server historically allows only one NULL, PostgreSQL/MySQL/Oracle allow many.)
A table can have many unique keys.

#### Example
an email column should be unique (no two users share an email) but might legitimately be null for a user who signed up via phone — that's a job for a unique key, not a primary key.

The differences, axis by axis

1. Number per table.

PK: exactly one per table (or none, but at most one).
Unique: many allowed.

2. NULL handling.

PK: never null.
Unique: allows null (and usually multiple nulls, engine-dependent). This is the single most important distinction candidates should lead with.

3. Purpose / semantics.

PK: the primary identity of the row — the thing foreign keys reference, the thing you use in WHERE id = ? lookups.
Unique: enforces an alternate business-uniqueness rule (e.g., email, username, phone) that isn't the row's identity.

4. Physical storage / index. Both are backed by an index (that's how the DB enforces uniqueness quickly). But:

In many engines (notably SQL Server and, by default, InnoDB/MySQL) the primary key creates a clustered index — the table's rows are physically ordered by the PK. A unique key creates a non-clustered (secondary) index.
Because there can be only one physical row ordering, there can be only one clustered index — reinforcing why there's only one PK.

5. Foreign key references. Foreign keys can reference either a primary key or a unique key (the referenced column must be uniquely constrained). But the PK is the conventional and most common target.

Concrete example
SQL
CREATE TABLE users (
    user_id   BIGINT       NOT NULL,        -- identity column
    email     VARCHAR(255),                 -- may be null (phone signups)
    username  VARCHAR(50)  NOT NULL,
    phone     VARCHAR(20),

#### CONS
TRAINT pk_users PRIMARY KEY (user_id),      -- ONE, unique + not null

#### CONS
TRAINT uq_email UNIQUE (email),             -- unique, allows null

#### CONS
TRAINT uq_username UNIQUE (username)         -- another unique key
);

Here user_id is the canonical identity; email and username are two separate unique keys enforcing business rules. Another table references the identity:

SQL
CREATE TABLE orders (
    order_id BIGINT PRIMARY KEY,
    user_id  BIGINT NOT NULL,

#### CONS
TRAINT fk_user FOREIGN KEY (user_id) REFERENCES users(user_id)
);
Common misconceptions to correct
"A primary key is just a unique key that's not null." Functionally that's close — a PK behaves like UNIQUE NOT NULL — but semantically the PK is the row identifier and only one may exist, whereas unique keys are alternate keys and can be many. Some databases even implement PK internally as a unique index + not-null; still, the design intent differs.
"Unique columns can't have NULLs." They can (that's the whole point of the distinction), though multiple-NULL behaviour is engine-specific.
"A primary key must be a single column." No — a composite primary key spans several columns (e.g., (order_id, line_no) in an order-items table).
Natural vs surrogate keys. A related judgment call: use a surrogate PK (an auto-generated user_id) rather than a natural PK (email) so the identity is stable, small, and non-personal — while still enforcing the natural attribute via a unique key.
Summary / what a strong answer emphasizes
A primary key uniquely identifies each row and is unique + NOT NULL; a table has exactly one; it's the standard foreign-key target and often the clustered index.
A unique key also forbids duplicates but allows NULL and a table may have many; it enforces alternate business-uniqueness rules.
Lead with the two crisp differences — NULL allowed? and how many per table? — then mention the clustered-index and foreign-key roles for depth.

#### Follow-up
probes: "Can a foreign key reference a unique key?" (yes). "How many NULLs can a unique column hold?" (engine-specific — SQL Server: one; PostgreSQL/MySQL/Oracle: many). "Composite primary key example?" (order-items (order_id, product_id)). "Surrogate vs natural key trade-offs?"

**Comparison:**

```
Aspect	Primary Key	Unique Key
Enforces uniqueness	Yes	Yes
Allows NULL	No (NOT NULL)	Yes
Number allowed per table	Exactly one	Many
Purpose	Canonical row identity	Alternate business-uniqueness rule
Default index	Often clustered	Non-clustered (secondary)
Typical FK target	Yes (conventional)	Allowed
Can be composite	Yes	Yes
```

**Code:**

```sql
-- One primary key (unique + not null) plus multiple unique keys
CREATE TABLE users (
    user_id   BIGINT       NOT NULL,
    email     VARCHAR(255),                 -- nullable
    username  VARCHAR(50)  NOT NULL,
    CONSTRAINT pk_users   PRIMARY KEY (user_id),
    CONSTRAINT uq_email    UNIQUE (email),
    CONSTRAINT uq_username UNIQUE (username)
);

-- Composite primary key (spans two columns)
CREATE TABLE order_items (
    order_id   BIGINT NOT NULL,
    product_id BIGINT NOT NULL,
    qty        INT    NOT NULL,
    PRIMARY KEY (order_id, product_id)   -- unique combination, both not null
);

-- Foreign key referencing the primary key
CREATE TABLE orders (
    order_id BIGINT PRIMARY KEY,
    user_id  BIGINT NOT NULL REFERENCES users(user_id)
);
```

**Example input / output:**

```text
- `INSERT INTO users(user_id, email, username) VALUES (2, NULL, 'rohit');` → **succeeds** (unique key allows NULL email).
- `INSERT INTO users(user_id, email, username) VALUES (NULL, 'x@y.com', 'x');` → **fails** (primary key cannot be NULL).
- `INSERT INTO users(user_id, email, username) VALUES (1, 'dup@y.com', 'dup');` → **fails** (duplicate primary key 1).
- `INSERT INTO users(user_id, email, username) VALUES (4, 'asha@example.com', 'asha2');` → **fails** (duplicate unique email).
```

**Data:**

```text
users
+---------+---------------------+----------+
| user_id | email               | username |
+---------+---------------------+----------+
| 1       | asha@example.com    | asha     |
| 2       | NULL                | rohit    |   -- email null: OK for UNIQUE, NOT ok for PK
| 3       | NULL                | meera    |   -- second null email: OK in PG/MySQL/Oracle
| 3 (dup) | ...                              |   -- REJECTED: duplicate primary key
+---------+---------------------+----------+
```

**Flow:**

1. Yes
2. No, alternate rule, may benull
3. Need to enforce uniquenesson a column set?
4. Is it the row's identityand never null?
5. PRIMARY KEYone per table, unique + notnull
6. UNIQUE KEYmany per table, allows null
7. Referenced by foreign keys

