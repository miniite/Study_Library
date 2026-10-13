---
id: "TOOLS/Oracle/PL-SQL/Trigger_Types"
level: 4
type: "concept"
topic: "TOOLS/Oracle/PL-SQL"
tools: ["Oracle"]
---

# Trigger Types

**L4 Advanced** · [↑ PL-SQL](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A trigger is a stored PL/SQL block the database fires automatically on an event; its *timing* (BEFORE/AFTER/INSTEAD OF), *level* (row/statement) and *event* (INSERT/UPDATE/DELETE, DDL, system) decide what it can see and do, and compound triggers solve the classic mutating-table problem.

### 1. The big picture
Triggers let the database itself enforce rules, record history and derive values regardless of which application changed the data. That power has costs: they run invisibly inside every matching statement, they can fire in unexpected orders, they add to transaction time, and they are hard to test. A mature approach uses triggers for a small set of jobs (auditing, defaulting columns like surrogate keys or timestamps, protecting integrity that cannot be declared) and prefers constraints, `MERGE`, or API procedures for business logic.

### 2. Core concepts and vocabulary
- **Triggering event** — `INSERT`, `UPDATE [OF col]`, `DELETE` on a table or view; DDL events (`CREATE`, `ALTER`, `DROP`); database events (`LOGON`, `LOGOFF`, `STARTUP`, `SERVERERROR`).
- **Timing** — `BEFORE` (before the change; can modify `:NEW`), `AFTER` (after the change; row data final), `INSTEAD OF` (replaces the DML on a view).
- **Level** — **statement-level** (fires once per statement, even if zero rows) vs. **row-level** (`FOR EACH ROW`, fires per affected row).
- **`:OLD` / `:NEW`** — pseudorecords with the row's previous and new values (not available at statement level; `:OLD` is NULL for `INSERT`, `:NEW` NULL for `DELETE`).
- **`WHEN (condition)`** — row-trigger filter evaluated before the body (written without the colon: `WHEN (NEW.status = 'X')`).
- **Conditional predicates** — `INSERTING`, `UPDATING`, `UPDATING('COL')`, `DELETING` in one multi-event trigger.
- **Compound trigger** — one trigger with up to four timing sections (`BEFORE STATEMENT`, `BEFORE EACH ROW`, `AFTER EACH ROW`, `AFTER STATEMENT`) sharing declarations and state.
- **Mutating table** — the table currently being changed by the statement that fired a row-level trigger; reading or modifying it from that trigger raises `ORA-04091`.
- **`FOLLOWS`** — clause that orders triggers of the same type and event.

### 3. How it works, step by step
Order of events for `UPDATE emp SET sal = sal*1.1 WHERE deptno = 10` with all four trigger kinds present:
1. **BEFORE STATEMENT** triggers fire once.
2. For each affected row:
   a. **BEFORE EACH ROW** triggers fire (may change `:NEW`).
   b. The row is changed (locks taken, undo/redo written).
   c. **AFTER EACH ROW** triggers fire.
3. Constraints deferred to statement end are checked.
4. **AFTER STATEMENT** triggers fire once.

Compound triggers run these same phases in one unit:
```sql
CREATE OR REPLACE TRIGGER emp_sal_audit
FOR UPDATE OF sal ON emp
COMPOUND TRIGGER
  TYPE t_chg IS TABLE OF emp_audit%ROWTYPE INDEX BY PLS_INTEGER;
  g_chg t_chg;                                   -- shared state across the phases

  BEFORE STATEMENT IS BEGIN g_chg.DELETE; END BEFORE STATEMENT;

  AFTER EACH ROW IS
  BEGIN
    g_chg(g_chg.COUNT+1).empno   := :NEW.empno;
    g_chg(g_chg.COUNT).old_sal   := :OLD.sal;
    g_chg(g_chg.COUNT).new_sal   := :NEW.sal;
    g_chg(g_chg.COUNT).changed_at:= SYSTIMESTAMP;
  END AFTER EACH ROW;

  AFTER STATEMENT IS
  BEGIN
    FORALL i IN 1..g_chg.COUNT INSERT INTO emp_audit VALUES g_chg(i);   -- one bulk insert
  END AFTER STATEMENT;
END emp_sal_audit;
/
```
The collection is filled per row and flushed once, in the statement phase, where querying the table is safe (no mutating-table error in the AFTER STATEMENT section).

Commit/rollback: a trigger runs *inside* the transaction of the triggering statement, so it **cannot** `COMMIT` or `ROLLBACK` (`ORA-04092`) — it becomes part of the same atomic unit; if the trigger raises an error, the triggering statement fails and is rolled back. Autonomous transactions are the escape hatch (for logging that must survive rollback), used sparingly.

### 4. Worked example
```sql
-- Row-level BEFORE trigger: defaults and derived values
CREATE OR REPLACE TRIGGER trg_orders_bi
BEFORE INSERT OR UPDATE ON orders
FOR EACH ROW
BEGIN
  IF INSERTING THEN
     :NEW.order_id   := NVL(:NEW.order_id, orders_seq.NEXTVAL);
     :NEW.created_at := SYSTIMESTAMP;
  END IF;
  :NEW.updated_at := SYSTIMESTAMP;
  IF UPDATING('STATUS') AND :OLD.status = 'CLOSED' AND :NEW.status <> 'CLOSED' THEN
     RAISE_APPLICATION_ERROR(-20020, 'A closed order cannot be reopened');
  END IF;
END;
/
-- Row trigger with WHEN: fires only for big changes
CREATE OR REPLACE TRIGGER trg_big_change
AFTER UPDATE OF amount ON payments
FOR EACH ROW
WHEN (NEW.amount > OLD.amount * 2)
BEGIN
  INSERT INTO payment_alerts(payment_id, old_amt, new_amt) VALUES (:NEW.id, :OLD.amount, :NEW.amount);
END;
/
-- INSTEAD OF trigger: make a join view updatable
CREATE OR REPLACE TRIGGER trg_v_emp_dept_ii
INSTEAD OF INSERT ON v_emp_dept
FOR EACH ROW
BEGIN
  INSERT INTO emp(empno, ename, deptno) VALUES (:NEW.empno, :NEW.ename, :NEW.deptno);
END;
/
```
Mutating table, the typical error:
```sql
-- AFTER INSERT ... FOR EACH ROW on emp that does SELECT COUNT(*) FROM emp  ->  ORA-04091: table EMP is mutating
```
Cures: (1) compound trigger collecting keys per row and querying in `AFTER STATEMENT`; (2) rethink as a constraint/aggregate maintained elsewhere; (3) pre-11g: package-variable arrays plus an AFTER STATEMENT trigger.

Managing triggers:
```sql
ALTER TRIGGER trg_orders_bi DISABLE;        ALTER TRIGGER trg_orders_bi ENABLE;
ALTER TABLE orders DISABLE ALL TRIGGERS;    DROP TRIGGER trg_orders_bi;

SELECT trigger_name, trigger_type, triggering_event, status, when_clause
FROM   user_triggers WHERE table_name = 'ORDERS';
```
Ordering: `CREATE TRIGGER t2 … FOLLOWS t1` forces `t2` to run after `t1` for the same event/timing (11g+); without it the order is undefined.

### 5. Under the hood
- **Storage and execution.** Trigger source is stored in the dictionary (`USER_TRIGGERS`, `USER_SOURCE`) and compiled like any PL/SQL unit; the row trigger body runs inside the DML's execution, so cost is multiplied by affected rows. Each row fire costs a PL/SQL frame setup plus any SQL in it (recursive SQL, with its own parse/exec/fetch).
- **Row vs. statement locking behavior.** A row trigger sees the row it is processing already locked; any additional DML it issues creates its own undo and redo in the *same* transaction. A trigger on `INSERT` that inserts into an audit table doubles redo volume and may introduce index contention on the audit table.
- **Mutating-table mechanics.** Oracle guarantees a consistent view of the table while a statement modifies it; reading the table mid-statement from a row trigger could expose a half-updated state, so it is prohibited at runtime (`ORA-04091`). Statement-level triggers and compound `AFTER STATEMENT` sections run after the change completes, so reading is allowed.
- **Recursion.** DML inside a trigger can fire other triggers (cascading); limited by `ORA-00036` (maximum recursive SQL levels) and by your own discipline; use `UPDATING('COL')` and `WHEN` clauses to avoid ping-pong.
- **Bulk operations.** `INSERT /*+ APPEND */` direct-path loads and `SQL*Loader` direct path bypass or disable row triggers (direct path will not run triggers and requires them to be disabled in some cases): do not rely on triggers for data from bulk loads.
- **Invalidations.** Triggers depend on the tables they reference; DDL on those tables invalidates them (recompile on next use). A trigger with compile errors causes DML on the table to fail with `ORA-04098` — a deployment hazard.
- **Autonomous transactions** in triggers commit independently and so are not rolled back with the statement: valid for audit logging of *attempts*, dangerous for business state.

### 6. Performance and design implications
- Keep trigger bodies short and set-based; avoid queries per row. Use `WHEN` to skip unneeded firings, `UPDATE OF col` to restrict to changed columns.
- Prefer a compound trigger when a row trigger must aggregate or look up the table itself.
- Sequence/identity defaults: since 12c use `GENERATED ... AS IDENTITY` or `DEFAULT seq.NEXTVAL` instead of a before-insert trigger.
- Auditing: consider built-in Fine-Grained Auditing (`DBMS_FGA`), unified auditing, Flashback Data Archive, or temporal validity before writing custom triggers.
- When the rule is "this write must also do X", an API package (`orders_api.close_order`) that does both explicitly is easier to test, debug and trace than a hidden trigger. Triggers still make sense when many unrelated clients write to the table and you must guarantee the effect.
- Monitor with `V$SQL`/AWR for recursive statements that originate in trigger code (`SQL` text with the trigger's table), or `DBA_SOURCE` grep.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE [OR REPLACE] TRIGGER … BEFORE|AFTER|INSTEAD OF … [FOR EACH ROW] [WHEN (…)]` | Define | Always specify timing/level explicitly |
| `COMPOUND TRIGGER` | Four phase sections with shared state | Avoid mutating table, bulk audit |
| `FOLLOWS other_trigger` | Deterministic order | Multiple triggers same event |
| `ALTER TRIGGER … ENABLE|DISABLE` / `ALTER TABLE … DISABLE ALL TRIGGERS` | Switch off during loads/maintenance | Data migration |
| `USER_TRIGGERS`, `USER_TRIGGER_COLS`, `USER_ERRORS` | Inventory and compile errors | Impact analysis |
| `INSERTING / UPDATING('C') / DELETING` | Event tests | Multi-event trigger |
| `PRAGMA AUTONOMOUS_TRANSACTION` | Independent commit | Audit logs of failed attempts |
| `DBMS_FGA`, unified auditing | Declarative auditing | Instead of custom triggers |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| BEFORE row trigger | Setting defaults/derived columns; simple validation not expressible as `CHECK` |
| AFTER row trigger | Writing audit rows based on final values |
| Statement trigger | Rule applies once per statement (e.g., block DML outside business hours) |
| INSTEAD OF | Making complex views updatable |
| Compound trigger | Bulk-processing row changes at statement end, mutating table avoidance |
| Constraint / `MERGE` / API procedure | Business logic with multi-table orchestration, external calls, or needing visibility and testing |

### 9. Common mistakes
- `COMMIT` or `ROLLBACK` in a trigger → `ORA-04092` → remove; let the caller commit.
- Reading the same table in a row trigger → `ORA-04091` → compound trigger.
- Multiple triggers with hidden ordering dependencies → intermittent bugs → `FOLLOWS` or consolidate into one.
- Triggers that call other DML-heavy procedures per row → bulk loads crawl → defer or batch.
- Using `:NEW` in a statement-level trigger → compile error → add `FOR EACH ROW` or aggregate at statement level.
- Not testing `UPDATE` that changes zero rows → statement triggers still fire → handle it.
- Disabling triggers for migration and forgetting to re-enable → integrity holes → script ENABLE in the same job and verify via `USER_TRIGGERS.STATUS`.

### 10. 🎯 Interview kit
**30-second answer:** "A trigger is a stored program that fires automatically on DML, DDL or system events. I choose timing (BEFORE to change `:NEW`, AFTER for audit, INSTEAD OF for views) and level (row vs. statement). They run in the caller's transaction so they can't commit, and row triggers can't read their own table (mutating table) — compound triggers fix that. I use them sparingly: auditing and defaults; business logic belongs in an API package or constraints."

**Structure a longer answer:** definition → timing/level matrix with firing order → `:OLD/:NEW` and `WHEN` → transaction semantics and mutating table → alternatives and maintainability argument.

**Likely follow-ups:**
- *Row vs. statement level?* → per row with `:OLD/:NEW` vs. once per statement.
- *Compound trigger problem solved?* → mutating table and bulk handling via shared state across phases.
- *Can a trigger commit?* → no (`ORA-04092`), autonomous transaction exception.
- *INSTEAD OF?* → replaces DML on a view.
- *How to order triggers?* → `FOLLOWS`.
- *When is a trigger the wrong tool?* → hidden side effects, hard to test, performance multiplied by rows; prefer constraint, `MERGE`, or API.

**Signals interviewers look for:** correct firing order, knowing the transaction limits, naming concrete alternatives with a maintainability argument.

### 11. Check yourself
- [ ] List the order in which four trigger kinds fire for one `UPDATE` touching three rows.
- [ ] Reproduce and fix `ORA-04091`.
- [ ] Write a `WHEN` clause correctly (no colon) and explain why it is cheaper than an `IF` inside the body.
- [ ] Explain why a trigger cannot `COMMIT` and when an autonomous transaction is justified.
- [ ] Use `FOLLOWS` to make two triggers deterministic.
- [ ] Replace a sequence-populating trigger with identity/default syntax.
<!-- explainer:end -->

## Implementations (tools)

- [Oracle](<../_index.md>)

## Practice questions

- [Basic] [What is a database trigger and when does it fire?](<../../../PRACTICE/TOOLS/Oracle/PL-SQL/Q-35729_What_is_a_database_trigger_and_when_does_it_fire.md>)
- [Intermediate] [What is the difference between a row-level trigger and a statement-level trigger?](<../../../PRACTICE/TOOLS/Oracle/PL-SQL/Q-35731_What_is_the_difference_between_a_row-level_trigger_and_a_sta.md>)
- [Intermediate] [How do you enable or disable a trigger and how do you drop it?](<../../../PRACTICE/TOOLS/Oracle/PL-SQL/Q-35788_How_do_you_enable_or_disable_a_trigger_and_how_do_you_drop_i.md>)
- [Intermediate] [What is the WHEN condition in a trigger and how does it restrict trigger execution?](<../../../PRACTICE/TOOLS/Oracle/PL-SQL/Q-35789_What_is_the_WHEN_condition_in_a_trigger_and_how_does_it_rest.md>)
- [Intermediate] [How do you find the list of triggers on a table using the USER_TRIGGERS data dictionary view?](<../../../PRACTICE/TOOLS/Oracle/PL-SQL/Q-35790_How_do_you_find_the_list_of_triggers_on_a_table_using_the_US.md>)
- [Deep] [Can you COMMIT or ROLLBACK inside a trigger? Why or why not?](<../../../PRACTICE/TOOLS/Oracle/PL-SQL/Q-35747_Can_you_COMMIT_or_ROLLBACK_inside_a_trigger_Why_or_why_not.md>)
- [Deep] [What is a compound trigger and what problem does it help solve?](<../../../PRACTICE/TOOLS/Oracle/PL-SQL/Q-35786_What_is_a_compound_trigger_and_what_problem_does_it_help_sol.md>)
- [Deep] [What is an INSTEAD OF trigger and on which object type is it typically used?](<../../../PRACTICE/TOOLS/Oracle/PL-SQL/Q-35787_What_is_an_INSTEAD_OF_trigger_and_on_which_object_type_is_it.md>)

## Related concepts

- [Triggers Behavior Performance and Alternatives](<../../../01_Databases_and_SQL/Database_Objects/Triggers/Triggers_Behavior_Performance_and_Alternatives.md>) *(Triggers)*
- [DML vs DDL Triggers](<../../../01_Databases_and_SQL/Database_Objects/Triggers/DML_vs_DDL_Triggers.md>) *(Triggers)*

