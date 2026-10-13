---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between ON DELETE CASCADE, ON DELETE SET NULL, and ON DELETE RESTRICT?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Constraint Definitions](<../../../../TOOLS/MySQL/MySQL_Platform/Constraint_Definitions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In MySQL, ON DELETE CASCADE, ON DELETE SET NULL, and ON DELETE RESTRICT are different behaviors defined in foreign key constraints to manage what happens when a referenced row is deleted. Let's break down each of these options:

TL;DR:
ON DELETE CASCADE cascades the delete operation to related rows.
ON DELETE SET NULL sets the foreign key column to null for related rows.
ON DELETE RESTRICT prevents the deletion if there are any related rows.

#### Triaging the Question

The question is essentially asking you to understand how different behaviors affect database integrity when a referenced row is deleted. The interviewer might be probing your understanding of database normalization, referential integrity, and potential impacts on data consistency.

#### Assumptions
You are familiar with basic SQL operations.
You have an understanding of primary keys and foreign keys in relational databases.
Step-by-Step Solution

ON DELETE CASCADE:

When a referenced row is deleted, all related rows in the referencing table(s) are also deleted automatically.
This ensures that there are no orphaned records in the child tables.

#### Example
SQL
ALTER TABLE orders ADD CONSTRAINT fk_customer_id FOREIGN KEY (customer_id)
  REFERENCES customers(id) ON DELETE CASCADE;
Deleting a customer will cascade and delete all associated orders.

ON DELETE SET NULL:

When a referenced row is deleted, the foreign key in the referencing table(s) is set to NULL.
This can be useful if you don’t want to remove related records but still want to maintain referential integrity.

#### Example
SQL
ALTER TABLE orders ADD CONSTRAINT fk_customer_id FOREIGN KEY (customer_id)
  REFERENCES customers(id) ON DELETE SET NULL;
Deleting a customer will set the customer_id in all associated orders to NULL.

ON DELETE RESTRICT:

This option prevents the deletion of the referenced row if there are any related records.
It ensures that referential integrity is maintained by not allowing deletions that would violate it.

#### Example
SQL
ALTER TABLE orders ADD CONSTRAINT fk_customer_id FOREIGN KEY (customer_id)
  REFERENCES customers(id) ON DELETE RESTRICT;
Deleting a customer will fail if there are any associated orders.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, ON DELETE CASCADE, SET NULL, and RESTRICT provide different ways to manage referential integrity when a referenced row is deleted. Each has its own use case:

CASCADE ensures data consistency by deleting related records.
SET NULL sets foreign keys to null, preserving the referencing table’s rows.
RESTRICT enforces referential integrity by preventing deletions that would violate it.

#### Common mistakes
include using CASCADE inappropriately, leading to unintended data loss. Always consider your schema design and business logic when choosing these constraints.

Self-Check

Correctness + completeness confirmed.

**Code:**

```sql
-- Example of ON DELETE CASCADE
ALTER TABLE orders ADD CONSTRAINT fk_customer_id FOREIGN KEY (customer_id)
  REFERENCES customers(id) ON DELETE CASCADE;

-- Example of ON DELETE SET NULL
ALTER TABLE orders ADD CONSTRAINT fk_customer_id FOREIGN KEY (customer_id)
  REFERENCES customers(id) ON DELETE SET NULL;

-- Example of ON DELETE RESTRICT
ALTER TABLE orders ADD CONSTRAINT fk_customer_id FOREIGN KEY (customer_id)
  REFERENCES customers(id) ON DELETE RESTRICT;
```

