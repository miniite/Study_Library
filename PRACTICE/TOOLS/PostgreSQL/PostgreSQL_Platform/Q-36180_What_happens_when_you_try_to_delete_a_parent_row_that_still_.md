---
type: "practice"
difficulty: "Intermediate"
---

# What happens when you try to delete a parent row that still has child rows referencing it via a foreign key?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Adding Constraints Online](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Adding_Constraints_Online.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When attempting to delete a parent row in PostgreSQL that has related child rows, PostgreSQL enforces referential integrity constraints by preventing such an operation from occurring unless specified otherwise. This behavior is crucial for maintaining data consistency and avoiding orphaned records.

#### Triaging the Question

The question is essentially asking about how PostgreSQL handles foreign key constraints during deletion operations. Specifically, it probes your understanding of:

Referential Integrity in databases.
Foreign Key Constraints.
Error handling mechanisms in SQL.

We'll assume that we are working with a typical schema where there's a parent table and a child table related via a foreign key. The child rows must be deleted before the parent row can be removed to avoid integrity violations.

#### First Principles Solution

Let’s break down the scenario step-by-step:

Schema Definition:

Parent Table (e.g., orders)
SQL
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  order_number INT NOT NULL,
  customer_id INT NOT NULL REFERENCES customers(id)
);
Child Table (e.g., order_items)
SQL
CREATE TABLE order_items (
  id SERIAL PRIMARY KEY,
  order_id INT NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
  item_name VARCHAR(255) NOT NULL
);

Scenario:

Suppose we have an orders table with a foreign key referencing the customers table.
There are order_items that reference rows in the orders table.

Attempt to Delete:

If you try to delete a row from the parent table (e.g., DELETE FROM orders WHERE order_id = 1;) and there are still child records referencing this row, PostgreSQL will raise an error:
SQL
ERROR: update or delete on table "orders" violates foreign key constraint "order_items_order_id_fkey"
DETAIL: Key (id)=(1) is still referenced from table "order_items".
Handling the Scenario

To successfully delete a parent row with child rows, you can:

Delete Child Rows First: Execute DELETE FROM order_items WHERE order_id = 1; before deleting the parent.
Set ON DELETE CASCADE: Modify the foreign key constraint to automatically delete child records when their parent is deleted. This involves altering the schema:
SQL
ALTER TABLE order_items ADD CONSTRAINT order_items_order_id_fkey FOREIGN KEY (order_id) REFERENCES orders(id) ON DELETE CASCADE;
With this setup, deleting a row from orders will cascade and delete related rows in order_items.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, PostgreSQL enforces referential integrity by preventing the deletion of a parent row that has dependent child rows unless explicitly managed. The common mistake is forgetting to handle or plan for cascading deletions when designing schemas.

#### Follow-up
questions might include:

How would you manage this situation in a multi-table transaction?
What are other strategies to maintain data consistency without relying on foreign key constraints?

**Code:**

```sql
-- Example of ON DELETE CASCADE setup
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  order_number INT NOT NULL,
  customer_id INT NOT NULL REFERENCES customers(id)
);

CREATE TABLE order_items (
  id SERIAL PRIMARY KEY,
  order_id INT NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
  item_name VARCHAR(255) NOT NULL
);
```

**Example input / output:**

```text
-- Attempting to delete a parent row with child rows
DELETE FROM orders WHERE order_id = 1; -- Fails

-- Deleting child rows first
DELETE FROM order_items WHERE order_id = 1;

-- Re-attempting deletion after cascade
DELETE FROM orders WHERE order_id = 1;
```

