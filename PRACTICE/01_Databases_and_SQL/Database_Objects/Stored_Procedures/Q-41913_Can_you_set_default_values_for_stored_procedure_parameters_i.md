---
type: "practice"
difficulty: "Intermediate"
---

# Can you set default values for stored procedure parameters in MySQL? What are the alternatives?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Procedure Parameters](<../../../../01_Databases_and_SQL/Database_Objects/Stored_Procedures/Procedure_Parameters.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a database development context, especially when dealing with MySQL, understanding how to handle stored procedure parameters is crucial. Specifically, one might wonder if setting default values for stored procedure parameters is possible and what alternative approaches exist. Let's dive into this topic from first principles.

#### Triaging the Question

The question revolves around two main points:

Setting Default Values: Can a stored procedure parameter in MySQL have a default value?
Alternatives: If not, what are some other ways to achieve similar functionality?

#### First Principles Solution
Default Values for Parameters

In MySQL, stored procedures do not support setting default values directly on parameters within the procedure definition. However, there are workarounds and alternatives that can achieve a similar outcome.

#### Example

SQL
DELIMITER //
CREATE PROCEDURE CalculateSalary(IN emp_id INT)
BEGIN
    -- This parameter does not have a default value.
    SELECT salary FROM employees WHERE id = emp_id;
END //
DELIMITER ;

In the above example, emp_id is an input parameter without a default value. If you want to provide a default behavior or handle missing values gracefully, other mechanisms need to be employed.

Alternatives

Check for Null/Default Values Inside the Procedure: You can check if a parameter is null or has a specific value and then set it accordingly within the procedure logic.

SQL
DELIMITER //
CREATE PROCEDURE CalculateSalary(IN emp_id INT)
BEGIN
    DECLARE default_emp_id INT DEFAULT 1; -- Default employee ID

    IF emp_id IS NULL THEN
        SET emp_id = default_emp_id;
    END IF;

    SELECT salary FROM employees WHERE id = emp_id;
END //
DELIMITER ;

Using a Default Parameter in the CALL Statement: You can also set the parameter value when calling the stored procedure.

SQL
CALL CalculateSalary(NULL); -- Uses default value from within the procedure.

Adding a Default Value in the Procedure Header: Although not directly supported, you can use a default value during the call to the procedure and then check for it inside the procedure.

Using a DEFAULT Clause with User-Defined Variables: Another approach involves using user-defined variables to handle defaults.

SQL
DELIMITER //
CREATE PROCEDURE CalculateSalary(IN emp_id INT)
BEGIN
    DECLARE default_emp_id INT DEFAULT 1; -- Default employee ID

    IF emp_id IS NULL THEN
        SET emp_id = @default_emp_id;
    ELSE
        SET @default_emp_id = emp_id;
    END IF;

    SELECT salary FROM employees WHERE id = @default_emp_id;
END //
DELIMITER ;

#### Closing Summary
and Common Mistakes/Follow-Ups

Setting default values directly on stored procedure parameters is not supported in MySQL. However, you can achieve similar behavior by checking for nulls or default values within the procedure logic or setting defaults during the call to the procedure. Always consider performance implications when using these workarounds.

#### Common mistakes
include:

Forgetting to check for null values and assuming they are always provided.
Overlooking the performance impact of default value checks inside the stored procedure, especially in large datasets.

For follow-ups, you might want to discuss how these solutions scale with larger databases or different use cases. Additionally, exploring other database systems that support default parameter values could be beneficial for a broader understanding.

