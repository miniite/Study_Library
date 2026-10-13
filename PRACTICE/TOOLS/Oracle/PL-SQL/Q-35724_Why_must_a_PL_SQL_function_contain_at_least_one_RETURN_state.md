---
type: "practice"
difficulty: "Basic"
---

# Why must a PL/SQL function contain at least one RETURN statement?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Functions in SQL vs PL/SQL](<../../../../TOOLS/Oracle/PL-SQL/Functions_in_SQL_vs_PLSQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In PL/SQL, functions are used to perform specific tasks and return a value to the caller. The RETURN statement is essential because it explicitly defines how the function communicates its result back to the calling context. Without a RETURN statement, the function wouldn’t provide any output or would implicitly return NULL, which isn't always desirable.

#### Triaging the Question

The core of this question is understanding why the RETURN statement is mandatory in PL/SQL functions. The interviewer might be probing your knowledge on how functions work and how they differ from procedures, as well as your grasp of value-returning vs. void operations.

#### First-Principles Solution

PL/SQL functions are designed to return a specific result to the caller based on some logic or computations. The RETURN statement is crucial because it ensures that the function's output is clear and predictable. Here’s why:

Value Return: Functions in PL/SQL should logically return a value as per their definition. This could be an arithmetic operation, string concatenation, or any other computation.
Explicitness: The RETURN statement makes it explicit that the function will output something meaningful to the caller. It ensures that the function doesn’t just execute some operations without providing any output.
Code Clarity and Maintainability: By using a RETURN statement, you make your code more readable and maintainable. Other developers can easily understand what the function is supposed to return.
Code Example
PLSQL
-- A simple PL/SQL function that calculates the sum of two numbers
function add_numbers(num1 in number, num2 in number) return number is
    result number;
begin
    -- Perform the addition and store it in 'result'
    result := num1 + num2;
    -- Return the computed value
    return result;
end add_numbers;

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, a PL/SQL function must contain at least one RETURN statement because functions are expected to provide output. Without it, the function would not be able to communicate its result back to the caller, which is against the fundamental purpose of a function.

#### Common mistakes
include:

Forgetting to use the RETURN statement in your code.
Assuming that a PL/SQL function can operate without returning any value (though this isn’t strictly true, as functions can return NULL, but it’s not a best practice).

#### Follow-up
questions might include:

How does the absence of a RETURN statement affect error handling?
Can you write an example where a RETURN is implicitly used?

**Code:**

```
PLSQL
Copy
-- Example function without explicit RETURN
function add_numbers(num1 in number, num2 in number) return number is
    result number;
begin
    -- Perform the addition and store it in 'result'
    result := num1 + num2;
    -- No explicit RETURN statement here, but this would implicitly return NULL or the last value of 'result' if explicitly set.
end add_numbers;
```

