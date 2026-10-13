---
type: "practice"
difficulty: "Intermediate"
---

# What does the ON DELETE CASCADE option do on a foreign key in Oracle?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Oracle SQL and Indexes](<../../../../TOOLS/Oracle/Oracle_Platform/Oracle_SQL_and_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

ON DELETE CASCADE is a crucial feature in database management that ensures referential integrity when you delete a row from a parent table. This feature automatically deletes related rows from child tables, ensuring consistency across tables linked by foreign keys.

The question here is essentially asking for an explanation of how ON DELETE CASCADE works and its implications within Oracle Database. Let's break it down step-by-step:

Understanding the Requirement: The interviewer wants to understand your grasp of referential integrity in databases and specifically how Oracle handles cascading deletions.
Key Assumptions:
You have a basic understanding of primary keys and foreign keys.
You know that ON DELETE CASCADE is used within the context of a foreign key constraint.

Now, let's delve into a detailed explanation:

Step-by-Step Explanation
Foreign Key Constraint: A foreign key is a column (or combination of columns) in one table that refers to the primary key of another table. This relationship ensures referential integrity by linking records between tables.
ON DELETE CASCADE: When you enable ON DELETE CASCADE on a foreign key, Oracle automatically deletes any rows from the child table whenever a row is deleted from its parent table. This prevents orphaned data in the child table.

#### Example Scenario

Let's consider two tables: ORDERS (parent) and ORDER_ITEMS (child):

SQL
-- Parent Table: ORDERS
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT
);

-- Child Table: ORDER_ITEMS
CREATE TABLE order_items (
    item_id INT PRIMARY KEY,
    order_id INT,
    product_name VARCHAR(255),
    quantity INT,
    FOREIGN KEY (order_id) REFERENCES orders(order_id) ON DELETE CASCADE
);

If we delete an order from the ORDERS table, all related rows in the ORDER_ITEMS table will also be deleted automatically:

SQL
DELETE FROM orders WHERE order_id = 1;

This deletion would cascade to and affect all records in ORDER_ITEMS that reference order_id = 1.

Why Use ON DELETE CASCADE?

Using ON DELETE CASCADE helps maintain data integrity by ensuring that related records are consistently removed, reducing the risk of orphaned data. However, it's not a one-size-fits-all solution; improper use can lead to accidental deletion of critical data.

#### Common Mistakes
and Follow-Ups
Overuse: Using ON DELETE CASCADE without careful consideration can lead to unintended deletions.
No Logging: Ensure you have proper logging in place when using this feature, as it can be difficult to reverse such operations.
Performance Impact: While efficient for small datasets, large-scale operations may impact performance.

#### Closing Summary

ON DELETE CASCADE is a powerful tool in Oracle Database that ensures data integrity by automatically deleting related records. However, its use should be carefully considered to avoid unintended consequences and maintain the integrity of your database schema.

**Code:**

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT
);

CREATE TABLE order_items (
    item_id INT PRIMARY KEY,
    order_id INT,
    product_name VARCHAR(255),
    quantity INT,
    FOREIGN KEY (order_id) REFERENCES orders(order_id) ON DELETE CASCADE
);
```

**Example input / output:**

```text
-- Deleting an order with related items
DELETE FROM orders WHERE order_id = 1;

-- Expected Output:
-- All records in ORDER_ITEMS where order_id = 1 are deleted.
```

