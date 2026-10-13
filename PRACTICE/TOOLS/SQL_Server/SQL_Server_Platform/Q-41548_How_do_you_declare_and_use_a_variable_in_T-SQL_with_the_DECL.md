---
type: "practice"
difficulty: "Basic"
---

# How do you declare and use a variable in T-SQL with the DECLARE and SET statements?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ CTEs in SQL Server](<../../../../TOOLS/SQL_Server/SQL_Server_Platform/CTEs_in_SQL_Server.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In SQL Server, variables are used to store data that can be referenced throughout your code. The DECLARE statement is used to define a local variable, while the SET (or SELECT) statement assigns a value to it.

#### Triaging the Question

The question asks you to explain how to declare and use a variable in T-SQL with DECLARE and SET. This involves understanding the syntax of these commands and how they work together. We will assume that the variable is declared as a scalar variable, which can hold single values like integers or strings.

#### First-Principles Solution

Declaring a Variable:

Use the DECLARE statement followed by the variable name.
Specify the data type of the variable after the variable name.

Assigning a Value to the Variable:

Use the SET or SELECT statement to assign a value to the declared variable.
Full Example

Here's an example of declaring and using a variable in T-SQL:

SQL
DECLARE @MyVariable INT; -- Declare a variable named MyVariable with data type INT

SET @MyVariable = 10; -- Assign the value 10 to the variable

SELECT @MyVariable AS MyValue; -- Output the value of the variable

#### Closing Summary
and Common Mistakes/Follow-Ups

In T-SQL, variables are declared using DECLARE and assigned values with either SET or SELECT. Always ensure that you declare the data type correctly to avoid runtime errors. A common mistake is declaring a variable without specifying its data type, which would result in an error.

If asked for additional details, you might be prompted to discuss best practices such as using meaningful names for your variables and avoiding global variables unless necessary.

**Code:**

```sql
DECLARE @MyVariable INT; -- Declare a variable named MyVariable with data type INT

SET @MyVariable = 10; -- Assign the value 10 to the variable

SELECT @MyVariable AS MyValue; -- Output the value of the variable
```

