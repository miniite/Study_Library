---
type: "practice"
difficulty: "Basic"
---

# What does the CONTINUE statement do inside a PL/SQL loop?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Functions in SQL vs PL/SQL](<../../../../TOOLS/Oracle/PL-SQL/Functions_in_SQL_vs_PLSQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The CONTINUE statement in PL/SQL is used to skip the current iteration of a loop and move directly to the next iteration. Essentially, it provides a way to bypass certain sections of code within a loop without terminating the entire loop execution.

#### Triage

The question asks about the functionality of the CONTINUE statement within a PL/SQL loop.
The interviewer is probing your understanding of control flow statements in PL/SQL and how they can be used effectively.

#### First-principles solution

#### Overview
In PL/SQL, when a CONTINUE statement is encountered inside a loop (such as a FOR, LOOP, or WHILE loop), the current iteration of the loop is skipped, and control immediately moves to the next iteration.
Syntax: The basic syntax for using CONTINUE within a PL/SQL loop looks like this:
PLSQL
CONTINUE WHEN condition;
Usage: You can use the CONTINUE statement in various scenarios where you want to skip certain iterations based on specific conditions.

#### Example
Consider the following PL/SQL block that iterates over a set of numbers and skips even numbers:
PLSQL
DECLARE
  num NUMBER := 1;
BEGIN
  LOOP
    IF mod(num, 2) = 0 THEN
      CONTINUE; -- Skip this iteration if 'num' is even
    END IF;

    DBMS_OUTPUT.PUT_LINE('Odd number: ' || num);
    num := num + 1;

    EXIT WHEN num > 10; -- Exit the loop after processing numbers up to 10
  END LOOP;
END;
Output: The above PL/SQL block would output:
CODE
Odd number: 1
Odd number: 3
Odd number: 5
Odd number: 7
Odd number: 9

#### Closing summary
+ common mistakes/follow-ups:

Summary: The CONTINUE statement in PL/SQL allows you to skip the rest of the loop body for a specific iteration, based on a condition. It is useful when you want to process elements selectively within a loop.
Mistakes:
Using CONTINUE without ensuring that the condition is meaningful can lead to unnecessary complexity and potential bugs if not tested thoroughly.
Overusing CONTINUE might make the code harder to read, so it's important to use it judiciously.

**Code:**

```
PLSQL
Copy
DECLARE
   num NUMBER := 1;
BEGIN
   LOOP
      IF mod(num, 2) = 0 THEN
         CONTINUE; -- Skip this iteration if 'num' is even
      END IF;

      DBMS_OUTPUT.PUT_LINE('Odd number: ' || num);
      num := num + 1;

      EXIT WHEN num > 10; -- Exit the loop after processing numbers up to 10
   END LOOP;
END;
```

