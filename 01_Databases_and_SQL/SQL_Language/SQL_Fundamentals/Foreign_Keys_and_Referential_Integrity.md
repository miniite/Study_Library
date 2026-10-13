---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Foreign_Keys_and_Referential_Integrity"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: ["MySQL"]
---

# Foreign Keys and Referential Integrity

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Keys define identity (primary/candidate/surrogate) and a foreign key makes the database refuse a child row whose parent doesn't exist — referential integrity is that guarantee, and where it's missing (warehouses) you check it with orphan anti-joins.

### 1. The big picture
A relational model is a set of tables related by keys. Two questions come up constantly: *how do I identify a row?* (keys) and *how do I keep related rows consistent?* (foreign keys). T-SQL Fundamentals puts it plainly: relational integrity is enforced by constraints, and the engine rejects statements that would violate them.

In OLTP systems you declare the constraints and the engine enforces them. In analytic warehouses constraints are often absent or not enforced (loading speed, distributed engines), so integrity becomes a *pipeline responsibility* verified by queries. This lesson covers both: the vocabulary of keys, how FK enforcement behaves (including why a "correct looking" insert can fail), surrogate-vs-natural choices, and validation queries.

### 2. Core concepts and vocabulary
- **Super key** — any set of columns that uniquely identifies a row (may include extra columns).
- **Candidate key** — a *minimal* super key (no column can be removed).
- **Primary key (PK)** — the chosen candidate key; unique and `NOT NULL`; one per table.
- **Alternate key** — other candidate keys, enforced with `UNIQUE`.
- **Composite key** — a key made of more than one column.
- **Natural key** — meaningful business value (email, ISBN, SSN).
- **Surrogate key** — system-generated meaningless value (identity, sequence, GUID).
- **Foreign key (FK)** — column(s) in a child referencing a parent's PK/unique key.
- **Orphan** — a child row whose parent doesn't exist.
- **Referential action** — what happens on parent delete/update: `NO ACTION`, `CASCADE`, `SET NULL`, `SET DEFAULT`.
- **Self-referencing FK** — a table pointing to itself (`employee.manager_id → employee.id`).

### 3. How it works, step by step
**Insert into a child:**
1. The engine checks the FK columns against the parent's unique index (a seek).
2. If a parent row exists (or the FK is `NULL`), the insert succeeds.
3. Otherwise it fails with a constraint-violation error and the statement rolls back.

**Delete/update on a parent:**
1. Engine looks for child rows referencing the old key (needs a seek on the child's FK column — an index there matters).
2. Depending on the action: reject (`NO ACTION`), delete children (`CASCADE`), null them (`SET NULL`).

**Why a child insert fails although data "looks correct":** the parent row isn't *visible yet* — it's in an uncommitted transaction, inserted later in the same batch, in another environment, differs by case/trailing space/collation/data type, or the load order put children first. A `NULL` FK always passes, so partially-null composite FKs behave surprisingly under default `MATCH SIMPLE` semantics.

### 4. Worked example
```sql
CREATE TABLE dbo.customer (
    customer_id int IDENTITY(1,1) CONSTRAINT PK_customer PRIMARY KEY,   -- surrogate
    email       nvarchar(255) NOT NULL CONSTRAINT UQ_customer_email UNIQUE   -- natural alternate key
);
CREATE TABLE dbo.orders (
    order_id    int IDENTITY(1,1) CONSTRAINT PK_orders PRIMARY KEY,
    customer_id int NOT NULL
        CONSTRAINT FK_orders_customer REFERENCES dbo.customer(customer_id)
        ON DELETE NO ACTION,
    order_date  date NOT NULL
);
CREATE INDEX IX_orders_customer ON dbo.orders(customer_id);   -- FK columns are NOT indexed automatically

-- Fails: parent missing
INSERT dbo.orders(customer_id, order_date) VALUES (999, '2026-10-01');
-- Msg 547: The INSERT statement conflicted with the FOREIGN KEY constraint "FK_orders_customer"

-- Orphan detection in a warehouse without FKs
SELECT f.order_id, f.customer_id
FROM   dw.fact_orders f
LEFT JOIN dw.dim_customer d ON d.customer_id = f.customer_id
WHERE  d.customer_id IS NULL;

-- Self-referencing FK cycle (employee → manager loops) at read time
WITH chain AS (
    SELECT emp_id, manager_id, CAST(emp_id AS varchar(max)) AS path, 0 AS depth
    FROM dbo.employee
    UNION ALL
    SELECT c.emp_id, e.manager_id, c.path + '>' + CAST(e.emp_id AS varchar(10)), c.depth + 1
    FROM chain c JOIN dbo.employee e ON e.emp_id = c.manager_id
    WHERE c.depth < 50 AND CHARINDEX('>' + CAST(e.emp_id AS varchar(10)), c.path) = 0
)
SELECT DISTINCT emp_id FROM chain WHERE depth >= 50;   -- very deep or cyclic starts
```
Reading it: the `IX_orders_customer` index makes both the FK check on parent deletes and joins from customer → orders a seek; without it, deleting a customer scans `orders`. The recursion carries the visited path to stop on cycles.

### 5. Under the hood (optional at L4)
A PK/UNIQUE constraint is implemented as a **unique index** (the PK is clustered by default in SQL Server). An FK has *no* index of its own on the child side, which is why the guidance "index every foreign key" exists. Constraints created `WITH NOCHECK` or disabled are marked **not trusted**: the optimizer can't use them for join elimination and rewrites, and you should re-enable `WITH CHECK CHECK CONSTRAINT`. Surrogate keys are narrow (4–8 bytes) and every non-clustered index carries the clustering key, so a short ever-increasing clustered key keeps all indexes small and inserts append-only; a random GUID clustered key causes page splits and fragmentation. Cascading actions run as part of the same statement and can lock many child rows.

### 6. Performance and design implications
- Index child FK columns (leading column = FK) for parent deletes and joins.
- Prefer narrow surrogate PKs for large tables; keep natural keys as `UNIQUE`.
- Avoid deep cascade chains; they surprise people and lock widely. Prefer explicit deletes in a controlled order or soft deletes.
- Load order: parents/dimensions first, then facts; or disable-and-recheck constraints for bulk loads and verify with `WITH CHECK`.
- In warehouses, run orphan queries per load and quarantine orphans or route to an `-1 Unknown` dimension row (late-arriving dimension pattern).
- Surrogate key generation in loads: identity/sequence on the dimension, with the natural key looked up in the dimension to translate facts.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `PRIMARY KEY`, `UNIQUE` | Entity integrity | Every table |
| `FOREIGN KEY … REFERENCES` | Referential integrity | OLTP |
| `ON DELETE CASCADE / SET NULL` | Referential actions | Truly owned children |
| `ALTER TABLE … WITH CHECK CHECK CONSTRAINT` | Re-validate and re-trust | After bulk load |
| `sys.foreign_keys.is_not_trusted` | Find untrusted FKs | Health check |
| `DBCC CHECKCONSTRAINTS` | Find violations | Audit |
| `IDENTITY` / `SEQUENCE` | Generate surrogate keys | Dimensions, OLTP |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Surrogate PK + natural `UNIQUE` | Natural key is stable, tiny, and never reused (e.g. ISO country code) |
| Declared FK | Extreme-throughput ingestion where you validate in the pipeline instead |
| `CASCADE` | Delete should be deliberate/audited; large cascades |
| `SET NULL` | Column is `NOT NULL` or NULL has business meaning conflict |
| GUID key | Ordered inserts matter — use `NEWSEQUENTIALID()` or a sequence |

### 9. Common mistakes
- No index on the FK column → slow parent delete, lock escalation → index it.
- Natural key as PK, then the value changes (email, SSN reuse) → cascading updates and "collisions" → surrogate PK.
- Disabling FKs for a load and never re-checking → untrusted constraints, orphans → `WITH CHECK CHECK`.
- Composite FK with a NULL part → silently unchecked → declare columns `NOT NULL`.
- Assuming a `UNIQUE` constraint prevents multiple NULLs → in SQL Server only one NULL allowed (use filtered unique index for many); other engines differ.
- Cyclic FKs between two tables → impossible inserts → make one nullable and update afterwards.
- Cascade on a self-reference → not allowed in SQL Server for multiple cascade paths → use triggers/procs.

### 10. 🎯 Interview kit
**30-second answer:** "A primary key uniquely and non-nullably identifies a row; a foreign key forces each child to point at an existing parent or be NULL. Surrogate keys decouple identity from changing business data. Where the warehouse doesn't enforce FKs, I check for orphans with a LEFT JOIN anti-join."

**Structure a longer answer:** clarify (OLTP vs warehouse, key stability) → mechanism (index-backed checks, referential actions) → trade-offs (surrogate vs natural, cascades, enforcement vs load speed) → scenario (orphan facts after a late dimension).

**Likely follow-ups:**
- *Candidate vs super vs composite?* → minimal unique set vs any unique set vs multi-column.
- *Why can a child insert fail?* → missing/invisible parent, type/collation mismatch, load order.
- *Detect cycles?* → recursive CTE with visited path, or trigger.
- *Surrogate key in a dimension load?* → identity on dim, lookup on natural key.

**Signals interviewers look for:** index-your-FKs awareness, trust of constraints, practical orphan handling.

### 11. Check yourself
- [ ] Define super, candidate, primary and alternate keys with an example.
- [ ] Write the orphan anti-join.
- [ ] Explain why FK columns need their own index.
- [ ] Say what "not trusted" means and how to fix it.
- [ ] Choose surrogate vs natural key for a customer table and justify.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../../../TOOLS/MySQL/_index.md>)

## Practice questions

- [Basic] [What is a primary key and how is it different from a unique key?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40486_What_is_a_primary_key_and_how_is_it_different_from_a_unique_.md>)
- [Basic] [What is a primary key, and what would you choose as the primary key for a table of embedded chunks?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40741_What_is_a_primary_key_and_what_would_you_choose_as_the_prima.md>)
- [Basic] [What is a primary key, and what would you choose as the primary key for a metrics table to avoid duplicate samples?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-41050_What_is_a_primary_key_and_what_would_you_choose_as_the_prima.md>)
- [Intermediate] [What is the difference between ON DELETE CASCADE, SET NULL, and RESTRICT for a foreign key?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40230_What_is_the_difference_between_ON_DELETE_CASCADE_SET_NULL_an.md>)
- [Intermediate] [What does ON DELETE CASCADE do, and what are the risks of using it?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-41130_What_does_ON_DELETE_CASCADE_do_and_what_are_the_risks_of_usi.md>)
- [Intermediate] [How would you design a schema to model soft-deleted records while keeping foreign keys, unique constraints, and query ergonomics sane? What are the partial-unique-index tricks involved?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40301_How_would_you_design_a_schema_to_model_soft-deleted_records_.md>)
- [Deep] [What is a surrogate key versus a natural key, and which would you choose?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-41075_What_is_a_surrogate_key_versus_a_natural_key_and_which_would.md>)
- [Deep] [Two services both need a user's email as a unique key, but one is being extracted into its own DB. How do you preserve uniqueness and referential guarantees once they no longer share a database?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40367_Two_services_both_need_a_user_s_email_as_a_unique_key_but_on.md>)

## Related concepts

- [Surrogate vs Natural Keys for Lookups](<../../Database_Design/Schema_Design/Surrogate_vs_Natural_Keys_for_Lookups.md>) *(Schema Design)*
- [Foreign Keys](<../../Database_Design/Constraints_and_Keys/Foreign_Keys.md>) *(Constraints and Keys)*
- [Surrogate Keys in Normalization](<../../Database_Design/Normalization/Surrogate_Keys_in_Normalization.md>) *(Normalization)*

## References

- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Background to T-SQL querying and programming
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 14 Query Design Analysis
- *SQL Server Tutorial* (YouTube series) — Part 5: Cascading referential integrity constraint ([watch](https://www.youtube.com/watch?v=ETepOVi7Xk8))

