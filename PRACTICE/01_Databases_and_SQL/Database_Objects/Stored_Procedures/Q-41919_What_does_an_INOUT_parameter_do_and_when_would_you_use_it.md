---
type: "practice"
difficulty: "Intermediate"
---

# What does an INOUT parameter do and when would you use it?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Procedure Parameters](<../../../../01_Databases_and_SQL/Database_Objects/Stored_Procedures/Procedure_Parameters.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

An INOUT parameter in a stored procedure acts as both input and output; its value can be modified within the procedure and also passed back to the calling application. You'd use an INOUT parameter when you want to manipulate data during execution but also return it to the caller, allowing for bidirectional communication.

Let's break down what this means step by step:

#### Triage
What is being asked: We need to explain the functionality and usage of the INOUT parameter in stored procedures.
Key assumptions: Assumes basic understanding of parameters in SQL and stored procedures.
What we're probing for: The candidate's knowledge of different parameter modes (in, out) and their practical applications.

#### First-Principles Solution

Understanding Parameters:

In a stored procedure, there are primarily three types of parameters: IN, OUT, and INOUT.
An IN parameter is used to pass input values from the calling application to the procedure.
An OUT parameter is used by the procedure to return output values back to the calling application.

Role of INOUT:

The INOUT mode combines both input and output functionalities.
When a value is passed as an INOUT parameter, it starts with the caller's initial value but can be modified within the procedure and then returned to the caller at the end.

Practical Usage Scenarios:

Updating Data and Returning Status: Imagine you have a stored procedure that updates a record in a database table and also needs to return whether the update was successful or not.
Returning Modified Data Structures: Sometimes, when processing complex data structures like arrays or records within a procedure, you might want to modify them and then pass them back.

#### Example Scenario

Let's consider an example where we have a stored procedure that updates a user’s email address in the database. We also need to return whether the update was successful.

SQL
CREATE PROCEDURE UpdateEmail(IN UserID INT, IN NewEmail VARCHAR(255), OUT Success BOOLEAN)
BEGIN
    UPDATE Users SET Email = NewEmail WHERE ID = UserID;

    IF ROW_COUNT() > 0 THEN
        SET Success = TRUE;
    ELSE
        SET Success = FALSE;
    END IF;
END;

In this example:

UserID and NewEmail are passed in as IN parameters.
Success is an OUT parameter that will be modified within the procedure to indicate success or failure of the update.

#### Closing Summary

To summarize, an INOUT parameter allows you to pass values into a stored procedure and then modify those values during execution. The final value can be returned to the caller, making it useful for scenarios where you need both input validation and output feedback.

#### Common Mistakes/Follow-Ups
Misunderstanding the Parameter Mode: Forgetting that INOUT parameters can only be used in stored procedures.
Confusion with Return Values: Often, developers confuse INOUT with return values from a procedure or function.
Incorrect Handling of Data Types and Sizes: Ensuring data types match between input and output.

Would you like to see an example of how this works in practice?

**Code:**

```sql
-- Example stored procedure definition
DELIMITER //
CREATE PROCEDURE UpdateEmail(IN UserID INT, IN NewEmail VARCHAR(255), OUT Success BOOLEAN)
BEGIN
    UPDATE Users SET Email = NewEmail WHERE ID = UserID;

    IF ROW_COUNT() > 0 THEN
        SET Success = TRUE;
    ELSE
        SET Success = FALSE;
    END IF;
END //
DELIMITER ;
```

**Example input / output:**

```text
-- Example input and output
CALL UpdateEmail(1, 'jane.doe@example.com', @Success);

-- Check the success variable to see if the update was successful
SELECT @Success;
```

**Data:**

```sql
-- Sample data for the Users table
INSERT INTO Users (ID, Email) VALUES (1, 'john.doe@example.com');
```

