---
type: "practice"
difficulty: "Deep"
---

# How do you handle the 'not found' condition when looping through a cursor?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Procedure Parameters](<../../../../01_Databases_and_SQL/Database_Objects/Stored_Procedures/Procedure_Parameters.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When handling the 'not found' condition in stored procedures, especially during cursor operations, it's crucial to ensure that your logic correctly manages situations where no records match the criteria. This is particularly important when looping through a cursor, as missing data can lead to unexpected behavior or errors if not properly handled.

#### Triaging the Question

The question asks how to handle the 'not found' condition during a loop within a stored procedure using cursors. The main focus here is understanding the potential pitfalls and how to robustly manage such conditions without causing your application to crash or behave unpredictably.

#### First-Principles Solution

To address this, we need to understand that cursor loops can iterate through rows returned by a query. If no rows are found (i.e., 'not found' condition), the loop might not execute at all, leading to unexpected outcomes. The key is to ensure that your stored procedure handles these scenarios gracefully.

Initialization and Setup: Properly initialize the cursor and set up any necessary conditions.
Loop Logic: Ensure the loop logic can handle both the presence of rows and their absence.
Error Handling: Implement error handling mechanisms to catch and manage 'not found' conditions appropriately.
Post-Iteration Cleanup: Clean up resources after the loop completes.

#### Example
Code
SQL
DELIMITER $$

CREATE PROCEDURE ProcessCursorData()
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE customerId VARCHAR(255);

    -- Declare cursor for customer IDs that meet certain criteria
    DECLARE customerCursor CURSOR FOR
        SELECT id FROM customers WHERE status = 'active' AND last_purchase_date > DATE_SUB(NOW(), INTERVAL 1 MONTH);

    -- Declaration of handler for no rows found condition
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;

    OPEN customerCursor;

    read_loop: LOOP
        FETCH NEXT FROM customerCursor INTO customerId;

        IF done THEN
            LEAVE read_loop;
        END IF;

        -- Process each customer ID
        CALL processCustomer(customerId);
    END LOOP;

    CLOSE customerCursor;
END$$

DELIMITER ;

#### Closing Summary
and Common Mistakes/Follow-Ups

In the provided example, we've ensured that our stored procedure correctly handles the 'not found' condition by setting up a CONTINUE HANDLER for NOT FOUND to set the done flag. This prevents potential issues where the cursor loop does not execute at all when no rows are returned.

#### Common Mistakes

Forgetting to Initialize Handlers: Not properly initializing handlers can lead to unexpected behavior if no rows are found.
Not Properly Closing Cursors: Failing to close cursors after processing can result in resource leaks.
Overlooking Post-Iteration Cleanup: Neglecting cleanup logic can leave resources open or affect the next procedure execution.

#### Follow-Ups

How would you optimize this stored procedure for performance?
What other error handling mechanisms could be useful in a similar scenario?
Can you explain how to handle multiple cursor loops within the same stored procedure?

**Comparison:**

| id   | status    | last_purchase_date |
|------|-----------|--------------------|
| cust1| active    | 2023-01-01         |
| cust2| inactive  | 2023-01-15         |
| cust3| active    | 2023-02-20         |

**Code:**

```text
DELIMITER $$

CREATE PROCEDURE ProcessCursorData()
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE customerId VARCHAR(255);

    -- Declare cursor for customer IDs that meet certain criteria
    DECLARE customerCursor CURSOR FOR
        SELECT id FROM customers WHERE status = 'active' AND last_purchase_date > DATE_SUB(NOW(), INTERVAL 1 MONTH);

    -- Declaration of handler for no rows found condition
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;

    OPEN customerCursor;

    read_loop: LOOP
        FETCH NEXT FROM customerCursor INTO customerId;

        IF done THEN
            LEAVE read_loop;
        END IF;

        -- Process each customer ID
        CALL processCustomer(customerId);
    END LOOP;

    CLOSE customerCursor;
END$$

DELIMITER ;
```

**Example input / output:**

```text
CALL ProcessCursorData();
-- Expected Output:
-- Processed Customer ID: cust1
-- Processed Customer ID: cust3
```

**Data:**

```sql
-- Sample Data
INSERT INTO customers (id, status, last_purchase_date) VALUES
('cust1', 'active', '2023-01-01'),
('cust2', 'inactive', '2023-01-15'),
('cust3', 'active', '2023-02-20');
```

**Flow:**

1. Yes
2. No
3. No
4. Yes
5. Start
6. Is Cursor Empty?
7. Set done = TRUE
8. FETCH Next Customer ID
9. Is done Set?
10. Close Cursor, End Loop

