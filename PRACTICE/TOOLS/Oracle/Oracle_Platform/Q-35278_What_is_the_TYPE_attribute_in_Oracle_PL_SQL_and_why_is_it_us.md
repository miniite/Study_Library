---
type: "practice"
difficulty: "Intermediate"
---

# What is the %TYPE attribute in Oracle PL/SQL and why is it useful?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ PL/SQL Overview](<../../../../TOOLS/Oracle/Oracle_Platform/PLSQL_Overview.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The %TYPE attribute in Oracle PL/SQL is a powerful feature that allows you to define variables with the exact data type of an existing database column. This ensures type safety, which can help prevent errors like passing incorrect data types into stored procedures or functions.

#### Triaging the Question

What the question is really asking: The %TYPE attribute in Oracle PL/SQL and its utility.

Probing points: Understanding of variable declaration, type safety, and best practices in database development.

#### Assumptions
Familiarity with basic SQL and PL/SQL concepts.
Detailed Solution
Definition and Syntax

The %TYPE attribute is a shorthand way to declare the data type of a column from an Oracle database table or view. For example:

SQL
CREATE OR REPLACE PROCEDURE update_customer_address(
    p_customer_id IN customers.customer_id%TYPE,
    p_new_address IN customers.address%TYPE)
IS
BEGIN
    UPDATE customers
    SET address = p_new_address
    WHERE customer_id = p_customer_id;
END;

In this example, p_customer_id and p_new_address are declared using the %TYPE attribute to match the data types of the corresponding columns in the customers table.

Why it is Useful
Type Safety: Using %TYPE ensures that variables are declared with the same data type as their counterparts in the database, reducing the risk of data integrity issues.
Flexibility and Maintenance: If the underlying column's data type changes, you only need to update the table definition, not all the PL/SQL code.
Readability and Clarity: It makes the intention clear by explicitly stating the expected data type of the variable.

#### Example Scenario

#### Cons
ider a scenario where a Customer table has an email column defined as VARCHAR2(100). If you use %TYPE to declare a variable for this column in your PL/SQL code, it will automatically inherit the VARCHAR2(100) data type.

SQL
DECLARE
    v_email customers.email%TYPE;
BEGIN
    SELECT email INTO v_email FROM customers WHERE customer_id = 1;
END;

#### Closing Summary
and Common Mistakes/Follow-Ups
Common Mistake: Overusing %TYPE can lead to overly verbose code. It's best used when necessary.

#### Follow-Up
Questions:
How does %TYPE interact with table synonyms or views?
Can you declare a variable using %ROWTYPE instead of %TYPE? What are the differences?
Summary

The %TYPE attribute in Oracle PL/SQL is useful for declaring variables that match the data types of database columns, ensuring type safety and maintainability. It's particularly handy when working with large-scale applications where frequent changes to table definitions might occur.

**Code:**

```sql
CREATE OR REPLACE PROCEDURE update_customer_address(
    p_customer_id IN customers.customer_id%TYPE,
    p_new_address IN customers.address%TYPE)
IS
BEGIN
    UPDATE customers
    SET address = p_new_address
    WHERE customer_id = p_customer_id;
END;
```

**Example input / output:**

```text
DECLARE
    v_email customers.email%TYPE;
BEGIN
    SELECT email INTO v_email FROM customers WHERE customer_id = 1;
END;
```

**Data:**

```sql
CREATE TABLE customers (
    customer_id NUMBER,
    name VARCHAR2(100),
    address VARCHAR2(100),
    email VARCHAR2(100)
);
```

