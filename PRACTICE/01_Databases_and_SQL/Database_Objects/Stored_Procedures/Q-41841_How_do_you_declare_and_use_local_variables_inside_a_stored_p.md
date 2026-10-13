---
type: "practice"
difficulty: "Basic"
---

# How do you declare and use local variables inside a stored procedure?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Stored Procedure Fundamentals](<../../../../01_Databases_and_SQL/Database_Objects/Stored_Procedures/Stored_Procedure_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with stored procedures in SQL, declaring and using local variables is a fundamental task that can help manage state within your procedure. Local variables are declared at the beginning of the stored procedure's body and are used to store temporary data.

#### Triage

The question asks about how to declare and use local variables inside a stored procedure. The key points the interviewer might be probing include understanding the syntax for declaring variables, initializing them, and using them within different parts of the stored procedure (e.g., within loops or conditional statements).

#### Solution
from First Principles:

Local variables in SQL are declared with the DECLARE statement. These variables can hold values that persist only during the execution of the stored procedure. Here’s a step-by-step guide on how to declare and use local variables:

Declare Variables: Use the DECLARE statement followed by the variable name, data type, and optional initial value.
Initialize Variables: Assign an initial value to the declared variables.
Use Variables: Refer to these variables within the stored procedure logic.

#### Example

Let's consider a simple example where we create a stored procedure that calculates the average of two numbers:

SQL
CREATE PROCEDURE calculate_average(
    @num1 INT,
    @num2 INT
)
AS
BEGIN
    -- Declare local variables
    DECLARE @sum INT, @average FLOAT;

    -- Initialize the sum variable with the sum of num1 and num2
    SET @sum = @num1 + @num2;

    -- Calculate the average by dividing the sum by 2
    SET @average = @sum / 2.0;

    -- Output the result
    SELECT @average AS Average;
END;

#### Closing Summary

In summary, declaring and using local variables in stored procedures involves using the DECLARE statement followed by setting their initial values. These variables can then be used throughout the procedure logic to manage temporary data.

#### Common Mistakes/Follow-Ups
Forgetting to Declare Variables: Always declare any variables you plan to use.
Incorrect Data Types: Ensure that the data types of your variables match the expected input.
Scope Issues: Remember that local variables are scoped only within the stored procedure and cannot be accessed outside it.

If we were asked about handling errors or best practices, we could extend this example by adding error handling clauses like BEGIN TRY and BEGIN CATCH.

**Code:**

```sql
CREATE PROCEDURE calculate_average(
    @num1 INT,
    @num2 INT
)
AS
BEGIN
    -- Declare local variables
    DECLARE @sum INT, @average FLOAT;

    -- Initialize the sum variable with the sum of num1 and num2
    SET @sum = @num1 + @num2;

    -- Calculate the average by dividing the sum by 2.0
    SET @average = @sum / 2.0;

    -- Output the result
    SELECT @average AS Average;
END;
```

