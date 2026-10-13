---
id: "01_Databases_and_SQL/Database_Objects/Triggers/DML_vs_DDL_Triggers"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Objects/Triggers"
tools: []
---

# DML vs DDL Triggers

**L4 Advanced** · [↑ Triggers](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A trigger is a stored procedure you cannot call — it is attached to an event and runs automatically inside the transaction of whatever caused it; *DML triggers* react to row changes (`INSERT`/`UPDATE`/`DELETE`), *DDL triggers* react to schema and server events (`CREATE`/`ALTER`/`DROP`, logins).

### 1. The big picture
A normal stored procedure runs when somebody `EXEC`s it. A trigger inverts that: you declare "when *this* event happens, run *this* code", and the engine invokes it for you. Because the invocation is implicit, triggers are powerful (every code path, including ad-hoc `UPDATE`s typed into SSMS, goes through them) and dangerous (nobody sees them in the calling code).

SQL Server supports two families. **DML triggers** are tied to a table or view and fire on data modification. **DDL triggers** are tied to a database or the whole server and fire on definition events (`CREATE TABLE`, `DROP INDEX`, `ALTER LOGIN`) and, in the case of logon triggers, on connection attempts. DML triggers protect and derive *data*; DDL triggers govern and audit the *schema and environment*.

This lesson covers the two families side by side, the timing options (`AFTER`, `INSTEAD OF`, and the `BEFORE`/`AFTER` model other engines use), the `inserted`/`deleted` pseudo-tables, and how to discover triggers that already exist. The next lesson covers behavior under load and when to avoid triggers.

### 2. Core concepts and vocabulary
- **Trigger** — a special stored procedure bound to an event; it cannot be executed explicitly.
- **DML trigger** — fires for `INSERT`, `UPDATE`, `DELETE` (and `MERGE`, which fires the trigger for each action it performs) on a table or view.
- **DDL trigger** — fires for definition events at database scope (`CREATE_TABLE`, `ALTER_PROCEDURE`…) or server scope (`CREATE_DATABASE`, `ALTER_LOGIN`…).
- **Logon trigger** — a server-scope trigger that fires on `LOGON`, before the session is established; can reject the login.
- **`AFTER` trigger** — runs after the triggering statement has done its work and passed constraints. Only on permanent tables. Several allowed per event.
- **`INSTEAD OF` trigger** — runs *instead of* the triggering statement. Allowed on tables and views; at most one per event per object. The usual way to make a multi-table view updatable.
- **`inserted` / `deleted`** — pseudo-tables available inside a DML trigger holding the *new* and *old* images of the affected rows. `INSERT` populates only `inserted`, `DELETE` only `deleted`, `UPDATE` both.
- **`EVENTDATA()`** — function returning XML describing the DDL event (which object, which login, the exact T-SQL text) inside a DDL trigger.
- **Statement-level firing** — SQL Server triggers fire *once per statement*, not once per row. A statement touching 5,000 rows fires the trigger once with 5,000 rows in `inserted`.
- **`BEFORE` trigger / `FOR EACH ROW`** — the model used by MySQL, PostgreSQL and Oracle: row-level triggers that can edit the incoming row (`NEW`) before it is stored. SQL Server has no `BEFORE` trigger; `INSTEAD OF` is the closest.

### 3. How it works, step by step
**DML trigger life cycle (`AFTER`):**
1. A statement such as `UPDATE Sales.Orders SET ...` starts executing inside a transaction (explicit, or the implicit one for that single statement).
2. The engine modifies the rows and checks constraints (`CHECK`, `FOREIGN KEY`, unique indexes) as part of the statement.
3. If the statement succeeded, the engine materializes the affected rows into `inserted`/`deleted` (they are read from the transaction log / row-version store, not real tables) and runs every matching `AFTER` trigger — in the order set by `sp_settriggerorder` for first/last, otherwise undefined.
4. If the trigger raises an error or executes `ROLLBACK`, the *whole* triggering statement — and by default the entire outer transaction — is undone.
5. If it finishes normally, the statement completes and the transaction continues.

**`INSTEAD OF` life cycle:** the original modification is *not* performed. The trigger body decides what, if anything, to do — typically inserting into the real base tables after checks. Because the base modification never ran, constraints are checked only on whatever the trigger itself writes.

**DDL trigger life cycle:** the DDL statement executes, then the trigger fires inside the same transaction; `ROLLBACK` in the trigger undoes the DDL. This is how you can *prevent* `DROP TABLE` in production.

| Aspect | DML trigger | DDL trigger |
|---|---|---|
| Bound to | Table or view | Database or server |
| Events | `INSERT`/`UPDATE`/`DELETE` | `CREATE`/`ALTER`/`DROP`, `GRANT`, logon… |
| Row data | `inserted`, `deleted` | `EVENTDATA()` XML |
| Typical use | Audit, derived columns, cross-row rules, updatable views | Schema-change audit, change control, block drops, log logins |
| `INSTEAD OF` available | Yes | No (only `AFTER`/`FOR`) — except logon policy via `ROLLBACK` |
| Granularity | Per statement | Per DDL statement |

### 4. Worked example
```sql
-- 1. DML AFTER trigger: audit every order insert (multi-row safe)
CREATE TRIGGER Sales.trg_Orders_AfterInsert
ON Sales.Orders
AFTER INSERT
AS
BEGIN
    SET NOCOUNT ON;
    INSERT INTO Audit.OrderLog (OrderID, Action, ActionDate, ActionBy)
    SELECT OrderID, 'INSERT', SYSDATETIME(), ORIGINAL_LOGIN()
    FROM inserted;                       -- set-based: works for 1 or 10,000 rows
END;
GO

-- 2. DML trigger that uses both pseudo-tables: only react when Status actually changed
CREATE TRIGGER Sales.trg_Orders_StatusChange
ON Sales.Orders
AFTER UPDATE
AS
BEGIN
    SET NOCOUNT ON;
    IF NOT UPDATE(Status) RETURN;        -- column not in the SET list: exit cheaply
    INSERT INTO Audit.OrderStatusHistory (OrderID, OldStatus, NewStatus, ChangedAt)
    SELECT d.OrderID, d.Status, i.Status, SYSDATETIME()
    FROM deleted AS d
    JOIN inserted AS i ON i.OrderID = d.OrderID
    WHERE ISNULL(d.Status,'') <> ISNULL(i.Status,'');   -- UPDATE(col) is TRUE even if value unchanged
END;
GO

-- 3. DDL trigger: log every schema change in this database
CREATE TRIGGER ddl_AuditSchema
ON DATABASE
FOR DDL_DATABASE_LEVEL_EVENTS
AS
BEGIN
    SET NOCOUNT ON;
    DECLARE @e XML = EVENTDATA();
    INSERT INTO Audit.SchemaChanges (EventTime, LoginName, EventType, ObjectName, TsqlText)
    VALUES (
      @e.value('(/EVENT_INSTANCE/PostTime)[1]',   'datetime2'),
      @e.value('(/EVENT_INSTANCE/LoginName)[1]',  'sysname'),
      @e.value('(/EVENT_INSTANCE/EventType)[1]',  'sysname'),
      @e.value('(/EVENT_INSTANCE/ObjectName)[1]', 'sysname'),
      @e.value('(/EVENT_INSTANCE/TSQLCommand/CommandText)[1]', 'nvarchar(max)'));
END;
GO

-- 4. Discover triggers (new developer on an unfamiliar database)
SELECT t.name, OBJECT_NAME(t.parent_id) AS parent, t.is_instead_of_trigger, t.is_disabled
FROM sys.triggers AS t;                              -- DML and database DDL triggers
SELECT * FROM sys.server_triggers;                   -- server-scope
SELECT OBJECT_DEFINITION(OBJECT_ID('Sales.trg_Orders_StatusChange'));
EXEC sp_helptrigger 'Sales.Orders';
```
**Reading it:** trigger 1 is correct because it joins to `inserted` as a set; a version written as `SELECT @id = OrderID FROM inserted` would silently audit only one row of a bulk insert. In trigger 2, `UPDATE(Status)` only says the column appeared in the `SET` clause — comparing old and new in the `WHERE` is what actually filters no-op updates. In trigger 3, `EVENTDATA()` gives the XML you shred with `.value()`.

### 5. Under the hood
- **Where `inserted`/`deleted` come from.** They are not stored tables. Under the default row-versioning-less model the engine reads before/after images from the transaction log records generated by the modification; with `READ_COMMITTED_SNAPSHOT` or `SNAPSHOT` isolation they come from the version store in `tempdb`. That is why triggers add `tempdb` and log pressure on busy tables.
- **Plan shape.** The statement that fires the trigger shows the trigger's work as additional operators or a separate "Trigger" branch in the actual plan; the triggering statement's elapsed time includes it.
- **Nesting and recursion.** `AFTER` triggers can fire other triggers (nested up to 32 levels; server option *nested triggers*). *Direct* recursion (a trigger updating its own table) is off by default (`RECURSIVE_TRIGGERS` database option). *Indirect* recursion needs that option too.
- **Why an `AFTER` trigger cannot change the row "before" it is stored.** By the time it runs the row is already written and constraint-checked; to change it you must issue a *second* `UPDATE` against the table (which costs another write and may re-fire triggers). Engines with `BEFORE ROW` triggers (Oracle, PostgreSQL, MySQL) can modify `NEW` for free, which is why "set `created_at`" is a `BEFORE INSERT` job there but a `DEFAULT` constraint job in SQL Server.
- **Compound triggers (Oracle).** One trigger body with `BEFORE STATEMENT`, `BEFORE EACH ROW`, `AFTER EACH ROW`, `AFTER STATEMENT` sections sharing variables. They exist to avoid the "mutating table" error and to batch row work into a single statement-level action.

### 6. Performance and design implications
- A trigger lengthens the triggering transaction: locks are held until the trigger finishes. Keep the body short, set-based, and index the columns it joins on.
- `inserted`/`deleted` have no indexes. Joining them to a large table is fine; joining them to each other in wide updates can be slow — consider copying to a `#temp` table with a key if the rowset is large.
- Use `IF NOT EXISTS (SELECT 1 FROM inserted) RETURN;` (or `@@ROWCOUNT = 0`) to exit early — a statement affecting zero rows still fires `AFTER` triggers.
- Measure with the actual execution plan, `sys.dm_exec_trigger_stats` (cumulative execution count, worker time, logical reads per trigger), and Extended Events.
- DDL triggers are cheap per event but run for *every* matching DDL — including index rebuilds issued by maintenance jobs. Scope the event list (`FOR CREATE_TABLE, ALTER_TABLE, DROP_TABLE`) rather than all events.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE TRIGGER … AFTER/INSTEAD OF …` | Define a DML trigger | Audit, derived data, view updatability |
| `CREATE TRIGGER … ON DATABASE / ON ALL SERVER` | Define a DDL/logon trigger | Change audit, block destructive DDL |
| `UPDATE(col)` / `COLUMNS_UPDATED()` | Was the column in the `SET` list? | Skip irrelevant updates |
| `EVENTDATA()` | XML for the DDL event | Inside DDL triggers |
| `sp_settriggerorder` | Mark first/last `AFTER` trigger | Only when order truly matters |
| `DISABLE / ENABLE TRIGGER … ON …` | Switch off temporarily | Bulk loads, then reconcile |
| `ALTER DATABASE … SET RECURSIVE_TRIGGERS` | Allow direct recursion | Rarely; usually leave off |
| `sys.triggers`, `sys.trigger_events`, `sys.server_triggers` | Catalog views | Discovery |
| `SET NOCOUNT ON` (inside body) | Suppress row-count messages | Always, so clients are not confused |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `AFTER` DML trigger for auditing or a cross-table rule no constraint can express | A `CHECK`, `FK`, `DEFAULT` or computed column does the job — constraints are declarative and cheaper |
| `INSTEAD OF` trigger to make a joined view updatable or to redirect writes | You need exactly-once guarantees; remember constraints are bypassed for the original statement |
| DDL trigger to audit or veto schema changes | You need to catch *who* in an environment with shared logins — `EVENTDATA` shows the login, not the person |
| Logon trigger to limit sessions or hours | The trigger has a bug — it can lock everyone out; keep the DAC (dedicated admin connection) available |
| Built-in alternatives (Change Tracking, CDC, Temporal tables, SQL Audit) for audit | You need custom business logic at write time |

### 9. Common mistakes
- Assuming a trigger fires per row and writing `SELECT @x = col FROM inserted` → only one row is processed in multi-row statements → write set-based code.
- Expecting `UPDATE(col)` to mean "value changed" → it is true even for `SET col = col` → compare `inserted` to `deleted`.
- Forgetting that `ROLLBACK` in a trigger kills the whole outer transaction → callers see error 3609 and lose unrelated work → prefer `THROW`/`RAISERROR` and let the caller decide, or design so rollback is intended.
- Putting integrity rules in triggers instead of constraints → slower and easy to bypass with `DISABLE TRIGGER` → use constraints first.
- Not documenting triggers → "a row update mysteriously changed three other tables" → keep a list (query `sys.triggers`) and name triggers `trg_<Table>_<Event>`.
- Letting a DDL trigger reference an audit table that can be dropped/renamed → every DDL fails → keep the audit table in a protected schema with permissions.

### 10. 🎯 Interview kit
**30-second answer:** A trigger is code that runs automatically when an event happens. DML triggers fire on `INSERT`/`UPDATE`/`DELETE` of a table and can see old/new rows through `deleted`/`inserted`; DDL triggers fire on schema or server events such as `CREATE`/`ALTER`/`DROP` and are used for auditing and change control. In SQL Server they are statement-level, run in the caller's transaction, and come as `AFTER` or `INSTEAD OF`.

**Structure a longer answer:** clarify (which engine? row vs statement trigger?) → mechanism (event → trigger inside same transaction → `inserted`/`deleted`) → trade-offs (hidden logic, lock duration, bulk-load cost) → scenario (audit table fed by an `AFTER UPDATE` trigger vs temporal tables).

**Likely follow-ups:**
- *Why can an `AFTER` trigger not alter the inserted values?* → the row is already written; you can only issue another `UPDATE`. Use `BEFORE` triggers (other engines), `DEFAULT`, or `INSTEAD OF`.
- *Difference between synchronous trigger logic and queue-based processing?* → triggers run inside the transaction and delay it; writing a message to a queue/Service Broker table and processing it asynchronously decouples latency and failure.
- *How would you list the triggers in a database?* → `sys.triggers`, `sys.server_triggers`, `OBJECT_DEFINITION`.
- *What would you use a DDL trigger for?* → audit schema changes with `EVENTDATA()`, block `DROP TABLE` in prod, enforce naming.
- *Compound trigger in Oracle?* → single object with statement- and row-level timing sections sharing state; solves mutating-table problems and lets you batch row work.

**Signals interviewers look for:** knows statement-level vs row-level, multi-row safety, transaction coupling, and that constraints/temporal tables are often the better tool.

### 11. Check yourself
- [ ] Explain what is in `inserted` and `deleted` for each of `INSERT`, `UPDATE`, `DELETE`.
- [ ] Write a trigger that rejects an `UPDATE` setting `Price` below `Cost` for *any* row in a multi-row update.
- [ ] Show how a DDL trigger could prevent `DROP TABLE` and what the caller sees.
- [ ] Find every trigger in an unfamiliar database and view one's definition.
- [ ] Explain why SQL Server has no `BEFORE` trigger and what you use instead.
<!-- explainer:end -->

## Practice questions

- [Basic] [What are the three main timing options for a DML trigger and what does each one mean?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Triggers/Q-43753_What_are_the_three_main_timing_options_for_a_DML_trigger_and.md>)
- [Basic] [On which DML operations can you define a trigger in SQL?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Triggers/Q-43766_On_which_DML_operations_can_you_define_a_trigger_in_SQL.md>)
- [Basic] [What is the difference between a DML trigger and a DDL trigger?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Triggers/Q-43789_What_is_the_difference_between_a_DML_trigger_and_a_DDL_trigg.md>)
- [Basic] [How do triggers show up in a database schema, and how would a new developer discover them?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Triggers/Q-43838_How_do_triggers_show_up_in_a_database_schema_and_how_would_a.md>)
- [Intermediate] [What could you use a DDL trigger for, such as auditing schema changes?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Triggers/Q-43790_What_could_you_use_a_DDL_trigger_for_such_as_auditing_schema.md>)
- [Deep] [What is the difference between synchronous trigger logic and asynchronous processing via a queue?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Triggers/Q-43798_What_is_the_difference_between_synchronous_trigger_logic_and.md>)
- [Deep] [Why would an AFTER trigger be unable to change the values that were just inserted into its own table?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Triggers/Q-43819_Why_would_an_AFTER_trigger_be_unable_to_change_the_values_th.md>)
- [Deep] [What is a compound trigger in Oracle and what problem does it help solve?](<../../../PRACTICE/01_Databases_and_SQL/Database_Objects/Triggers/Q-43831_What_is_a_compound_trigger_in_Oracle_and_what_problem_does_i.md>)

## Related concepts

- [Trigger Types](<../../../TOOLS/Oracle/PL-SQL/Trigger_Types.md>) *(PL-SQL)*
- [Dynamic SQL](<../../Database_Internals/DBMS_Concepts/Dynamic_SQL.md>) *(DBMS Concepts)*
- [DML Statements](<../../SQL_Language/SQL_Command_Types/DML_Statements.md>) *(SQL Command Types)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 7: Dynamic SQL, Triggers, and Table-Valued Functions (TVFs) › Using AFTER and INSTEAD OF Triggers
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 7: Dynamic SQL, Triggers, and Table-Valued Functions (TVFs) › AFTER Triggers
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 7: Dynamic SQL, Triggers, and Table-Valued Functions (TVFs) › INSTEAD OF Triggers
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 7: Dynamic SQL, Triggers, and Table-Valued Functions (TVFs) › Differences between AFTER and INSTEAD OF Triggers
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Programmable objects
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — User-Defined Data Types › Triggers
- *Cracking the Data Engineering Interview* — Kedeisha Bryan, Taamir Ransome — Must-know advanced database concepts › Triggers
- *Mapping Data Flows in Azure Data Factory* — Mark Kromer — Basics of CICD and Pipeline Scheduling › Custom Events Trigger
- *SQL Server Tutorial* (YouTube series) — Part 43: DML triggers in sql server ([watch](https://www.youtube.com/watch?v=k0S4P-a6d5w))
- *SQL Server Tutorial* (YouTube series) — Part 44: After update trigger ([watch](https://www.youtube.com/watch?v=P_BREQy6bOo))
- *SQL Server Tutorial* (YouTube series) — Part 45: Instead of insert trigger ([watch](https://www.youtube.com/watch?v=MseKoztMpoo))
- *SQL Server Tutorial* (YouTube series) — Part 46: Instead of update triggers in sql server - Part 46.avi ([watch](https://www.youtube.com/watch?v=pkQk1Z834Rc))

