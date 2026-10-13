---
type: "practice"
difficulty: "Intermediate"
---

# What does ON DELETE CASCADE do, and what are the risks of using it?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Foreign Keys and Referential Integrity](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Foreign_Keys_and_Referential_Integrity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Triage

The question is really two questions bundled together: (1) mechanically, what does the referential action ON DELETE CASCADE do on a foreign key, and (2) judgment — why do experienced engineers treat it with caution in production? At the "intermediate experience" level the interviewer is not just checking that you know the definition; they want to see that you understand the blast radius of a cascade, the performance and locking implications, and when you would deliberately choose it over alternatives like RESTRICT, SET NULL, or soft deletes.

Clarifying assumptions: I'll assume a relational database (PostgreSQL/MySQL InnoDB semantics, which behave the same for the core question), foreign keys declared with REFERENCES, and that "risks" means real operational risks, not syntax gotchas.

Plan: define the mechanic, show it with a concrete parent/child schema, enumerate the referential-action alternatives, then walk through the four concrete risks (accidental mass deletion, hidden performance/locking cost, deep multi-level cascades, and interaction with app-layer/soft-delete logic), and finish with when to use it vs. not.

What ON DELETE CASCADE does

A foreign key constraint says "a row in the child table points to a row in the parent table." ON DELETE CASCADE is a referential action attached to that constraint. It answers the question: "when the parent row is deleted, what happens to the child rows that reference it?" With CASCADE, the database automatically deletes the referencing child rows in the same transaction. You delete one customers row, and every orders row for that customer disappears too — and if order_items has ON DELETE CASCADE to orders, those get deleted as well. It cascades down the graph.

The alternatives you should name:

ON DELETE RESTRICT / NO ACTION (the default in most engines): the delete of the parent is blocked with a foreign-key violation error if any child rows still reference it. Safe by default — you must clean up children first.
ON DELETE SET NULL: child rows survive, but their FK column is set to NULL (requires the column to be nullable). Good for "orphan but keep" cases, e.g. a deleted user's audit-log rows keep the log, user_id becomes NULL.
ON DELETE SET DEFAULT: sets the FK to its column default (less common; the default must itself be a valid parent key or NULL).

The key mental model: CASCADE moves the responsibility for maintaining referential integrity from your application code into the database engine, and it does so silently and automatically.

The risks

1. Accidental mass deletion — the "one row" that wasn't one row. The single biggest risk. A developer or an operator runs DELETE FROM customers WHERE id = 42 expecting to remove one row, but that customer has 50,000 orders and 400,000 order-items, all of which vanish in the same transaction. There is no prompt, no confirmation. A cascade turns a small statement into a large, unbounded one. Worse: DELETE FROM customers WHERE region = 'X' could nuke millions of dependent rows. Because the deletion is implicit, it also often escapes code review — the destructive intent lives in the DDL, not the DML.

2. Performance and locking. Cascaded deletes are not free. The engine must find and delete every dependent row, which means index lookups on each child FK (so the FK columns must be indexed — an unindexed child FK makes each parent delete a full table scan of the child). All of this happens in one transaction, holding locks on all affected rows for the whole duration. On a hot OLTP table this can cause lock contention, long-running transactions, replication lag, and in extreme cases lock-wait timeouts or deadlocks. A "quick delete" can stall the system.

3. Deep / wide cascade chains are hard to reason about. In a rich schema the cascade graph can be several levels deep and fan out widely. It's genuinely hard for an engineer to look at one DELETE and predict the full set of tables and rows it will touch. This also makes the effect of a data-fix script unpredictable and complicates backups/point-in-time reasoning.

4. It conflicts with soft-deletes, audit, and business logic. Many production systems never hard-delete important data — they soft-delete (deleted_at timestamp) for audit, compliance (GDPR "right to be forgotten" is the exception), analytics, or undo. ON DELETE CASCADE is a hard delete; if you rely on it, you lose that data permanently and bypass any application-level hooks (emitting events, updating aggregates/counters, invalidating caches, writing to a Kafka outbox). Data that should have been retained for a legal hold can be silently erased.

5. Circular or self-referential cascades can be rejected by the engine or produce surprising behavior; MySQL, for instance, restricts cascading updates/deletes across some circular FK setups.

When to use it vs. not

ON DELETE CASCADE is genuinely the right tool for strongly-owned, no-independent-life child data: order_items truly belong to an order; a post's attachments/tags belong to the post; a tenant's config rows belong to the tenant. When the child has no meaning without the parent and you want a hard delete, cascade is correct, correct-by-construction (integrity can't drift), and removes error-prone cleanup code.

Avoid it — prefer RESTRICT plus explicit, batched application-level deletion — when: children are valuable/auditable, the volume is large (delete in bounded batches to avoid long locks), you need side effects on delete, or you use soft-deletes. A common senior pattern is RESTRICT on the FK so an accidental parent delete fails loudly, and a deliberate service method that soft-deletes or batch-deletes children with proper events and observability.

Summary — what a strong answer emphasizes

State the mechanic crisply (auto-delete of referencing rows within the same transaction), contrast it with RESTRICT/SET NULL, and then spend most of your time on judgment: the blast-radius/accidental-mass-delete risk, the need to index child FK columns, the single-transaction locking cost, and the clash with soft-delete/audit/event-driven systems. Conclude with a clear rule: use cascade for tightly-owned child data you truly want hard-deleted; default to RESTRICT + explicit batched deletes for everything valuable.

Common follow-ups: "Do you need an index on the child FK?" (yes, for cascade performance). "How do you delete 10M dependent rows without locking the table?" (batch in the app layer, not one cascade). "How does this interact with ON DELETE SET NULL?" (children survive, FK nulled).

**Comparison:**

```
Referential action	Parent delete	Child rows	Typical use
CASCADE	Succeeds	Deleted automatically (same txn)	Tightly-owned children (order_items, tenant config) you want hard-deleted
RESTRICT / NO ACTION (default)	Blocked if children exist	Untouched	Safe default; force explicit cleanup / loud failure
SET NULL	Succeeds	Kept, FK set to NULL	Orphan-but-keep (audit logs after user removal)
SET DEFAULT	Succeeds	FK set to column default	Rare; default must be a valid key/NULL
Risk	Why it bites	Mitigation
Accidental mass delete	One DELETE silently removes millions of dependent rows	Prefer RESTRICT; guard destructive ops; review DDL
Performance / locking	All cascades run in one txn; unindexed child FK = table scan	Index every child FK; batch large deletes in app layer
Deep cascade chains	Hard to predict full row set touched	Keep cascade graphs shallow; document ownership
Bypasses soft-delete/audit/events	Hard delete skips app hooks, loses auditable data	Use soft-delete + explicit service deletes with events
```

**Code:**

```sql
-- Parent / child with CASCADE: deleting a customer removes their orders,
-- and deleting an order removes its line items.
CREATE TABLE customers (
    id   BIGINT PRIMARY KEY,
    name TEXT NOT NULL
);

CREATE TABLE orders (
    id          BIGINT PRIMARY KEY,
    customer_id BIGINT NOT NULL,
    total_cents BIGINT NOT NULL,
    CONSTRAINT fk_orders_customer
        FOREIGN KEY (customer_id) REFERENCES customers(id)
        ON DELETE CASCADE
);
-- IMPORTANT: index the child FK column, or each cascade delete scans the table.
CREATE INDEX idx_orders_customer_id ON orders(customer_id);

CREATE TABLE order_items (
    id       BIGINT PRIMARY KEY,
    order_id BIGINT NOT NULL,
    sku      TEXT   NOT NULL,
    CONSTRAINT fk_items_order
        FOREIGN KEY (order_id) REFERENCES orders(id)
        ON DELETE CASCADE
);
CREATE INDEX idx_order_items_order_id ON order_items(order_id);

-- Alternatives you would name:
-- ON DELETE RESTRICT  -> blocks the parent delete if children exist (safe default)
-- ON DELETE SET NULL  -> keeps child rows, nulls the FK (column must be nullable)

-- Safer production pattern: RESTRICT + a deliberate, batched service delete.
ALTER TABLE orders
    DROP CONSTRAINT fk_orders_customer,
    ADD CONSTRAINT fk_orders_customer
        FOREIGN KEY (customer_id) REFERENCES customers(id)
        ON DELETE RESTRICT;  -- accidental parent delete now fails loudly
```

**Example input / output:**

```text
-- Seed
customers:    (1, 'Acme')
orders:       (100, 1, 5000), (101, 1, 3000)
order_items:  (900, 100, 'SKU-A'), (901, 100, 'SKU-B'), (902, 101, 'SKU-C')

-- With ON DELETE CASCADE on both FKs:
SQL> DELETE FROM customers WHERE id = 1;
Effect (all in ONE transaction):
  - customers row 1 deleted
  - orders rows 100, 101 deleted (cascade)
  - order_items rows 900, 901, 902 deleted (cascade)
Result: 1 statement removed 1 + 2 + 3 = 6 rows.  <-- the blast radius.

-- With ON DELETE RESTRICT instead:
SQL> DELETE FROM customers WHERE id = 1;
ERROR: update or delete on table "customers" violates foreign key
       constraint "fk_orders_customer" on table "orders"
DETAIL: Key (id)=(1) is still referenced from table "orders".
Result: 0 rows deleted; you must remove children first (safe by default).
```

**Flow:**

1. CASCADE
2. CASCADE
3. RESTRICT
4. DELETE FROM customersWHERE id = 42
5. FK action onorders.customer_id?
6. Find all orders for customer 42via idx_orders_customer_id
7. FK action onorder_items.order_id?
8. Delete matching order_items
9. Delete matching orders
10. Delete customer row
11. Child rows exist -> ERROR,parent delete blocked
12. All deletions committed inONE transaction / locks heldthroughout

