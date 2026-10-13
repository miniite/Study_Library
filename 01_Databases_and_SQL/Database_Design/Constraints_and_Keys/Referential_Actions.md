---
id: "01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Referential_Actions"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Database_Design/Constraints_and_Keys"
tools: []
---

# Referential Actions

**L4 Advanced** · [↑ Constraints and Keys](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Referential actions (NO ACTION/RESTRICT, CASCADE, SET NULL, SET DEFAULT) define what the database does to child rows when the parent row they reference is deleted or its key is updated.

### 1. The big picture
A foreign key guarantees children have parents. The open question is *what should happen when someone deletes the parent?* Options: refuse, delete the children too, or detach them. That policy is declared with `ON DELETE` and `ON UPDATE` on the foreign key. The choice is a business decision with data-loss and performance consequences: CASCADE is convenient and dangerous; NO ACTION is safe and forces explicit cleanup.

### 2. Core concepts and vocabulary
- **NO ACTION** — default. Reject the parent change if children exist (checked at end of statement).
- **RESTRICT** — like NO ACTION but checked immediately and cannot be deferred (PostgreSQL, MySQL; not in SQL Server).
- **CASCADE** — propagate: delete the children (ON DELETE) or update their FK value (ON UPDATE).
- **SET NULL** — set child FK column(s) to NULL (column must be nullable).
- **SET DEFAULT** — set child FK to its DEFAULT value (which must exist as a parent).
- **Cascade path** — chain of cascades across several tables.
- **Soft delete** — mark row inactive (`is_deleted`) instead of removing it.
- **Orphan / dangling reference** — child without parent (what actions prevent).

### 3. How it works, step by step
For `DELETE FROM parent WHERE id = 5`:
1. Engine finds child rows with FK = 5 (index probe, or scan if unindexed).
2. By action:
   - NO ACTION/RESTRICT: if any exist → error, nothing deleted.
   - CASCADE: delete those children (recursively applying *their* children's actions), then delete the parent.
   - SET NULL: update child FK to NULL, then delete parent.
   - SET DEFAULT: update child FK to default, then delete parent.
3. All in one statement/transaction: any failure rolls everything back.

| Action | Children after parent delete | Typical use |
|---|---|---|
| NO ACTION / RESTRICT | Delete blocked | Core business entities, audit data |
| CASCADE | Deleted with parent | Strongly owned parts: order → order lines |
| SET NULL | Kept, detached | Optional relation: employee → manager, ticket → assignee |
| SET DEFAULT | Reassigned to default parent | "Unassigned" / "Unknown" placeholder |

ON UPDATE CASCADE does the same for changes of the parent key; it only matters when keys change, which good surrogate keys never do.

### 4. Worked example
```sql
CREATE TABLE dbo.Orders (
    orderid INT PRIMARY KEY,
    custid  INT NOT NULL REFERENCES dbo.Customers(custid)          -- NO ACTION: keep history
);

CREATE TABLE dbo.OrderLines (
    orderid INT NOT NULL,
    lineno  INT NOT NULL,
    PRIMARY KEY (orderid, lineno),
    CONSTRAINT FK_Lines_Orders FOREIGN KEY (orderid)
        REFERENCES dbo.Orders(orderid) ON DELETE CASCADE           -- lines die with the order
);

CREATE TABLE dbo.Tickets (
    ticketid INT PRIMARY KEY,
    assignee INT NULL,
    CONSTRAINT FK_Tickets_Emp FOREIGN KEY (assignee)
        REFERENCES dbo.Employees(empid) ON DELETE SET NULL         -- ticket stays, unassigned
);
```
Scenario: `DELETE FROM dbo.Orders WHERE orderid = 7;` removes the order and its lines. `DELETE FROM dbo.Customers WHERE custid = 3;` fails if customer 3 has orders — which is the intent, so history isn't destroyed.

The accidental-deletion example: `Customers → Orders → OrderLines → Shipments → Invoices` all with CASCADE; deleting one customer erased years of invoices that finance needed. Fix: CASCADE only for owned detail rows, NO ACTION (or soft delete) up the chain.

### 5. Under the hood
- Cascades are executed by the engine as part of the DML statement: the plan for one `DELETE` includes extra operators deleting from child tables, each needing locks. A deep cascade of N tables and millions of rows is one large transaction: big transaction-log writes, long locks, lock escalation to table level, and potentially a 30-second application stall.
- Each level's child lookup uses the child's FK index; unindexed ⇒ table scan per cascade level.
- SQL Server forbids multiple cascade paths and cycles to the same table (error 1785 "may cause cycles or multiple cascade paths") because the final result could depend on order. Workarounds: use INSTEAD OF triggers or stored procedures, or choose NO ACTION on one path.
- Triggers on child tables fire during cascades; combined with auditing triggers this multiplies work.
- CASCADE and replication/CDC: a single parent delete generates many child delete events downstream.
- SET NULL requires nullable column; SET DEFAULT requires a valid default parent row, otherwise the FK check fails.

### 6. Performance and design implications
- **Index the child FK** in every case; cascade delete speed is dominated by that probe.
- Keep cascades shallow (1–2 levels) and bounded; for large deletes use batches (`DELETE TOP (5000) ... LOOP`) or archival/partition switching instead of a single cascading statement.
- Prefer soft deletes or archival tables for entities with legal/audit retention; use CASCADE for pure composition.
- ON UPDATE CASCADE on a clustered PK relocates rows and rewrites all index entries — avoid by using immutable surrogate keys.
- Measure: look at the delete plan for child "Clustered Index Delete" operators and logical reads; check log growth during a test delete.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ON DELETE NO ACTION` | Block parent delete | Default, important data |
| `ON DELETE RESTRICT` | Immediate block | MySQL/PostgreSQL semantics |
| `ON DELETE CASCADE` | Delete children | Owned detail rows |
| `ON DELETE SET NULL` | Detach children | Optional relationships |
| `ON DELETE SET DEFAULT` | Reassign to default | Placeholder parents |
| `ON UPDATE CASCADE` | Follow key changes | Rare natural-key cases |
| `DEFERRABLE INITIALLY DEFERRED` | Check at commit | Swapping keys, circular references (PG/Oracle) |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| CASCADE: child has no meaning without parent (lines, attachments) | Child holds independent business/audit value |
| SET NULL: relationship optional, child survives | Column must stay NOT NULL; null breaks reports |
| NO ACTION: protect history and force explicit decisions | Users need to delete frequently → provide a controlled procedure |
| Soft delete | Storage matters; unique keys must treat deleted rows specially |

### 9. Common mistakes
- Blanket `ON DELETE CASCADE` everywhere → one delete wipes a graph of data → apply case by case.
- Cascading across five tables on a hot table → 30-second lock, log explosion → batch deletes, archive, or restrict cascade depth.
- SET NULL on a NOT NULL column → DDL error → make nullable or choose another action.
- No index on FK child column → cascade scans → create it.
- Using ON UPDATE CASCADE to "fix" changing natural keys → heavy rewrites → surrogate keys.
- Expecting application delete order to matter with CASCADE → engine decides → know the actions.
- Forgetting triggers/CDC amplification → downstream floods → account for events.

### 10. 🎯 Interview kit
**30-second answer:** Referential actions define what happens to child rows when a parent row is deleted or its key updated. NO ACTION/RESTRICT blocks it, CASCADE deletes or updates the children, SET NULL detaches them, SET DEFAULT reassigns them.

**Structure a longer answer:** clarify the relationship (ownership vs reference) → actions with examples → risks (accidental data loss, long locks) → alternatives (soft delete, batching) → scenario.

**Likely follow-ups:**
- *CASCADE vs SET NULL?* → delete the dependents vs keep them detached.
- *RESTRICT vs NO ACTION?* → timing/deferrability differences.
- *Cascade chain locked the app 30 s — redesign?* → shallow cascades, NO ACTION up the chain, batched/archival deletes, index FKs, soft delete.
- *When ON UPDATE CASCADE?* → rarely, natural keys that change.
- *Multiple cascade paths error?* → SQL Server restriction; use NO ACTION/triggers.

**Signals interviewers look for:** risk awareness (data loss, locks), ownership reasoning, index and batching knowledge.

### 11. Check yourself
- [ ] Choose an action for order→lines, customer→orders, ticket→assignee and justify each.
- [ ] Explain why a five-table cascade can stall an application and how to redesign.
- [ ] Describe SQL Server's multiple cascade path error and two ways around it.
- [ ] Write a batched delete loop that avoids lock escalation.
<!-- explainer:end -->

## Practice questions

- [Intermediate] [What does ON DELETE CASCADE do on a foreign key?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10151_What_does_ON_DELETE_CASCADE_do_on_a_foreign_key.md>)
- [Intermediate] [What is the difference between ON DELETE CASCADE and ON DELETE SET NULL?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10152_What_is_the_difference_between_ON_DELETE_CASCADE_and_ON_DELE.md>)
- [Intermediate] [What does ON DELETE RESTRICT or NO ACTION do, and how do they differ from CASCADE?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10153_What_does_ON_DELETE_RESTRICT_or_NO_ACTION_do_and_how_do_they.md>)
- [Intermediate] [What does ON UPDATE CASCADE do and when is it useful?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10154_What_does_ON_UPDATE_CASCADE_do_and_when_is_it_useful.md>)
- [Deep] [Give a real-world example of when ON DELETE CASCADE could accidentally delete more data than intended.](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10211_Give_a_real-world_example_of_when_ON_DELETE_CASCADE_could_ac.md>)
- [Deep] [Why might you prefer ON DELETE SET NULL over ON DELETE CASCADE in some designs?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10213_Why_might_you_prefer_ON_DELETE_SET_NULL_over_ON_DELETE_CASCA.md>)
- [Deep] [Your ON DELETE CASCADE chain cascaded across five tables and locked the app for 30 seconds. How would you redesign to avoid runaway cascade deletes?](<../../../PRACTICE/01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Q-10049_Your_ON_DELETE_CASCADE_chain_cascaded_across_five_tables_and.md>)

## Related concepts

- [DROP and CASCADE](<../../SQL_Language/SQL_Command_Types/DROP_and_CASCADE.md>) *(SQL Command Types)*
- [Polymorphic Associations](<../Schema_Design/Polymorphic_Associations.md>) *(Schema Design)*
- [DELETE vs TRUNCATE](<../../SQL_Language/SQL_Command_Types/DELETE_vs_TRUNCATE.md>) *(SQL Command Types)*

