---
type: "practice"
difficulty: "Deep"
---

# What is a DECLARE HANDLER statement in MySQL and what does CONTINUE versus EXIT mean?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Stored Procedure Fundamentals](<../../../../01_Databases_and_SQL/Database_Objects/Stored_Procedures/Stored_Procedure_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To begin with, a DECLARE HANDLER statement in MySQL is used to define handlers that can catch and respond to specific conditions or errors during the execution of stored procedures. These handlers are essentially blocks of code that run when certain events occur, such as warnings, errors, or triggers.

The question specifically asks about the differences between CONTINUE and EXIT keywords within these handlers. Here’s a detailed explanation:

#### Triaging the Question
Understanding DECLARE HANDLER: This statement is used to define event handlers that can be triggered by specific SQL statements or conditions.
Key Concepts: The question focuses on two outcomes: CONTINUE and EXIT. These keywords determine whether the handler should continue executing the stored procedure after handling the event, or if it should terminate.

#### First-Principles Solution
DECLARE HANDLER Syntax
SQL
DECLARE HANDLER FOR condition_list handler_action;
condition_list: A list of conditions that can be caught by the handler.
handler_action: The block of code to execute when the specified conditions are met. This can include CONTINUE or EXIT.
Controlling Execution Flow with CONTINUE and EXIT

CONTINUE:

When a condition is triggered, the handler executes its action.
After executing the handler's block, execution continues from where it left off in the stored procedure.

EXIT:

When a condition is triggered, the handler executes its action.
After executing the handler's block, execution of the stored procedure stops immediately, and control returns to the point that called the stored procedure or exits the entire routine.

#### Example
s

#### Example
with CONTINUE

Suppose you have a stored procedure that processes a set of records. If an error occurs during processing, you want to log it but continue processing other records:

SQL
DELIMITER //
CREATE PROCEDURE process_records()
BEGIN
    DECLARE CONTINUE HANDLER FOR SQLEXCEPTION
        BEGIN
            -- Log the error and reset any conditions
            SELECT 'Error occurred' AS error;
        END;

    DECLARE done INT DEFAULT FALSE;
    DECLARE val INT;
    DECLARE cur CURSOR FOR SELECT id FROM records;
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;

    OPEN cur;

    read_loop: LOOP
        FETCH cur INTO val;
        IF done THEN
            LEAVE read_loop;
        END IF;

        -- Process the record
        INSERT INTO processed_records (id) VALUES (val);
    END LOOP;

    CLOSE cur;
END //
DELIMITER ;

#### Example
with EXIT

If you want to terminate processing when an error occurs, you can use EXIT:

SQL
DELIMITER //
CREATE PROCEDURE process_records()
BEGIN
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
        BEGIN
            -- Log the error and exit the procedure
            SELECT 'Error occurred' AS error;
            LEAVE process_records;  -- Exit the entire stored procedure
        END;

    DECLARE done INT DEFAULT FALSE;
    DECLARE val INT;
    DECLARE cur CURSOR FOR SELECT id FROM records;

    OPEN cur;

    read_loop: LOOP
        FETCH cur INTO val;
        IF done THEN
            LEAVE read_loop;
        END IF;

        -- Process the record
        INSERT INTO processed_records (id) VALUES (val);
    END LOOP;

    CLOSE cur;
END //
DELIMITER ;

#### Closing Summary
and Common Mistakes
CONTINUE is useful for continuing execution after handling an error or warning. It allows you to recover from errors without terminating the entire procedure.
EXIT is used when you want to immediately stop executing the stored procedure upon encountering a specific condition, ensuring that any unprocessed data or statements are not executed.

#### Common Mistakes
Misunderstanding the difference between CONTINUE and EXIT.
Not recognizing situations where handling an error with CONTINUE is preferable over stopping execution with EXIT.

By understanding these concepts, you can effectively use handlers in MySQL stored procedures to manage errors and ensure robust data processing.

**Code:**

```

DELIMITER // CREATE PROCEDURE process_records() BEGIN DECLARE CONTINUE HANDLER FOR SQLEXCEPTION BEGIN -- Log the error and reset any conditions SELECT 'Error occurred' AS error; END;

CODE
Copy
DECLARE done INT DEFAULT FALSE;
DECLARE val INT;
DECLARE cur CURSOR FOR SELECT id FROM records;
DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;

OPEN cur;

read_loop: LOOP
    FETCH cur INTO val;
    IF done THEN
        LEAVE read_loop;
    END IF;

    -- Process the record
    INSERT INTO processed_records (id) VALUES (val);
END LOOP;

CLOSE cur;

END // DELIMITER ;

CODE
Copy

```sql
DELIMITER //
CREATE PROCEDURE process_records()
BEGIN
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
        BEGIN
            -- Log the error and exit the procedure
            SELECT 'Error occurred' AS error;
            LEAVE process_records;  -- Exit the entire stored procedure
        END;

    DECLARE done INT DEFAULT FALSE;
    DECLARE val INT;
    DECLARE cur CURSOR FOR SELECT id FROM records;

    OPEN cur;

    read_loop: LOOP
        FETCH cur INTO val;
        IF done THEN
            LEAVE read_loop;
        END IF;

        -- Process the record
        INSERT INTO processed_records (id) VALUES (val);
    END LOOP;

    CLOSE cur;
END //
DELIMITER ;
```

**Flow:**

1. DECLARE HANDLER FORSQLEXCEPTION
2. CONTINUE
3. Continue execution after errorhandling
4. EXIT
5. Exit procedure immediately

