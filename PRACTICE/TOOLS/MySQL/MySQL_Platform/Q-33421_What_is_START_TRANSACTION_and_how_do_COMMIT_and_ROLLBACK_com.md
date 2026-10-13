---
type: "practice"
difficulty: "Basic"
---

# What is START TRANSACTION and how do COMMIT and ROLLBACK complete a transaction?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Deadlocks in MySQL](<../../../../TOOLS/MySQL/MySQL_Platform/Deadlocks_in_MySQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In MySQL, START TRANSACTION initiates a new database transaction. This marks the beginning of a sequence of operations that can be treated as a single unit of work. Once started, you perform various SQL statements within this context. The transaction will only be considered complete when either COMMIT or ROLLBACK is executed.

To understand how these commands work, let's break down their roles and implications step by step.

**Code:**

```sql
-- Example of a simple transaction in MySQL

START TRANSACTION;

-- Perform some operations like inserts, updates, or deletes
INSERT INTO users (name, email) VALUES ('John Doe', 'john@example.com');
UPDATE orders SET status = 'shipped' WHERE order_id = 12345;
DELETE FROM products WHERE product_name = 'outdated_product';

-- Now commit the transaction to save all changes
COMMIT;

/*
If something goes wrong during execution, you can roll back:
*/
START TRANSACTION;

INSERT INTO users (name, email) VALUES ('Jane Doe', 'jane@example.com');
UPDATE orders SET status = 'shipped' WHERE order_id = 67890;
DELETE FROM products WHERE product_name = 'another_outdated_product';

-- Assume an error occurs during execution
ROLLBACK;
```

**Example input / output:**

```text
None applicable here, as the example is conceptual rather than a concrete operation with specific inputs and outputs.
```

**Flow:**

1. All operations complete
2. Error during operations
3. START TRANSACTION
4. Perform Operations
5. COMMIT
6. ROLLBACK

