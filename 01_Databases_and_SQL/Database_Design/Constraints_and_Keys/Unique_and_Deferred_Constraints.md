---
id: "01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Unique_and_Deferred_Constraints"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Design/Constraints_and_Keys"
tools: []
---

# Unique and Deferred Constraints

**L4 Advanced** · [↑ Constraints and Keys](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A UNIQUE constraint guarantees that no two rows share the same value (or the same combination of values) in the constrained columns — and the details that trip people up are how NULLs count, what index backs it, and when the check actually runs.

### 1. The big picture
A primary key identifies a row. But real tables usually have *other* columns that must also never repeat: a username, an email, a phone number, a national ID, a (tenant, slug) pair. These are **alternate keys**, and the way to make the database enforce them is a UNIQUE constraint. Without it, uniqueness lives only in application code, and application code races: two simultaneous sign-ups both check "is this email free?", both see yes, both insert.

This lesson sits under *Constraints and Keys* next to Primary and Unique Keys (the basics) and Foreign Keys. Here we go one level deeper: composite uniqueness, NULL semantics across engines, conditional (filtered/partial) uniqueness, case-insensitive uniqueness, adding a constraint to a live table without an outage, and *deferred* checking — validating at COMMIT instead of per statement.

### 2. Core concepts and vocabulary
- **Unique constraint** — a declared rule that the values of one or more columns must be distinct across all rows. The database rejects any INSERT/UPDATE that would create a duplicate.
- **Alternate key** — a candidate key that was not chosen as the primary key. Enforced with UNIQUE.
- **Composite unique constraint** — uniqueness over a *combination* of columns, e.g. `UNIQUE (tenant_id, slug)`. Each column alone may repeat; the pair may not.
- **Unique index** — the physical structure (almost always a B-tree) that engines use to check for duplicates quickly. A constraint is the *logical rule*; the index is the *mechanism*.
- **Partial / filtered unique index** — a unique index that only covers rows matching a predicate (`WHERE is_active = 1`). It enforces "unique among these rows only".
- **Functional (expression) unique index** — unique over a computed value such as `lower(email)`.
- **Deferrable constraint** — a constraint whose check can be postponed to the end of the transaction (PostgreSQL, Oracle). `INITIALLY IMMEDIATE` checks per statement; `INITIALLY DEFERRED` checks at COMMIT.
- **Collation** — the rules for comparing and sorting strings (case, accents). Uniqueness is decided by the collation: 'a' and 'A' are duplicates under a case-insensitive collation.

### 3. How it works, step by step
1. You declare the constraint (inline, named, or via ALTER TABLE).
2. The engine creates (or reuses) a unique index on those columns. This is why a UNIQUE constraint is *not free*: every INSERT and every UPDATE touching those columns does an index lookup and an index write.
3. On INSERT/UPDATE, the engine seeks the new key value in the index. If it finds an equal key, the statement fails with a duplicate-key error (SQL Server 2627/2601, PostgreSQL `23505 unique_violation`, MySQL 1062).
4. If the value is NULL, the NULL rules of the engine apply (below).
5. Concurrency: the engine must serialize two sessions inserting the same new key. In practice the second one blocks on the first's index entry until it commits (then fails) or rolls back (then succeeds). That is why uniqueness is race-free when enforced by the database and racy when enforced by "SELECT then INSERT".

**NULL behaviour — the single most asked detail**

| Engine | Single-column UNIQUE | Notes |
|---|---|---|
| SQL Server | **one** NULL allowed (NULLs treated as equal) | use a filtered unique index `WHERE col IS NOT NULL` to allow many |
| PostgreSQL | **many** NULLs allowed (NULL ≠ NULL) | PG 15+: `UNIQUE NULLS NOT DISTINCT` gives SQL-Server-like behaviour |
| MySQL/InnoDB, MariaDB | many NULLs allowed | |
| Oracle | many NULLs allowed on single column; for a composite key, rows with *all* columns NULL are exempt, rows with some NULLs are compared on the non-null parts | |

The SQL standard says a UNIQUE constraint treats NULLs as "not equal" (so many allowed). SQL Server is the notable outlier. The common wrong belief "UNIQUE allows exactly one NULL" is true only for SQL Server.

### 4. Worked example
```sql
-- PostgreSQL
CREATE TABLE app_user (
  user_id   bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  email     text NOT NULL,
  username  text NOT NULL,
  phone     text,
  CONSTRAINT unq_username UNIQUE (username),
  CONSTRAINT unq_phone    UNIQUE (phone)          -- many NULL phones are fine
);

-- Case-insensitive email, original casing preserved for display:
CREATE UNIQUE INDEX unq_email_ci ON app_user (lower(email));

INSERT INTO app_user(email, username) VALUES ('Ann@Example.com', 'ann');
INSERT INTO app_user(email, username) VALUES ('ann@example.com', 'ann2');
-- ERROR: duplicate key value violates unique constraint "unq_email_ci"
```
What happened: the stored value keeps `Ann@Example.com`, but the index key is `ann@example.com`, so the second insert collides. Queries must use the same expression (`WHERE lower(email) = lower($1)`) to use the index.

**Conditional uniqueness — one active subscription per user:**
```sql
-- PostgreSQL / SQLite
CREATE UNIQUE INDEX one_active_sub ON subscription (user_id) WHERE status = 'active';
-- SQL Server
CREATE UNIQUE INDEX one_active_sub ON dbo.subscription (user_id) WHERE status = 'active';
```
Many cancelled rows per user are allowed; at most one active row. Same trick solves **soft delete** (`UNIQUE (email) WHERE deleted_at IS NULL`) so a new account can reuse an email after the old row was soft-deleted. MySQL has no partial indexes; emulate with a generated column that is NULL for non-active rows and put UNIQUE on it (NULLs don't collide).

**Deferred check — swapping two values:**
```sql
CREATE TABLE seat (
  row_no int, seat_no int, passenger text,
  CONSTRAINT unq_seat UNIQUE (row_no, seat_no) DEFERRABLE INITIALLY IMMEDIATE
);
BEGIN;
SET CONSTRAINTS unq_seat DEFERRED;
UPDATE seat SET seat_no = 2 WHERE seat_no = 1;   -- temporarily duplicate; no error yet
UPDATE seat SET seat_no = 1 WHERE seat_no = 2 AND passenger = 'B';
COMMIT;                                          -- checked now; passes if no duplicate remains
```
Without deferral the first UPDATE would fail even though the *final* state is valid. If the final state still has a duplicate, the error appears at COMMIT, far from the statement that caused it — log the constraint name so the application can map it to a meaningful message.

### 5. Under the hood
- **Index first.** In SQL Server a UNIQUE constraint *is* a unique nonclustered index (the constraint metadata sits on top). In PostgreSQL the constraint creates a unique B-tree index automatically; you can see it in `pg_indexes`. Dropping the constraint drops the index. So "does UNIQUE create an index?" — yes in every mainstream OLTP engine; the misconception in the question bank is the opposite. Data warehouses (Snowflake, BigQuery, Redshift) accept the syntax but **do not enforce** it.
- **How the duplicate is detected.** The B-tree is descended to the leaf where the key would live; if an equal key (visible or in-flight) is there, the insert errors or waits. In PostgreSQL's MVCC, dead row versions can still hold index entries, so the check also consults heap visibility.
- **Deferrable cost.** PostgreSQL non-deferrable unique constraints are checked immediately per *row* (not per statement as the standard says), which is why `UPDATE t SET id = id + 1` can fail transiently. Deferrable ones queue "recheck" events per row and process them at commit — extra memory and overhead on big batches.
- **Engine support for deferral:** PostgreSQL and Oracle yes; SQL Server and MySQL no (workarounds: temporary placeholder values, or a single set-based UPDATE that SQL Server evaluates as a whole).
- **Building on a live table.** `CREATE UNIQUE INDEX CONCURRENTLY` (PostgreSQL) then `ALTER TABLE ... ADD CONSTRAINT ... UNIQUE USING INDEX idx` attaches it with a very short lock. SQL Server: `CREATE UNIQUE INDEX ... WITH (ONLINE = ON)` (Enterprise/Azure). A failed concurrent build in PostgreSQL leaves an **INVALID** index behind: drop it and retry.

### 6. Performance and design implications
- Each unique constraint is a full index: extra write cost, extra storage, extra buffer-pool pressure. Add them where the business rule demands, not "just in case".
- Column order of a composite unique index matters for *reads*: `(tenant_id, slug)` also serves queries by `tenant_id` alone, but not by `slug` alone.
- Wide keys (long strings) make fat indexes; consider a hash column only if you also handle collisions.
- Unique index = optimizer information: it knows at most one row matches, which improves cardinality estimates and enables join elimination in some engines.
- Heavy contention: many sessions inserting the same key serialize on that index entry. Idempotency keys rely on exactly this — the second request fails fast with a duplicate error you handle as "already processed".
- Diagnosing "duplicates that shouldn't exist": check NULLs in a composite key, collation differences (accents, trailing spaces, `ß`/`ss` under some collations), and whether the "unique" column is actually in a warehouse that doesn't enforce.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ALTER TABLE t ADD CONSTRAINT c UNIQUE (a,b)` | add a named unique constraint | after cleaning duplicates |
| `ALTER TABLE t DROP CONSTRAINT c` | remove it (and its index) | MySQL uses `DROP INDEX c` |
| `CREATE UNIQUE INDEX ... WHERE ...` | partial/filtered uniqueness | soft delete, "one active row" |
| `CREATE UNIQUE INDEX CONCURRENTLY` (PG) | non-blocking build | adding uniqueness in production |
| `ADD CONSTRAINT ... UNIQUE USING INDEX` (PG) | promote an existing index | finish the concurrent pattern |
| `DEFERRABLE INITIALLY DEFERRED` / `SET CONSTRAINTS ... DEFERRED` | postpone checks to COMMIT | row swaps, bulk reorders |
| `UNIQUE NULLS NOT DISTINCT` (PG 15+) | treat NULLs as equal | "only one NULL" rule |
| `INSERT ... ON CONFLICT` / `MERGE` | handle duplicates deliberately | upserts, idempotent loads |
| `EXCLUDE USING gist (...)` (PG) | exclusion constraint | no overlapping date ranges per key |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Plain UNIQUE on a natural alternate key | the "natural" key can legitimately change or be reused (phone numbers, emails) — it will fight you |
| Composite UNIQUE `(tenant_id, x)` for multi-tenant isolation | `tenant_id` is nullable — NULL rows silently escape the rule |
| Functional index `lower(email)` | queries don't use the same expression, so the index isn't used |
| `citext` (PG) for case-insensitive text | you need portability or a different collation per query |
| Partial unique index for conditional rules | the engine lacks them (MySQL) — use a generated column |
| Deferrable constraint | you only need it to hide a design flaw; also not available in SQL Server |
| dbt `unique` test / dedup in warehouses | you need real-time protection; tests only report after the fact |

### 9. Common mistakes
- Believing UNIQUE allows exactly one NULL everywhere → on PostgreSQL/MySQL you get many NULL duplicates → use `NULLS NOT DISTINCT`, a `NOT NULL` column, or a sentinel with a CHECK.
- Multi-column unique with a nullable column → "duplicates" with a NULL pass → make the column NOT NULL or add a partial index for the NULL case.
- Checking uniqueness in application code only → race conditions → always back it with a constraint and catch the violation.
- Upsert that swallows the violation (`ON CONFLICT DO NOTHING`) → silent lost writes → use `DO UPDATE` or check affected-row counts, and log conflicts.
- Adding the constraint with a blocking `ALTER` on a huge table → long lock → build the index concurrently/online first.
- Forgetting soft-deleted rows → "email already exists" for users who deleted accounts → partial unique index on non-deleted rows.
- Assuming a warehouse enforces UNIQUE → duplicates load fine → enforce with MERGE/dedup or tests.

### 10. 🎯 Interview kit
**30-second answer:** A UNIQUE constraint enforces that a column or combination of columns has no duplicates, backed by a unique index. Unlike a primary key it may be nullable; how many NULLs it allows depends on the engine (SQL Server one, PostgreSQL/MySQL many).

**Structure a longer answer:** clarify the engine and the business rule → explain constraint vs index and NULL semantics → cover composite and conditional variants → mention operational concerns (concurrent build, deferred checks, error handling) → close with a scenario such as soft-delete emails.

**Likely follow-ups:**
- *Does UNIQUE create an index?* → Yes in OLTP engines; the index is the enforcement mechanism and has write cost.
- *Composite unique vs two single-column uniques?* → Pair must be unique vs each column independently unique — very different strictness.
- *Only one active subscription per user?* → Partial/filtered unique index on `user_id WHERE status='active'`.
- *Add UNIQUE on a production table that failed midway?* → Find duplicates, fix them, `CREATE UNIQUE INDEX CONCURRENTLY`, drop any INVALID leftover, attach with `USING INDEX`.
- *Case-insensitive email, keep casing?* → Functional index on `lower(email)` (or citext/CI collation); trade-off is query-expression matching.
- *Deferred unique violation?* → Raised at COMMIT; capture the constraint name to give a useful message.

**Signals interviewers look for:** you separate the logical rule from the index, know NULLs differ by engine, and think about concurrency and migrations, not just syntax.

### 11. Check yourself
- [ ] State how many NULLs a single-column UNIQUE allows in SQL Server, PostgreSQL and Oracle.
- [ ] Write a unique rule for "one active row per user" in your engine.
- [ ] Explain why `lower(email)` unique index needs matching query expressions.
- [ ] Describe the safe steps to add a UNIQUE constraint to a 500M-row production table.
- [ ] Give an example where deferral is needed and say what happens if the final state is still duplicated.
- [ ] Explain why a warehouse "unique" column can still contain duplicates.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is a UNIQUE constraint and how many NULL values can a UNIQUE column allow?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10046_What_is_a_UNIQUE_constraint_and_how_many_NULL_values_can_a_U.md>)
- [Basic] [How do you drop a UNIQUE constraint from a table?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10229_How_do_you_drop_a_UNIQUE_constraint_from_a_table.md>)
- [Intermediate] [A UNIQUE index rebuild during an online DDL failed and left a duplicate. How do you find and resolve duplicate keys before re-adding the constraint?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10135_A_UNIQUE_index_rebuild_during_an_online_DDL_failed_and_left_.md>)
- [Deep] [How would you enforce a conditional uniqueness rule, e.g. only one 'active' subscription per user, using a partial/filtered unique index?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10050_How_would_you_enforce_a_conditional_uniqueness_rule_e_g_only.md>)
- [Deep] [How would you design keys and constraints for a soft-delete pattern so that a UNIQUE(email) still allows a new active row after deletion?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10081_How_would_you_design_keys_and_constraints_for_a_soft-delete_.md>)
- [Deep] [How do idempotency keys map to unique constraints in a payments API, and what happens on a duplicate request that hits the constraint?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10094_How_do_idempotency_keys_map_to_unique_constraints_in_a_payme.md>)
- [Deep] [How do you handle a UNIQUE constraint on a column that must be unique only within a soft-partition (per-year) — partial index, generated column, or partitioned unique?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10109_How_do_you_handle_a_UNIQUE_constraint_on_a_column_that_must_.md>)
- [Deep] [Explain how a violated deferred unique constraint reports at COMMIT time, and how you'd surface a meaningful error to the application in that case.](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10143_Explain_how_a_violated_deferred_unique_constraint_reports_at.md>)

## Related concepts

- [Unique Constraints for Dedup](<../../SQL_Language/SQL_Fundamentals/Unique_Constraints_for_Dedup.md>) *(SQL Fundamentals)*
- [Adding Unique Indexes Safely](<../../Query_Performance/Indexing/Adding_Unique_Indexes_Safely.md>) *(Indexing)*
- [Adding Constraints Online](<../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Adding_Constraints_Online.md>) *(PostgreSQL Platform)*

## References

- *Grokking Relational Database Design* — Qiang Hao, Michail Tsikerdekis — Implementation
- *Practical Graph Structures in SQL Server and Azure SQL* — Louis Davidson — SQL Graph Tables Extended Topics › Uniqueness Constraints (and Indexes)
- *Designing Data-Intensive Applications* — Martin Kleppmann — Relying on Linearizability › Constraints and uniqueness guarantees
- *Designing Data-Intensive Applications* — Martin Kleppmann — Enforcing Constraints › Uniqueness constraints require consensus
- *Designing Data-Intensive Applications* — Martin Kleppmann — Enforcing Constraints › Loosely interpreted constraints
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — 13. Implementing Logic in the Database › Adding UNIQUE Constraints
- *Data Engineering Design Patterns* — Bartosz Konieczny — Idempotency Design Patterns › Consequences
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Default Values
- *SQL Server Tutorial* (YouTube series) — Part 9: Unique key constraint ([watch](https://www.youtube.com/watch?v=oqrsfatxTYE))

