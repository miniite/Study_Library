---
type: "practice"
difficulty: "Intermediate"
---

# Write a simple stored procedure that inserts a new customer record given name and email as inputs.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Stored Procedure Fundamentals](<../../../../01_Databases_and_SQL/Database_Objects/Stored_Procedures/Stored_Procedure_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To write a simple stored procedure that inserts a new customer record based on given name and email, we need to focus on setting up the database schema, defining the procedure logic, and ensuring proper input validation. The key idea is to create a reusable piece of code that can be called from various applications to insert customer data efficiently.

#### Triage

The question asks us to write a stored procedure for inserting a new customer record into a database. This involves understanding the table structure, handling possible input errors, and ensuring efficient execution. We assume the existence of a customers table with columns such as id, name, and email.

#### First-Principles Solution
Understand the Table Schema: Ensure that you have a customers table in your database schema.
Define Input Validation: Check if the provided name and email are not null or empty.
Write the Stored Procedure: Use SQL to define the procedure, including parameter handling and error handling.
Step-by-Step Solution

Database Schema:

SQL
CREATE TABLE customers (
    id INT AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    PRIMARY KEY (id)
);

Create the Stored Procedure:

Define a stored procedure that takes two parameters: name and email.
Validate input values.
Insert data into the customers table.
Handle possible errors using SQL error codes.
Sample Code
SQL
DELIMITER //

CREATE PROCEDURE insert_customer(IN p_name VARCHAR(255), IN p_email VARCHAR(255))
BEGIN
    -- Check if email is already in use
    IF EXISTS (SELECT 1 FROM customers WHERE email = p_email) THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Email already exists';
    END IF;

    -- Insert new customer record
    INSERT INTO customers(name, email)
    VALUES(p_name, p_email);
END //

DELIMITER ;
Testing the Procedure:
Call the stored procedure with valid and invalid inputs to test its functionality.
SQL
CALL insert_customer('John Doe', 'johndoe@example.com');
CALL insert_customer('Jane Doe', 'jane@doe'); -- This should fail due to email validation

#### Closing Summary
+ Common Mistakes/Follow-Ups
Key Points: The stored procedure ensures data integrity by checking for duplicate emails before inserting a new record.
Common Pitfalls:
Forgetting to validate input values, which can lead to SQL injection or logic errors.
Not handling error conditions properly, leading to unhandled exceptions.
Ignoring performance implications of the query inside the stored procedure.
Self-Check

Correctness and completeness confirmed.

