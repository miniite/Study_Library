---
id: "TOOLS/MySQL/MySQL_Platform/Constraint_Definitions"
level: 4
type: "concept"
topic: "TOOLS/MySQL/MySQL_Platform"
tools: ["MySQL"]
---

# Constraint Definitions

**L4 Advanced** · [↑ MySQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Constraints (`NOT NULL`, `DEFAULT`, `UNSIGNED`, `CHECK`, `PRIMARY KEY`, `UNIQUE`, `FOREIGN KEY`) let the database itself reject bad data — in MySQL/InnoDB each one also has storage, indexing and locking consequences you should know before choosing it.

### 1. The big picture
Application code can validate data, but only the database can guarantee that *every* writer — the web app, the batch job, the person with a console — obeys the same rules. Constraints are those rules declared in the schema. They sit between data-type choice (what *kind* of value) and normalization (how to split facts across tables): types limit the shape of a value, constraints limit which values and relationships are valid, and normalization decides where the facts live.

In MySQL there are a few details that differ from the textbook. `CHECK` constraints were parsed but ignored until 8.0.16. A `REFERENCES` clause written inline on a column is parsed and ignored — only a table-level `FOREIGN KEY` is enforced. Foreign keys exist only in InnoDB. A `UNIQUE` constraint *is* a unique index. The primary key is the clustered index, so choosing it is a physical design decision, not just a logical one.

This lesson covers the definitions; the keys-and-relationships deep dive lives in the Database Design lessons.

### 2. Core concepts and vocabulary
- **Domain constraint** — restricts values of one column: type, `NOT NULL`, `DEFAULT`, `UNSIGNED`, `CHECK`.
- **Entity integrity** — every row is identifiable: the primary key is unique and non-null.
- **Referential integrity** — a foreign-key value must match an existing parent key (or be `NULL`); no orphans.
- **Candidate key** — any minimal column set that uniquely identifies a row; the one you pick is the **primary key**, the rest become `UNIQUE`.
- **Natural key** — a real-world attribute (ISBN, email, country code). **Surrogate key** — an artificial identifier (`AUTO_INCREMENT`, UUID) with no business meaning.
- **Composite key** — a key made of several columns (e.g., `(order_id, line_no)`).
- **Column-level vs table-level constraint** — written next to one column vs as a separate clause; table-level is required for multi-column constraints and for foreign keys in MySQL.
- **Functional dependency** — `A → B`: the value of A determines B. Normal forms are rules about which dependencies may exist in a table.
- **Anomaly** — insert/update/delete problems caused by redundancy.

### 3. How it works, step by step
1. **Declare** the constraint in `CREATE TABLE` or `ALTER TABLE`.
2. **On every `INSERT`/`UPDATE`/`DELETE`** InnoDB checks it row by row, immediately — MySQL does not support deferred constraints (statement fails, and in a transaction the statement is rolled back, not necessarily the whole transaction).
3. **Violations** raise an error (e.g., 1062 duplicate entry, 1048 column cannot be null, 1452 foreign key fails, 3819 check constraint violated). `INSERT IGNORE` converts some into warnings — dangerous because it also silently coerces values.
4. **Indexes are created or required**: a `PRIMARY KEY` and each `UNIQUE` build a B+tree; a foreign key needs an index on the child column (MySQL creates one if absent) and an index starting with the referenced column on the parent.
5. **Locks**: a foreign-key check takes a shared lock on the parent index record; inserting many children for the same parent therefore contends with updates of that parent.
6. **Referential actions** run when a parent row changes: `RESTRICT`/`NO ACTION` (reject; same in MySQL), `CASCADE`, `SET NULL`. `SET DEFAULT` is parsed but rejected by InnoDB.

### 4. Worked example
```sql
CREATE TABLE customer (
  customer_id  BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
  email        VARCHAR(190)    NOT NULL,
  age          TINYINT UNSIGNED NULL,
  status       ENUM('active','blocked') NOT NULL DEFAULT 'active',
  created_at   DATETIME        NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (customer_id),
  UNIQUE KEY uq_customer_email (email),
  CONSTRAINT ck_customer_age CHECK (age IS NULL OR age BETWEEN 0 AND 150)
) ENGINE=InnoDB;

CREATE TABLE orders (
  order_id     BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
  customer_id  BIGINT UNSIGNED NOT NULL,
  total        DECIMAL(12,2)   NOT NULL,
  PRIMARY KEY (order_id),
  KEY ix_orders_customer (customer_id),
  CONSTRAINT fk_orders_customer FOREIGN KEY (customer_id)
      REFERENCES customer (customer_id)
      ON DELETE RESTRICT ON UPDATE CASCADE,
  CONSTRAINT ck_orders_total CHECK (total >= 0)
) ENGINE=InnoDB;

INSERT INTO orders (customer_id, total) VALUES (999, 10.00);
-- ERROR 1452: Cannot add or update a child row: a foreign key constraint fails
DELETE FROM customer WHERE customer_id = 1;   -- fails while orders exist (RESTRICT)
UPDATE customer SET age = -5 WHERE customer_id = 1;  -- ERROR 3819 (or 1264 for UNSIGNED)
```
Reading it: `age` is protected twice — `TINYINT UNSIGNED` (0–255) bounds the storage, the `CHECK` narrows the business range. The `REFERENCES` is a table-level clause, so it is enforced. Deleting a parent with children is blocked; with `ON DELETE CASCADE` the children would vanish and — important — the cascade does **not** fire triggers on the child table.

Case-insensitive uniqueness of email works because the default collation is `_ci`; to be explicit and safe across collations, add a normalized column:
```sql
ALTER TABLE customer
  ADD COLUMN email_norm VARCHAR(190) GENERATED ALWAYS AS (LOWER(TRIM(email))) STORED,
  ADD UNIQUE KEY uq_customer_email_norm (email_norm);
```

### 5. Under the hood
- **Primary key = clustered index.** InnoDB stores the whole row in the PK B+tree. With no PK, it uses the first `UNIQUE` index whose columns are all `NOT NULL`; otherwise it adds a hidden 6-byte row ID. Every secondary index leaf holds the PK value, so a wide PK (a 36-char UUID, a 4-column composite) bloats every index.
- **Auto-increment.** Counter is kept per table; values are not reused after rollback or failed inserts, so gaps are normal. In 8.0 the counter is persisted in the redo log (earlier versions recomputed `MAX+1` on restart). `TRUNCATE` resets it; `DELETE` does not.
- **UNIQUE and NULL.** A unique index permits multiple `NULL`s, because `NULL = NULL` is unknown. If "at most one row per X, with X optional" is the rule, that works; if "exactly one" is the rule, add `NOT NULL`.
- **Unique constraint vs unique index.** In MySQL `UNIQUE KEY` and `CREATE UNIQUE INDEX` are the same object in the data dictionary; there is no cheaper "constraint-only" mode.
- **CHECK.** Stored in the data dictionary, evaluated on insert/update; must be deterministic (no subqueries, no non-deterministic functions, no references to auto-increment columns). `NOT ENFORCED` can mark it informational.
- **Foreign key checks** happen inside InnoDB (not the SQL layer), so cascades are invisible to the binary log as separate statements and do not activate triggers; `SET foreign_key_checks = 0` disables checks per session — useful for bulk loads, and a way to create orphans.
- **UNSIGNED arithmetic.** `UNSIGNED - UNSIGNED` that would go negative raises `BIGINT UNSIGNED value is out of range` unless `NO_UNSIGNED_SUBTRACTION` SQL mode is set; cast to `SIGNED` first.

### 6. Performance and design implications
- Every constraint is a write-time cost: unique and FK checks add index lookups and locks. They are almost always worth it, but know the price on hot paths (bulk loads are the classic place to disable and re-validate).
- Choose a short, monotonically increasing PK for insert-heavy tables (append at the right edge of the B+tree, no page splits in the middle); random UUIDv4 keys fragment the clustered index. If you need UUIDs, store as `BINARY(16)` and consider time-ordered variants (`UUID_TO_BIN(uuid, 1)`).
- Always index FK child columns for join and delete performance — otherwise deleting a parent scans the child table while holding locks.
- `INT UNSIGNED` doubles the positive range (≈4.29 billion) — an `AUTO_INCREMENT` on a signed `INT` overflows at 2,147,483,647 and then every insert fails with duplicate-key 1062 on the max value. Use `BIGINT UNSIGNED` for high-volume tables.
- Normalize first, denormalize with evidence: 1NF (atomic values, no repeating groups), 2NF (no partial dependency on part of a composite key), 3NF (no transitive dependency — non-key attributes depend only on the key).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `NOT NULL`, `DEFAULT expr` | Reject `NULL`, supply a value when omitted | Almost every column |
| `UNSIGNED` | Non-negative integers, doubled upper range | Counters, IDs, quantities |
| `CHECK (expr)` (8.0.16+) | Enforce a row-level predicate | Ranges, enum-like rules, cross-column rules |
| `PRIMARY KEY (cols)` | Unique, non-null, clustered | One per table |
| `UNIQUE KEY name (cols)` | Unique index (NULLs allowed) | Natural keys, idempotency keys |
| `FOREIGN KEY … ON DELETE/UPDATE` | Referential integrity and cascade | Parent/child tables in InnoDB |
| `SHOW CREATE TABLE t` | See all constraints | Audit |
| `information_schema.TABLE_CONSTRAINTS`, `CHECK_CONSTRAINTS`, `REFERENTIAL_CONSTRAINTS` | Inventory | Review, tooling |
| `ALTER TABLE t DROP CHECK/FOREIGN KEY/INDEX name` | Remove | Migrations |
| `SET foreign_key_checks=0/1` | Skip FK validation in a session | Controlled bulk loads, then verify |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Surrogate `AUTO_INCREMENT` PK + `UNIQUE` on the natural key | Distributed writers need globally unique IDs without a coordinator (use UUIDv7/ULID/snowflake) |
| Natural key as PK (country code, ISO currency) | Value can change or is long — it is copied into every secondary index and FK |
| `ON DELETE CASCADE` for true ownership (order → order lines) | Cascading across many levels or large sets: huge locks, no triggers, surprising deletions |
| `ON DELETE RESTRICT` for shared reference data | Teams that then do manual cleanup in the wrong order |
| `CHECK` for simple invariants | Rules needing other rows (use FK/trigger/app) or MySQL < 8.0.16 |
| Foreign keys everywhere | Sharded/very high write-throughput systems that enforce integrity in the app instead |

### 9. Common mistakes
- **Writing inline `col INT REFERENCES parent(id)`** → MySQL silently ignores it → no integrity → use a table-level `FOREIGN KEY`.
- **Relying on `CHECK` in 5.7** → never enforced → upgrade or use triggers.
- **Mismatched FK column types** (`INT` vs `BIGINT UNSIGNED`, different charset) → error 3780/1215 → make types identical.
- **`UNIQUE` on a nullable column expecting one `NULL`** → many `NULL`s allowed → add `NOT NULL` or a sentinel.
- **Composite PK where a single-column surrogate would do** → fat secondary indexes and awkward joins → surrogate PK + unique composite.
- **Forgetting that `DEFAULT` is only used when the column is omitted** → explicit `NULL` into `NOT NULL` fails (error 1048) in strict mode; omit the column or use `DEFAULT`.
- **Signed `INT` auto-increment on a busy table** → overflow in production → `BIGINT UNSIGNED`.
- **Trusting `INSERT IGNORE`** → hides FK/constraint violations as warnings → use explicit `ON DUPLICATE KEY UPDATE` and handle errors.

### 10. 🎯 Interview kit
**30-second answer:** Constraints enforce integrity in the database: `NOT NULL`/`DEFAULT`/`CHECK` for column values, `PRIMARY KEY` and `UNIQUE` for identity, `FOREIGN KEY` for relationships. In InnoDB the PK is the clustered index, `UNIQUE` is a unique index, and FKs need indexes and take locks.

**Structure a longer answer:** define each kind → how MySQL enforces it (immediately, via indexes) → MySQL-specific gotchas (inline REFERENCES ignored, CHECK from 8.0.16, FK actions) → trade-offs (write cost, cascades) → a concrete scenario.

**Likely follow-ups:**
- *Primary key vs unique key?* → One PK per table, never `NULL`, becomes the clustered index; many unique keys, `NULL`s allowed.
- *What does a primary key create in InnoDB?* → The clustered index containing the full rows; secondary indexes reference it.
- *CASCADE vs SET NULL vs RESTRICT?* → delete children / null the FK (column must be nullable) / refuse the parent change.
- *Natural vs surrogate key?* → Business meaning and stability vs compact, immutable, join-friendly.
- *How do you make email unique case-insensitively?* → `_ci` collation, or a normalized generated column with a unique index.
- *Explain 1NF/2NF/3NF.* → atomic values; no partial dependency; no transitive dependency.

**Signals interviewers look for:** you mention MySQL's inline-REFERENCES and CHECK history, you connect keys to clustered-index physical design, and you know that cascades bypass triggers.

### 11. Check yourself
- [ ] Write a table with PK, unique natural key, FK with `ON DELETE RESTRICT`, a `CHECK`, and a default.
- [ ] Explain what InnoDB does when no primary key exists.
- [ ] Show why `UNIQUE (a)` allows two `NULL` rows and how you would forbid it.
- [ ] Give the max value of `TINYINT`, `SMALLINT`, `INT` signed and unsigned.
- [ ] Describe an insertion, an update and a deletion anomaly and the normal form that removes each.
- [ ] Explain why a child-table FK column needs an index.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../_index.md>)

## Practice questions

- [Basic] [What is a NOT NULL constraint and when would you apply it to a column?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33356_What_is_a_NOT_NULL_constraint_and_when_would_you_apply_it_to.md>)
- [Basic] [What is a candidate key and how does it relate to the primary key you finally choose?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33382_What_is_a_candidate_key_and_how_does_it_relate_to_the_primar.md>)
- [Basic] [What happens if you try to insert a foreign key value that does not exist in the referenced table?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33386_What_happens_if_you_try_to_insert_a_foreign_key_value_that_d.md>)
- [Basic] [What does the SIGNED versus UNSIGNED attribute on an integer column change about its range?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33008_What_does_the_SIGNED_versus_UNSIGNED_attribute_on_an_integer.md>)
- [Intermediate] [What is the difference between ON DELETE CASCADE, ON DELETE SET NULL, and ON DELETE RESTRICT?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33387_What_is_the_difference_between_ON_DELETE_CASCADE_ON_DELETE_S.md>)
- [Intermediate] [What is a partial dependency and how does moving to 2NF remove it?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33394_What_is_a_partial_dependency_and_how_does_moving_to_2NF_remo.md>)
- [Intermediate] [What is a primary key constraint automatically creating behind the scenes in InnoDB?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33434_What_is_a_primary_key_constraint_automatically_creating_behi.md>)
- [Intermediate] [What is normalization and can you explain 1NF, 2NF, and 3NF briefly?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33047_What_is_normalization_and_can_you_explain_1NF_2NF_and_3NF_br.md>)

## Related concepts

- [Normal Forms](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Normal_Forms.md>) *(DBMS Concepts)*
- [Foreign Keys and Referential Integrity](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Foreign_Keys_and_Referential_Integrity.md>) *(SQL Fundamentals)*
- [Surrogate Keys in Normalization](<../../../01_Databases_and_SQL/Database_Design/Normalization/Surrogate_Keys_in_Normalization.md>) *(Normalization)*

