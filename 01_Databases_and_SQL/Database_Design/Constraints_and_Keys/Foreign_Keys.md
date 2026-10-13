---
id: "01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Foreign_Keys"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Design/Constraints_and_Keys"
tools: []
---

# Foreign Keys

**L4 Advanced** · [↑ Constraints and Keys](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A foreign key forces every value in a child column to match an existing key in a parent table (or be NULL), so orphan rows cannot exist — at the price of validation work on writes and the need to index the child column.

### 1. The big picture
Relational design splits data across tables joined by keys: `Orders.customer_id` points at `Customers.customer_id`. Without a declared constraint nothing stops an order for a customer that doesn't exist, or a customer being deleted while orders still reference it. A foreign key (FK) is that guard: **referential integrity** enforced by the engine.

Designers often forget FKs because ER diagrams don't show them, but the cost of forgetting is orphan rows, broken joins and inconsistent reports. The cost of having them is modest *if* the child column is indexed. This lesson covers declaration, behaviour, and the performance mechanics; the referential actions (CASCADE etc.) are in the next lesson.

### 2. Core concepts and vocabulary
- **Parent / referenced table** — holds the key being pointed to.
- **Child / referencing table** — holds the FK column(s).
- **Referenced key** — must be a PRIMARY KEY or UNIQUE key of the parent.
- **FK column(s)** — same count and compatible types as the referenced key.
- **Orphan** — child row whose FK value has no parent.
- **Self-referencing FK** — parent and child are the same table (`Employees.mgrid → Employees.empid`).
- **Composite FK** — multiple columns together.
- **Optional relationship** — FK column nullable; NULL means "no parent".
- **Declarative referential integrity (DRI)** — integrity via constraints rather than code.
- **Trusted FK** — validated for all rows; usable for optimization.

### 3. How it works, step by step
**On INSERT/UPDATE of the child:**
1. If any FK column is NULL (for the default MATCH SIMPLE), skip the check.
2. Otherwise look up the value in the parent's referenced key (index seek).
3. No match → error (SQL Server 547, PostgreSQL 23503).

**On DELETE/UPDATE of the parent key:**
1. Look for child rows referencing the old value.
2. If found, apply the referential action (NO ACTION/RESTRICT error, or CASCADE, SET NULL, SET DEFAULT).

So the parent is checked when children change, and the **child** is checked when the parent changes. The second check scans the child's FK column — this is the one that hurts without an index.

### 4. Worked example
```sql
CREATE TABLE dbo.Customers (
    custid INT NOT NULL CONSTRAINT PK_Customers PRIMARY KEY,
    name   VARCHAR(100) NOT NULL);

CREATE TABLE dbo.Orders (
    orderid INT NOT NULL CONSTRAINT PK_Orders PRIMARY KEY,
    custid  INT NOT NULL,
    empid   INT NULL,
    CONSTRAINT FK_Orders_Customers FOREIGN KEY (custid) REFERENCES dbo.Customers(custid)
);
-- SQL Server and Oracle do not auto-index the FK column; MySQL InnoDB does; PostgreSQL does not.
CREATE INDEX IX_Orders_custid ON dbo.Orders(custid);

-- self-reference
ALTER TABLE dbo.Employees ADD CONSTRAINT FK_Employees_Mgr
    FOREIGN KEY (mgrid) REFERENCES dbo.Employees(empid);   -- mgrid NULL for the CEO
```
Behaviour:
```sql
INSERT INTO dbo.Orders VALUES (1, 999, NULL);   -- fails: customer 999 does not exist
DELETE FROM dbo.Customers WHERE custid = 5;     -- fails if orders for 5 exist (NO ACTION)
```
Find orphans before adding an FK to existing data:
```sql
SELECT o.* FROM dbo.Orders o
LEFT JOIN dbo.Customers c ON c.custid = o.custid
WHERE c.custid IS NULL;
```
Find unindexed FKs (SQL Server): compare `sys.foreign_key_columns` with `sys.index_columns` leading columns.

### 5. Under the hood
- Validation on child write = one index seek on the parent's PK/unique index — cheap, since that key is almost always indexed (and clustered).
- Validation on parent delete/update = a probe of the child table by FK value. **Without an index on the child column this is a scan of the whole child table**, taken while locks are held; with many parents deleted it's a scan per delete (or a hash/merge semi-join in the plan: "Assert" / "Left Semi Join" operators in SQL Server's DML plans).
- Locking: an FK check takes shared locks on the looked-up parent row (or key range) so that parent can't vanish before commit; concurrent parent deletes and child inserts can block or deadlock. Under SERIALIZABLE/snapshot isolation implementations differ.
- Enforcement is immediate by default; PostgreSQL/Oracle allow `DEFERRABLE INITIALLY DEFERRED` (check at commit), useful for circular references and bulk loading.
- Trusted FKs let the optimizer eliminate joins: if a query joins child to parent only to guarantee existence and selects no parent column, a trusted non-null FK allows **join elimination**; cardinality estimates are also better.
- Cross-database/shard foreign keys are not supported by engines (or by distributed stores such as Cassandra and most warehouses, where FKs are informational, `NOT ENFORCED`).

### 6. Performance and design implications
- Always index FK columns, leading with the FK column (e.g. `(custid, orderdate)` serves FK checks *and* queries). This single habit prevents most FK-related blocking and slow deletes.
- Bulk loading: disable/not-validate constraints then re-validate (`WITH CHECK CHECK CONSTRAINT`) — otherwise plans lose trust. Load parents first.
- Wide composite FKs enlarge indexes; prefer surrogate keys for heavily referenced tables.
- In data warehouses FKs are often declared but not enforced (informational) for optimizer hints; quality is guaranteed by the load process (and tests such as `relationships` in dbt).
- Hot parent rows (e.g. a single "default" customer) can become a contention point under high insert concurrency.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `FOREIGN KEY (c) REFERENCES p(k)` | Declare | Every real relationship |
| `ON DELETE / ON UPDATE ...` | Referential action | Policy for parent changes |
| `NOCHECK CONSTRAINT` / `WITH NOCHECK` | Skip validation | Loading legacy data (temporarily) |
| `WITH CHECK CHECK CONSTRAINT` | Revalidate, restore trust | After loads |
| `DEFERRABLE INITIALLY DEFERRED` | Check at commit | Circular references, bulk loads (PG/Oracle) |
| `CREATE INDEX ON child(fk)` | Supports FK checks | Always |
| `sys.foreign_keys` (`is_not_trusted`) | Inspect | Audits |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| OLTP source of truth | Pure analytics/warehouse tables with load-time guarantees → informational FKs |
| Optional relationship → nullable FK | NULL semantics confuse reports → consider a separate association table |
| Index the FK | Write-heavy table where the index hurts and parent never changes — rare |
| Single-column surrogate FK | Natural composite keys across many tables → wide indexes |

### 9. Common mistakes
- No FK declared ("the app handles it") → orphans accumulate → add constraints, clean existing data first.
- Unindexed FK column → parent delete scans the child; deadlocks/blocking → index it.
- Mismatched types (INT vs BIGINT, collation) → implicit conversions, can't create FK → align definitions.
- Loading with constraints disabled and never revalidating → untrusted FK → optimizer ignores it → `WITH CHECK CHECK`.
- Assuming NULL FK values violate → they don't (MATCH SIMPLE) → add NOT NULL when the relationship is mandatory.
- Circular FK between two tables without deferrable → can't insert → nullable column then update, or deferrable constraints.
- Believing FK guarantees one-to-one → it's one-to-many; add UNIQUE on the FK for one-to-one.

### 10. 🎯 Interview kit
**30-second answer:** A foreign key constrains a column in the child table to values existing in the parent's primary or unique key, or NULL. It prevents orphans, is checked on child insert/update and on parent delete/update, and the child column should be indexed.

**Structure a longer answer:** clarify → definition and example → what happens on each operation → referential actions → performance (index, locking, trusted) → scenario (delete slow, deadlocks).

**Likely follow-ups:**
- *Does the FK column need an index?* → not required, but nearly always should have one.
- *Can it reference a non-key?* → no, must be PK or unique.
- *FK to the same table?* → yes (hierarchies).
- *NULL in FK?* → allowed, check skipped.
- *Why might a bulk load disable FKs?* → speed; revalidate to re-trust.
- *One-to-one with FK?* → add UNIQUE constraint.

**Signals interviewers look for:** you mention the child index and the two directions of checks, and trusted constraints.

### 11. Check yourself
- [ ] Draw the two validation directions for parent and child.
- [ ] Explain why deleting one parent row can scan the whole child table.
- [ ] Write a query finding orphans and one finding unindexed FKs.
- [ ] Model a one-to-one relationship with an FK.
<!-- explainer:end -->

## Practice questions

- [Basic] [Write the SQL to add a foreign key on dept_id in employees referencing departments.](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10112_Write_the_SQL_to_add_a_foreign_key_on_dept_id_in_employees_r.md>)
- [Intermediate] [What does ON DELETE SET DEFAULT do on a foreign key?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10155_What_does_ON_DELETE_SET_DEFAULT_do_on_a_foreign_key.md>)
- [Intermediate] [Must the data type of a foreign key column match the data type of the referenced primary key column? Why?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10198_Must_the_data_type_of_a_foreign_key_column_match_the_data_ty.md>)
- [Intermediate] [Explain how a foreign key with ON DELETE SET NULL interacts with a NOT NULL constraint on the same column, and how the DB resolves the conflict.](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10121_Explain_how_a_foreign_key_with_ON_DELETE_SET_NULL_interacts_.md>)
- [Deep] [Can a primary key column be a foreign key at the same time? Give an example.](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10222_Can_a_primary_key_column_be_a_foreign_key_at_the_same_time_G.md>)
- [Deep] [Explain how deferred constraint checking within a transaction differs from immediate checking, and give a scenario where deferral is essential.](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10077_Explain_how_deferred_constraint_checking_within_a_transactio.md>)
- [Deep] [How do you enforce referential integrity between a relational table and a JSONB column holding an array of child IDs? Discuss the trade-offs.](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10117_How_do_you_enforce_referential_integrity_between_a_relationa.md>)
- [Expert] [When would you deliberately choose NOT to enforce foreign keys at the database level in a high-throughput system, and what safeguards replace them?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10043_When_would_you_deliberately_choose_NOT_to_enforce_foreign_ke.md>)

## Related concepts

- [Foreign Keys and Referential Integrity](<../../SQL_Language/SQL_Fundamentals/Foreign_Keys_and_Referential_Integrity.md>) *(SQL Fundamentals)*
- [Surrogate vs Natural Keys for Lookups](<../Schema_Design/Surrogate_vs_Natural_Keys_for_Lookups.md>) *(Schema Design)*
- [Polymorphic Associations](<../Schema_Design/Polymorphic_Associations.md>) *(Schema Design)*
- [Lock Granularity](<../../Database_Internals/DBMS_Concepts/Lock_Granularity.md>) *(DBMS Concepts)*
- [Adding Constraints Online](<../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Adding_Constraints_Online.md>) *(PostgreSQL Platform)*

## References

- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Foreign Key
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Foreign Keys
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Unindexed Foreign Keys
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Background to T-SQL querying and programming
- *Grokking Relational Database Design* — Qiang Hao, Michail Tsikerdekis — Grokking Relational Database Design (cont.)
- *Grokking Relational Database Design* — Qiang Hao, Michail Tsikerdekis — Implementation
- *Grokking Relational Database Design* — Qiang Hao, Michail Tsikerdekis — Implementation (cont.)
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — 13. Implementing Logic in the Database › Creating Foreign Keys
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — 13. Implementing Logic in the Database › Creating Foreign Keys with Delete and Update Rules
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 14: Unit Testing T-SQL with tSQLt › Handling Foreign Keys in Fake Tables
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Database design › 4.3 #13 Not using foreign keys
- *SQL Server Tutorial* (YouTube series) — Part 3: Creating and working with tables ([watch](https://www.youtube.com/watch?v=JLeaM8pK8dE))

