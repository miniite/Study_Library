---
id: "01_Databases_and_SQL/SQL_Language/SQL_Command_Types/DROP_and_CASCADE"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Command_Types"
tools: []
---

# DROP and CASCADE

**L4 Advanced** · [↑ SQL Command Types](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** `DROP` permanently removes an object (table, index, database, column) together with its data; dependencies (foreign keys, views, other objects) either block it or, with `CASCADE`, are removed or deleted along with it — so the safe path is always to inspect dependencies and have a recovery plan first.

### 1. The big picture
`DROP` is the most destructive routine command: structure, data, indexes and constraints disappear in one statement. Because relational data is linked, other objects usually depend on the dropped one: child rows through foreign keys, views and procedures that reference a table, indexes and constraints that belong to it. Databases therefore protect you by *refusing* the drop while dependencies exist (`RESTRICT`, the default), or let you push through with `CASCADE`, which removes the dependents too. A related concept applies to deleting *rows*: `ON DELETE CASCADE` on a foreign key propagates row deletions to children.

Lakehouse engines add another dimension: dropping a *managed* table deletes its data files; dropping an *external* table only removes metadata.

### 2. Core concepts and vocabulary
- **`DROP TABLE`** — removes table definition, data, indexes, constraints, triggers and permissions; no `WHERE`, no undo outside a transaction/backup.
- **`DROP DATABASE`** — removes the whole database and all its objects and files.
- **`DROP INDEX / VIEW / CONSTRAINT`** — removes a single object.
- **`ALTER TABLE … DROP COLUMN`** — removes a column and its data; fails if indexes, constraints or computed columns depend on it.
- **`RESTRICT`** — refuse if anything depends on the object (default in the standard and PostgreSQL).
- **`CASCADE`** — also drop dependent objects (PostgreSQL: views, FK constraints — *not* the child rows).
- **`ON DELETE CASCADE`** — referential action: deleting a parent row deletes its child rows; also `SET NULL`, `SET DEFAULT`, `NO ACTION`.
- **Orphan rows** — children whose parent was removed; the FK constraint exists to prevent them.
- **Managed vs external table** — managed: engine owns the data files (dropped with the table); external: data lives elsewhere and stays.
- **Soft delete** — a flag column instead of physical deletion.

### 3. How it works, step by step
1. **Dependency check** — the engine consults the catalog. A table referenced by an FK cannot be dropped (SQL Server: *Could not drop object because it is referenced by a FOREIGN KEY constraint*; PostgreSQL: *cannot drop table … because other objects depend on it*).
2. **Resolution**:
   - Drop the dependent FK constraints first (`ALTER TABLE child DROP CONSTRAINT fk_…`), or drop the child table first, **or**
   - PostgreSQL: `DROP TABLE parent CASCADE` drops the *constraints* (and dependent views) — the child *table and its rows remain*.
3. **Deallocation** — pages/extents are released; catalog entries removed; in a transaction the drop can be rolled back (SQL Server, PostgreSQL), not in Oracle/MySQL (implicit commit).
4. **Row-level `ON DELETE CASCADE`**: deleting a parent row triggers deletion of matching children, recursively through multi-level chains, inside the same statement.

### 4. Worked example
```sql
CREATE TABLE customers (cust_id INT PRIMARY KEY, name VARCHAR(50));
CREATE TABLE orders (
   order_id INT PRIMARY KEY,
   cust_id  INT NOT NULL
        CONSTRAINT fk_orders_cust REFERENCES customers(cust_id) ON DELETE CASCADE
);

-- Row-level cascade
DELETE FROM customers WHERE cust_id = 1;     -- also deletes that customer's orders

-- Table-level protection
DROP TABLE customers;
-- ERROR: cannot drop table customers because other objects depend on it
-- DETAIL: constraint fk_orders_cust on table orders depends on table customers

-- Options
ALTER TABLE orders DROP CONSTRAINT fk_orders_cust;  DROP TABLE customers;    -- explicit, any engine
DROP TABLE customers CASCADE;                       -- PostgreSQL: drops the FK, keeps orders table
DROP TABLE orders; DROP TABLE customers;            -- child first

-- Safe habits
BEGIN TRAN;  DROP TABLE staging.tmp_load;  -- verify, then
COMMIT;      -- or ROLLBACK
DROP TABLE IF EXISTS staging.tmp_load;     -- idempotent scripts (SQL Server 2016+, PostgreSQL)

-- Find what depends on an object (SQL Server)
SELECT referencing_schema_name, referencing_entity_name
FROM sys.dm_sql_referencing_entities('dbo.customers', 'OBJECT');
```
What to observe: after `DROP TABLE customers CASCADE` in PostgreSQL, `\d orders` shows the table still exists but without the foreign key — silent loss of integrity, which is why `CASCADE` should be used consciously.

### 5. Under the hood   *(optional at L4)*
- Dropping a table is mostly **metadata + allocation work**: the engine deallocates the IAM/extents; big tables may take time but are far cheaper than `DELETE`. In SQL Server the physical page deallocation can be deferred (deferred drop) so space appears later.
- `DROP TABLE` takes a schema-modification lock: it blocks all concurrent access and waits on running queries; long transactions can block it (and then everything queues behind it).
- Dependent objects in SQL Server are tracked in `sys.sql_expression_dependencies`; schema-bound views and computed columns *prevent* the drop outright.
- **Delta Lake / Databricks**: `DROP TABLE` on a managed table removes metastore entry and data files (no undo); on an external table only the metastore entry. `DELETE FROM table` is recoverable through time travel, `DROP` with file removal is not; create a clone or backup before irreversible actions.
- Recovery: from backup / point-in-time restore; PostgreSQL has no undrop; some cloud services provide recycle bins.

### 6. Performance and design implications
- `DROP` is fast relative to deleting rows; use it (or partition drop/switch) for large removal.
- Cascading deletes multiply work: deleting one parent can delete thousands of children and lock them; index FK columns so lookups are not scans.
- Multi-level cascades hide large blast radius; many teams prefer `NO ACTION` plus explicit application-level or batch deletes.
- Dropping a column rewrites or marks metadata depending on engine (SQL Server marks it dropped; space reclaimed on rebuild; PostgreSQL marks dropped, reclaim on rewrite).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `DROP TABLE [IF EXISTS] t` | Remove a table | Decommissioning, scratch objects |
| `DROP TABLE t CASCADE` (PG) | Also drop dependent constraints/views | After checking the dependency list |
| `ALTER TABLE … DROP CONSTRAINT` | Remove an FK/check | Before dropping/replacing a parent |
| `ALTER TABLE … DROP COLUMN` | Remove a column | Schema cleanup (after code stops using it) |
| `ON DELETE CASCADE / SET NULL / NO ACTION` | Row-level referential action | Parent-child lifecycle design |
| `DROP DATABASE` | Remove a database | Last resort; check connections |
| `sys.dm_sql_referencing_entities`, `pg_depend`, `information_schema` | Find dependents | Before any drop |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `ON DELETE CASCADE` for true ownership (order → order lines) | Parent is shared reference data (e.g. countries) |
| `RESTRICT/NO ACTION` | You want automatic cleanup |
| `SET NULL` | The FK column is `NOT NULL` or the child is meaningless without a parent |
| Soft delete | Storage/GDPR requirements demand physical deletion |
| `DROP … CASCADE` | You haven't listed what it will remove |

### 9. Common mistakes
- Running `DROP` in the wrong database/environment → loss → confirm connection, use least-privilege roles, require change tickets.
- `CASCADE` used to "make the error go away" → silently removed views/constraints → read the dependency list first.
- Assuming `DROP TABLE … CASCADE` deletes child rows → it drops constraints (PostgreSQL) → delete rows explicitly.
- Dropping an external table expecting data deletion (or a managed one expecting retention) → surprise loss or orphaned files.
- No backup before dropping → permanent loss → verified backup/clone first.

### 10. 🎯 Interview kit
**30-second answer:** "`DROP` removes the object and its data permanently; it fails if foreign keys or other objects depend on it. `CASCADE` drops dependents too (constraints/views), whereas `ON DELETE CASCADE` is a row-level FK action that deletes children when a parent row is deleted. Check dependencies and have a backup first."

**Structure a longer answer:** what DROP removes → dependency protection → `RESTRICT`/`CASCADE` → row-level cascades → engine differences (transactional DDL, managed/external tables) → safe procedure.

**Likely follow-ups:**
- *`DROP TABLE` vs `DROP DATABASE`?* → table scope vs whole database.
- *`DROP` vs `DELETE` vs `TRUNCATE`?* → structure+data / rows with `WHERE` / all rows, keeps structure.
- *How to drop a column?* → `ALTER TABLE t DROP COLUMN c` after removing dependent indexes/constraints.
- *Can you roll back a DROP?* → in SQL Server/PostgreSQL inside a transaction, not in Oracle/MySQL.

**Signals interviewers look for:** you distinguish the DDL cascade from the FK cascade, and describe a safety routine.

### 11. Check yourself
- [ ] Explain why dropping a referenced parent table fails and list three ways forward.
- [ ] State what `DROP TABLE … CASCADE` removes in PostgreSQL.
- [ ] Compare `DROP`, `DELETE`, `TRUNCATE` in a table.
- [ ] Describe what happens when you drop a managed vs external Spark/Databricks table.
<!-- explainer:end -->

## Practice questions

- [Basic] [How do you drop a column from a table using ALTER?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22964_How_do_you_drop_a_column_from_a_table_using_ALTER.md>)
- [Basic] [What is the difference between DROP TABLE and DROP DATABASE?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22968_What_is_the_difference_between_DROP_TABLE_and_DROP_DATABASE.md>)
- [Basic] [What is the purpose of the ALTER command and give one example use?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22981_What_is_the_purpose_of_the_ALTER_command_and_give_one_exampl.md>)
- [Basic] [How do you drop a constraint from a table?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22994_How_do_you_drop_a_constraint_from_a_table.md>)
- [Intermediate] [What is the risk of running a DELETE without a WHERE clause and how do you avoid it?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22859_What_is_the_risk_of_running_a_DELETE_without_a_WHERE_clause_.md>)
- [Intermediate] [How do you delete rows in one table based on a condition in another table?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-23000_How_do_you_delete_rows_in_one_table_based_on_a_condition_in_.md>)
- [Intermediate] [What does the CASCADE option do in a DROP or REVOKE statement?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-23023_What_does_the_CASCADE_option_do_in_a_DROP_or_REVOKE_statemen.md>)
- [Intermediate] [Your DELETE cascade unexpectedly removed rows in five child tables. How do you audit ON DELETE CASCADE chains before shipping such a change?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Q-22868_Your_DELETE_cascade_unexpectedly_removed_rows_in_five_child_.md>)

## Related concepts

- [Online Schema Changes](<../../../TOOLS/MySQL/MySQL_Platform/Online_Schema_Changes.md>) *(MySQL Platform)*
- [DELETE TRUNCATE and DROP](<../../../TOOLS/PostgreSQL/PostgreSQL_Platform/DELETE_TRUNCATE_and_DROP.md>) *(PostgreSQL Platform)*
- [Scalar Subqueries](<../SQL_Fundamentals/Scalar_Subqueries.md>) *(SQL Fundamentals)*

## References

- *Grokking Relational Database Design* — Qiang Hao, Michail Tsikerdekis — Grokking Relational Database Design (cont.)
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 1: Getting Started with T-SQL › DROP
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Diving into the Delta Lake Ecosystem › Deleting tables
- *Delta Lake: The Definitive Guide* — Denny Lee et al. — Maintaining Your Delta Lake › Dropping tables
- *Hadoop: The Definitive Guide* — Tom White — Hive › Dropping Tables
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Managing Data with Delta Lake › Dropping Delta Lake Tables
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Mastering Relational Entities in Databricks › Dropping tables
- *Databricks Certified Data Engineer Associate Study Guide* — Derar Alhussein — Implementing Data Governance › Dropping tables
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Manipulating Data › Deleting Rows
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Manipulating Data › Deleting from a Table in a JOIN
- *SQL Server Tutorial* (YouTube series) — Part 67: Alter database table columns without dropping table ([watch](https://www.youtube.com/watch?v=eQu_kRlvinE))

