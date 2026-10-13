---
type: "practice"
difficulty: "Deep"
---

# What is meant by set-based versus procedural thinking and how do cursors relate to it?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Cursors vs Set-based SQL](<../../../../01_Databases_and_SQL/Database_Objects/Cursors/Cursors_vs_Set-based_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database development, the distinction between set-based and procedural thinking revolves around how you process data. Set-based operations focus on manipulating sets of rows as a whole, whereas procedural thinking involves iterating through data using cursors or loops to manipulate one row at a time.

Cursors are tools that enable procedural logic in SQL by allowing us to work with individual rows from a result set, just like how we would do with imperative programming languages. This can be useful in scenarios where complex operations need to be performed on each row independently of the others.

#### Triage

The core of this question is about understanding the differences between these two paradigms and seeing how cursors fit into procedural thinking within a database context. The key assumptions here are that you understand basic SQL set-based operations and have some exposure to procedural programming concepts.

#### First-Principles Solution
Set-Based Thinking

Set-based operations work on entire sets of data at once, leveraging the power of SQL’s declarative nature to achieve results without explicitly handling each row. For example:

SQL
SELECT * FROM customers WHERE country = 'USA';

This statement returns all customers from a particular country without specifying each customer individually.

Procedural Thinking and Cursors

In contrast, procedural thinking requires explicit iteration over data using constructs like loops or cursors. A cursor allows you to fetch rows one at a time and perform operations on them sequentially. For instance:

SQL
DECLARE cur CURSOR FOR SELECT * FROM customers WHERE country = 'USA';
FETCH NEXT FROM cur INTO @customer;
WHILE @@FETCH_STATUS = 0 BEGIN
    -- Perform some operation on @customer
    FETCH NEXT FROM cur INTO @customer;
END
CLOSE cur;
DEALLOCATE cur;

This example iterates over the result set, performing operations sequentially.

#### Example Scenario
Updating Records

Imagine you want to update a customer's address. In a set-based approach:

SQL
UPDATE customers SET address = '123 New Street' WHERE country = 'USA';

In a procedural approach with cursors:

SQL
DECLARE cur CURSOR FOR SELECT * FROM customers WHERE country = 'USA';
FETCH NEXT FROM cur INTO @customer;
WHILE @@FETCH_STATUS = 0 BEGIN
    -- Perform some complex operation on @customer, like adding a timestamp or logging updates
    UPDATE customers SET address = '123 New Street' WHERE customer_id = @customer.customer_id;
    FETCH NEXT FROM cur INTO @customer;
END
CLOSE cur;
DEALLOCATE cur;

#### Closing Summary
and Common Mistakes/Follow-Ups

Cursors can be powerful but are often overused, leading to performance issues. Always consider set-based approaches first as they are generally more efficient and easier to maintain. However, there are scenarios where cursors provide necessary flexibility.

#### Common mistakes
include:

Overusing cursors for simple operations.
Poor error handling or resource management in cursor logic.
Not considering the performance implications of using cursors extensively.

#### Follow-up
questions might involve discussing specific use cases where cursors are beneficial and when to avoid them, along with potential optimizations to improve their efficiency.

**Comparison:**

| customer_id | name        | address     | country |
|-------------|-------------|-------------|---------|
| 1           | John Doe    | 456 Old St. | USA     |
| 2           | Jane Smith  | 789 Main St.| Canada  |
| 3           | Mike Johnson| 101 Elm Rd. | USA     |

**Code:**

```sql
-- Example set-based update statement
UPDATE customers SET address = '123 New Street' WHERE country = 'USA';

-- Example cursor usage for complex operations
DECLARE cur CURSOR FOR SELECT * FROM customers WHERE country = 'USA';
FETCH NEXT FROM cur INTO @customer;
WHILE @@FETCH_STATUS = 0 BEGIN
    -- Perform some operation on @customer, like adding a timestamp or logging updates
    UPDATE customers SET address = '123 New Street' WHERE customer_id = @customer.customer_id;
    FETCH NEXT FROM cur INTO @customer;
END
CLOSE cur;
DEALLOCATE cur;
```

**Example input / output:**

```text
-- Input query to update records set-based
UPDATE customers SET address = '123 New Street' WHERE country = 'USA';

-- Output (assuming 2 rows updated)
(2 rows affected)

-- Input for cursor example
DECLARE cur CURSOR FOR SELECT * FROM customers WHERE country = 'USA';
FETCH NEXT FROM cur INTO @customer;

-- Expected output:
-- Customer ID: 1, Name: John Doe, Address: 456 Old Street, Country: USA
-- Customer ID: 3, Name: Mike Johnson, Address: 101 Elm Rd, Country: USA

-- Input to close and deallocate cursor
CLOSE cur;
DEALLOCATE cur;
```

**Data:**

```sql
-- Sample table schema
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    address VARCHAR(255),
    country VARCHAR(50)
);

-- Sample data
INSERT INTO customers (customer_id, name, address, country) VALUES
(1, 'John Doe', '456 Old Street', 'USA'),
(2, 'Jane Smith', '789 Main St', 'Canada'),
(3, 'Mike Johnson', '101 Elm Rd', 'USA');
```

**Flow:**

1. Yes
2. No
3. Procedurally necessary
4. Not necessary
5. Set-Based Operation
6. Is the operation simple and ona large dataset?
7. Use Set-Based Approach
8. Cursors or Procedural Logic?
9. Use Cursor with Proper ErrorHandling
10. Refactor to Set-Based Logic
11. Optimize Cursor Performance
12. Monitor and Optimize

