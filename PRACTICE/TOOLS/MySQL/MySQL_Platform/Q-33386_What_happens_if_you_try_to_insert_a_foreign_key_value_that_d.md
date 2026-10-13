---
type: "practice"
difficulty: "Basic"
---

# What happens if you try to insert a foreign key value that does not exist in the referenced table?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Constraint Definitions](<../../../../TOOLS/MySQL/MySQL_Platform/Constraint_Definitions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When inserting data into a table with a foreign key constraint, MySQL will check if the value being inserted exists in the referenced (parent) table. If it doesn't, MySQL will reject the insert operation and return an error message indicating that the foreign key constraint has been violated.

This is to ensure referential integrity, which means that every row in a child table must have a corresponding row in the parent table. Let's break this down step by step from first principles.

**Comparison:**

| Customer Table (customers)    |
|-------------------------------|
| customer_id | name   |
| 1           | Alice  |
| 3           | Bob    |
| Orders Table (orders)         |
|------------------------------|
| order_id | customer_id |
| NULL     | 2           |

**Code:**

```sql
-- Example scenario
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

INSERT INTO orders (order_id, customer_id) VALUES (1001, 2); -- Assume there is no customer with ID 2 in the customers table
```

**Example input / output:**

```

Running the following insert statement will result in an error:

SQL
Copy
INSERT INTO orders (order_id, customer_id) VALUES (1001, 2);

MySQL response:

CODE
Copy
ERROR 1452 (23000): Cannot add or update a child row: a foreign key constraint fails (`database_name`.`orders`, CONSTRAINT `orders_ibfk_1` FOREIGN KEY (`customer_id`) REFERENCES `customers` (`customer_id`))

This error message indicates that the value 2 does not exist in the customers table, violating the foreign key constraint.
```

**Data:**

```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(50)
);

-- Note: There is no entry for customer_id = 2 in this table.
```

**Flow:**

1. Yes
2. No
3. Start
4. Is the value in referencedtable?
5. Proceed with insertion
6. Reject insert and raise error
7. Error message: 'Cannot add orupdate a child row...'

