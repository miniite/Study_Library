---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between an anonymous block error at compile time and one at runtime?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Exception Handling](<../../../../TOOLS/Oracle/PL-SQL/Exception_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the difference between a PL/SQL anonymous block error at compile time versus one at runtime, it's crucial to understand the lifecycle of a PL/SQL program unit, which includes compilation and execution phases. Let's dive into each phase step-by-step.

#### Triaging the Question

The question is essentially asking us to distinguish between errors that occur during the compilation process (compile-time) and those that happen during the runtime (execution). The interviewer may be probing your understanding of PL/SQL fundamentals, error handling, and debugging techniques. We'll assume a basic familiarity with PL/SQL syntax and execution flow.

Compile-Time Errors

Compile-time errors are detected by the PL/SQL compiler before any code is executed. These errors typically indicate logical mistakes or syntactical issues that prevent the code from being compiled successfully.

#### Example
s: Incorrect SQL statements, mismatched data types, undeclared variables, incorrect usage of keywords.

SQL
DECLARE
  v_var NUMBER;
BEGIN
  SELECT * FROM table_name; -- Syntax error: missing INTO clause
END;
/
Runtime Errors

Runtime errors occur during the execution phase and can be more complex. These errors usually indicate issues that arise when the code is being executed, such as logical flaws or external conditions not met.

#### Example
s: Division by zero, null value in a NOT NULL column, database object not found.

SQL
DECLARE
  v_divisor NUMBER := 0;
BEGIN
  DBMS_OUTPUT.PUT_LINE(1 / v_divisor); -- Runtime error: division by zero
END;

#### First-Principles Solution

To illustrate the differences, let's consider a scenario where we have an anonymous block that attempts to divide two numbers:

SQL
DECLARE
  v_numerator NUMBER := 50;
  v_denominator NUMBER := 0;
BEGIN
  DBMS_OUTPUT.PUT_LINE('Result: ' || (v_numerator / v_denominator));
END;
Compile-Time Error Example

Suppose we have an error in the declaration section:

SQL
DECLARE
  v_numerator NUMBER := 50;
  v_denominator STRING(2) := '0';
BEGIN
  DBMS_OUTPUT.PUT_LINE('Result: ' || (v_numerator / v_denominator));
END;

In this case, a compile-time error would occur because STRING and NUMBER are mismatched data types. The PL/SQL compiler will detect this issue before execution.

Runtime Error Example

Now, let's modify the code to include a potential runtime error:

SQL
DECLARE
  v_numerator NUMBER := 50;
  v_denominator NUMBER := 0;
BEGIN
  DBMS_OUTPUT.PUT_LINE('Result: ' || (v_numerator / v_denominator));
END;

Here, the division by zero is an example of a runtime error. This will not be caught during compilation but will occur when the block is executed.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Compile-time errors are easier to catch and fix because they are identified before execution, whereas runtime errors can only be detected during actual program execution. Understanding both types is crucial for effective debugging and maintenance of PL/SQL code.

#### Common mistakes
often include overlooking data type mismatches (compile-time) or logical flaws that only manifest during runtime. Always ensure thorough testing and validation to handle both scenarios effectively.

For follow-ups, the interviewer might ask about best practices for error handling, how to manage exceptions in PL/SQL blocks, or specific techniques for debugging compile-time errors.

**Code:**

```sql
-- Example of a compile-time error due to type mismatch
DECLARE
  v_numerator NUMBER := 50;
  v_denominator STRING(2) := '0';
BEGIN
  DBMS_OUTPUT.PUT_LINE('Result: ' || (v_numerator / v_denominator));
END;

-- Example of a runtime error due to division by zero
DECLARE
  v_numerator NUMBER := 50;
  v_denominator NUMBER := 0;
BEGIN
  DBMS_OUTPUT.PUT_LINE('Result: ' || (v_numerator / v_denominator));
END;
```

