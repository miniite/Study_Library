---
type: "practice"
difficulty: "Intermediate"
---

# How would you validate input parameters inside a stored procedure before running the main logic?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Procedure Parameters](<../../../../01_Databases_and_SQL/Database_Objects/Stored_Procedures/Procedure_Parameters.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with database development, especially in an intermediate role, it's crucial to ensure that your stored procedures handle and validate input parameters correctly. This ensures data integrity and prevents potential errors or security vulnerabilities. In this context, we'll discuss how to properly validate input parameters before executing the main logic of a stored procedure.

#### Triaging the Question

The question asks about validating input parameters inside a stored procedure. The interviewer is likely probing your understanding of best practices for database programming, including error handling and input validation. You will need to assume that the stored procedure receives several input parameters, which could include strings, numbers, dates, or other data types.

#### First-Principles Solution

To validate input parameters in a stored procedure, you should:

Check Data Types: Ensure that each parameter is of the correct type.
Validate Ranges and Formats: For example, check if a numeric value is within an acceptable range or if a date string has a valid format.
Handle Null Values: Determine whether null values are allowed for each parameter.
Use Error Handling: Implement mechanisms to gracefully handle invalid inputs.
Step-by-Step Solution
1. Define the Stored Procedure

Let's assume we have a stored procedure that processes customer orders. The procedure takes several parameters: order_id, customer_name, order_date, and amount.

SQL
CREATE PROCEDURE process_order (
    IN order_id INT,
    IN customer_name VARCHAR(255),
    IN order_date DATE,
    IN amount DECIMAL(10, 2)
)
BEGIN
    -- Validation logic goes here

END;
2. Validate Data Types and Ranges

You can use conditional statements to check the data types and ranges of each parameter.

SQL
-- Check if order_id is an integer
IF NOT order_id THEN
    SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Order ID must be a non-null integer';
END IF;

-- Check if customer_name is not null or empty
IF customer_name IS NULL OR customer_name = '' THEN
    SIGNAL SQLSTATE '45001' SET MESSAGE_TEXT = 'Customer name cannot be null or empty';
END IF;

-- Check if order_date is valid (e.g., within a certain range)
SET @min_order_date := DATE_SUB(CURDATE(), INTERVAL 30 DAY);
IF order_date < @min_order_date THEN
    SIGNAL SQLSTATE '45002' SET MESSAGE_TEXT = 'Order date must be within the last 30 days';
END IF;

-- Check if amount is positive and not zero
IF amount <= 0 THEN
    SIGNAL SQLSTATE '45003' SET MESSAGE_TEXT = 'Amount must be a positive value';
END IF;
3. Handle Null Values

Decide whether null values are allowed for each parameter. For instance, order_date might not always be required if the order is processed on the current date.

SQL
-- Example of handling optional parameters
IF order_date IS NULL THEN
    SET order_date := CURDATE();
END IF;
4. Error Handling and Graceful Failure

Use appropriate error handling to ensure that the stored procedure fails gracefully when invalid input is detected. This prevents the execution of subsequent logic with potentially incorrect data.

SQL
-- Example of raising an error and exiting early
IF NOT (order_id > 0 AND customer_name IS NOT NULL) THEN
    SIGNAL SQLSTATE '45004' SET MESSAGE_TEXT = 'Invalid input parameters';
END IF;
5. Main Logic Execution

Only after all validations pass, you can proceed with the main logic of the stored procedure.

SQL
-- Main logic here
INSERT INTO orders (order_id, customer_name, order_date, amount) VALUES (order_id, customer_name, order_date, amount);

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, validating input parameters in a stored procedure is crucial for maintaining data integrity and security. The key steps involve checking data types, ranges, and formats, handling null values appropriately, and implementing robust error handling mechanisms.

#### Common mistakes
to avoid include:

Failing to check for valid date formats.
Not handling null values correctly.
Missing necessary type checks.
Overlooking the importance of graceful failure in case of invalid inputs.

During an interview, you might be asked follow-up questions such as:

How would you handle validation with complex data types like JSON or XML?
Can you demonstrate how to validate and process user input in a more complex scenario?

**Comparison:**

| Field        | Type          | Description                      |
|--------------|---------------|----------------------------------|
| order_id     | INT           | Unique identifier for an order    |
| customer_name| VARCHAR(255)  | Name of the customer              |
| order_date   | DATE          | Date when the order was placed    |
| amount       | DECIMAL(10, 2)| Total amount of the order         |

**Code:**

```sql
CREATE PROCEDURE process_order (
    IN order_id INT,
    IN customer_name VARCHAR(255),
    IN order_date DATE,
    IN amount DECIMAL(10, 2)
)
BEGIN
    IF NOT order_id THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Order ID must be a non-null integer';
    END IF;

    IF customer_name IS NULL OR customer_name = '' THEN
        SIGNAL SQLSTATE '45001' SET MESSAGE_TEXT = 'Customer name cannot be null or empty';
    END IF;

    SET @min_order_date := DATE_SUB(CURDATE(), INTERVAL 30 DAY);
    IF order_date < @min_order_date THEN
        SIGNAL SQLSTATE '45002' SET MESSAGE_TEXT = 'Order date must be within the last 30 days';
    END IF;

    IF amount <= 0 THEN
        SIGNAL SQLSTATE '45003' SET MESSAGE_TEXT = 'Amount must be a positive value';
    END IF;

    INSERT INTO orders (order_id, customer_name, order_date, amount) VALUES (order_id, customer_name, order_date, amount);
END;
```

**Example input / output:**

```text
Input:
order_id = 101
customer_name = 'John Doe'
order_date = '2023-10-01'
amount = 50.00

Output:
A new order record is inserted into the `orders` table.

Failure Example:
order_id = NULL
customer_name = ''
order_date = 'invalid-date'
amount = -20.00

Error:
Order ID must be a non-null integer
Customer name cannot be null or empty
Order date must be within the last 30 days
Amount must be a positive value
```

**Data:**

```sql
-- Sample data schema for the `orders` table
CREATE TABLE orders (
    order_id INT,
    customer_name VARCHAR(255),
    order_date DATE,
    amount DECIMAL(10, 2)
);
```

**Flow:**

1. Yes
2. Yes
3. Yes
4. Yes
5. No
6. No
7. No
8. No
9. Start
10. Is Order ID valid?
11. Is Customer Name valid?
12. Is Order Date valid?
13. Is Amount valid?
14. Insert into Orders
15. Error: Invalid amount
16. Error: Invalid date
17. Error: Empty customer name
18. Error: Invalid order ID
19. End

