---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between ON DELETE CASCADE and ON DELETE SET NULL for a foreign key?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ Lock Granularity](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Lock_Granularity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

What the question is really asking: Explain the two referential action options on a foreign key that fire when the parent (referenced) row is deleted — ON DELETE CASCADE vs ON DELETE SET NULL — how each behaves, and, at an intermediate level, when to choose which, plus the caveats that bite people in production.

What the interviewer is probing for: Not just the definition, but judgment. Do you know that CASCADE propagates deletes down the graph (and can silently wipe out large subtrees, and even recurse), while SET NULL orphans children by nulling the FK (and therefore requires the FK column to be nullable)? Do you know the data-modeling implication — CASCADE says "the child cannot exist without the parent" (composition/ownership), SET NULL says "the child can exist independently" (weak association)? Can you reason about performance, triggers, auditability, and alternatives like RESTRICT/NO ACTION and SET DEFAULT?

#### Assumptions
/clarifications: I'll answer in standard SQL and flag MySQL/InnoDB and PostgreSQL specifics where they matter. I'll assume a concrete customers → orders (or departments → employees) model to ground the reasoning.

Plan: Recap what a referential action is → define both actions precisely → contrast with a worked example → give the decision framework (ownership vs association) → cover the gotchas (nullable requirement, multi-level cascade, self-referencing loops, triggers not firing, performance, auditing) → summarize.

Background: what a referential action is

A foreign key (FK) enforces referential integrity: a child row's FK value must match an existing parent key (or be NULL). The question is what should happen to the child rows when the referenced parent row is deleted or its key is updated. SQL lets you declare this with ON DELETE <action> (and ON UPDATE <action>). The options are: NO ACTION/RESTRICT (block the delete — the default), CASCADE, SET NULL, and SET DEFAULT. This question compares two of them for the delete case.

ON DELETE CASCADE

Behavior: when a parent row is deleted, the database automatically deletes all child rows that reference it — in the same statement, atomically. The delete "cascades" from parent to children.

Semantics / when to use: this models ownership / composition — the child cannot meaningfully exist without its parent. Classic cases:

order_items referencing orders: delete an order, its line items are meaningless → cascade.
comments referencing a post; sessions referencing a user; addresses owned by a user.

Effect: keeps the database clean of orphaned rows automatically; you don't have to manually delete children first.

ON DELETE SET NULL

Behavior: when a parent row is deleted, the child rows are kept, but their FK column is set to NULL. The children become "orphaned but retained" — they no longer point to any parent.

Semantics / when to use: this models a weak/optional association — the child can exist on its own; losing the parent just removes the link. Classic cases:

employees.manager_id referencing employees(id): a manager leaves, their reports shouldn't be deleted — set manager_id = NULL until reassigned.
orders.coupon_id referencing coupons: delete a coupon, keep the order but null the coupon reference.
Any "soft" reference where the child's own data is independently valuable.

Hard requirement: the FK column(s) must be nullable. If the column is NOT NULL, ON DELETE SET NULL is illegal — the DDL will error (or the runtime delete will fail). This is the number-one gotcha.

The core difference (say this)
	ON DELETE CASCADE	ON DELETE SET NULL
Child rows	Deleted	Kept
FK value	gone (row gone)	set to NULL
Data meaning	child owned by parent (composition)	child independently valid (weak link)
FK column	any	must be nullable
Risk	silent bulk/recursive deletes	orphaned rows carrying NULL FKs

Put crisply: CASCADE removes the children with the parent; SET NULL keeps the children but breaks their link. Choosing between them is a data-modeling decision about whether the child's existence depends on the parent.

Intermediate-level gotchas and judgment

Cascade is multi-level and can be huge. If A →(cascade) B →(cascade) C, deleting one A row can wipe out entire subtrees across three tables in one statement. On large graphs this can lock many rows and be slow, or surprise you by deleting far more than expected. Always know your cascade graph before deleting.

Self-referencing cascades. A table referencing itself (e.g., employees.manager_id → employees.id) with ON DELETE CASCADE deletes whole hierarchies; some engines disallow certain self-referential cascade/set-null combos or behave differently (SQL Server outright rejects cycles that could cause multiple cascade paths).

Application triggers may NOT fire per row as you'd expect. In MySQL/InnoDB, FK-driven cascades/set-nulls happen inside the storage engine and do not activate your AFTER DELETE/BEFORE DELETE triggers on the child table — a real trap if you rely on triggers for auditing. PostgreSQL runs referential actions via internal system triggers and does fire user triggers, but ordering/visibility differs. Don't assume your child-table triggers run during a cascade.

Auditing / soft-delete conflict. CASCADE hard-deletes rows — if you need an audit trail or "who deleted what," a physical cascade destroys evidence. Many production systems avoid ON DELETE CASCADE in favor of application-level soft deletes (is_deleted flag) precisely so history is retained. SET NULL retains the child row but loses the linkage, which can also complicate audits.

Performance & locking. Both actions require the child FK column to be indexed for the referential action to be efficient; without an index, a parent delete forces a full scan of the child table to find matching rows. Cascading deletes across millions of children can be a long, lock-heavy transaction — sometimes batching the delete in the app is safer.

NULLs downstream. After SET NULL, queries and joins on the child must handle the now-NULL FK; aggregations grouping by that FK will bucket orphans under NULL. Make sure downstream logic tolerates it.

Alternatives worth naming: ON DELETE RESTRICT/NO ACTION (block the parent delete if children exist — safest default, forces explicit cleanup), and ON DELETE SET DEFAULT (set the FK to a sentinel default parent, e.g., an "Unassigned" department — the default value must itself exist as a parent key).

Summary — what a strong answer emphasizes
Both are referential actions fired by deleting the parent; they express what happens to children.
CASCADE deletes the children — use for ownership/composition where a child is meaningless without its parent (order → order_items).
SET NULL keeps the children and nulls the FK — use for optional/weak links where the child stands alone (employee → manager); FK must be nullable.
Show judgment: watch multi-level/recursive cascades, index the FK, know that MySQL cascades bypass child triggers, prefer soft-delete/RESTRICT when auditability matters. That practitioner nuance is what an intermediate interviewer is grading.

#### Follow-up
probes: "What if the FK column is NOT NULL and you want SET NULL?" (illegal — must make it nullable or pick another action). "What's the default action?" (NO ACTION/RESTRICT). "How does CASCADE interact with triggers/soft deletes?"

**Comparison:**

```
Aspect	ON DELETE CASCADE	ON DELETE SET NULL	ON DELETE RESTRICT / NO ACTION
What happens to child	Deleted with parent	Kept; FK set to NULL	Parent delete blocked
FK column requirement	Any	Must be nullable	Any
Data model meaning	Ownership / composition	Optional / weak association	Explicit cleanup required
Orphan rows left?	No (children gone)	Yes (children with NULL FK)	No (nothing deleted)
Main risk	Silent bulk/recursive deletes; bypasses child triggers (MySQL)	NULLs to handle downstream	Deletes fail until children handled
Typical use	order → order_items, post → comments	employee → manager, order → coupon	financial/audit tables you must not auto-delete
```

**Code:**

```sql
-- Parent
CREATE TABLE customers (
    id   INT PRIMARY KEY,
    name VARCHAR(50)
);

-- CASCADE: order_items belong to an order; delete order -> delete its items
CREATE TABLE orders (
    id          INT PRIMARY KEY,
    customer_id INT NOT NULL,
    FOREIGN KEY (customer_id) REFERENCES customers(id)
        ON DELETE CASCADE            -- deleting a customer deletes their orders
);
CREATE INDEX idx_orders_customer ON orders(customer_id);   -- index the FK

-- SET NULL: an order keeps existing even if its coupon is deleted
CREATE TABLE orders_with_coupon (
    id        INT PRIMARY KEY,
    coupon_id INT NULL,             -- MUST be nullable for SET NULL
    FOREIGN KEY (coupon_id) REFERENCES coupons(id)
        ON DELETE SET NULL
);

-- ILLEGAL: SET NULL on a NOT NULL column -> DDL error
-- coupon_id INT NOT NULL ... ON DELETE SET NULL   -- rejected by the DBMS
```

**Example input / output:**

```

Setup:

CODE
Copy
customers                orders (ON DELETE CASCADE, FK customer_id -> customers.id)
 id | name                id | customer_id
----+------               ----+------------
  1 | Asha                101 |     1
  2 | Ravi                102 |     1
                          103 |     2
SQL
Copy
DELETE FROM customers WHERE id = 1;

Result — orders 101 and 102 are also deleted (cascade):

CODE
Copy
customers                orders
 id | name                id | customer_id
----+------               ----+------------
  2 | Ravi                103 |     2

Now the SET NULL case:

CODE
Copy
coupons                  orders_with_coupon (ON DELETE SET NULL)
 id | code                id | coupon_id
----+------               ----+----------
 10 | SAVE10              201 |    10
                          202 |    10
SQL
Copy
DELETE FROM coupons WHERE id = 10;

Result — orders remain, their coupon_id becomes NULL:

CODE
Copy
orders_with_coupon
 id | coupon_id
----+----------
201 |   NULL
202 |   NULL
```

**Flow:**

1. CASCADE
2. yes
3. no
4. SET NULL
5. no
6. yes
7. RESTRICT / NO ACTION
8. yes
9. no
10. DELETE parent row
11. ON DELETE action?
12. Delete all child rowsreferencing it
13. Children have their owncascades?
14. Recurse: delete grandchildren...
15. Done (subtree removed)
16. FK column nullable?
17. ERROR / illegal DDL
18. Keep children; set FK = NULL
19. Any child rows exist?
20. Block parent DELETE (error)
21. Parent deleted

