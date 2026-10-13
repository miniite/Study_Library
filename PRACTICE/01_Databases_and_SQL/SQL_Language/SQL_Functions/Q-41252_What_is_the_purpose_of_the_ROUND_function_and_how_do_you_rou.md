---
type: "practice"
difficulty: "Basic"
---

# What is the purpose of the ROUND function and how do you round to two decimal places?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Numeric Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/Numeric_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The ROUND function in SQL is used to round a numeric value to a specified number of decimal places, making it easier to handle precise numerical data. In this context, we will focus on rounding to two decimal places.

#### Triaging the Question
Purpose: The question asks about the purpose and usage of the ROUND function specifically for rounding numbers to two decimal places.

#### Assumptions
We are dealing with floating-point or numeric values in a database context.
The goal is to ensure that calculations and data representation are accurate up to two digits after the decimal point.

#### First-Principles Solution

The ROUND function in SQL takes at least two arguments:

A numeric value to be rounded.
An integer representing the number of decimal places to which you want to round the value.

To round a value to two decimal places, we would use:

SQL
ROUND(numeric_value, 2);

For example, if we have a floating-point number 3.14159, using the ROUND function with an argument of 2 will result in 3.14.

Step-by-Step Breakdown:
Identify the Value: Determine the numeric value that needs to be rounded.
Specify Decimal Places: Set the second parameter to 2 to indicate rounding to two decimal places.
Apply the Function: Use the ROUND function with both parameters.

Let's illustrate this with an example:

SQL
SELECT ROUND(3.14159, 2); -- Result: 3.14

#### Closing Summary
and Common Mistakes/Follow-Ups:
Summary: The ROUND function in SQL is used to round a number to the nearest value with a specified number of decimal places. For two decimal places, use the second parameter as 2.

#### Common Mistakes
Failing to specify the correct number of decimal places.
Confusing rounding modes (e.g., whether 3.145 should be rounded to 3.15 or 3.14).

#### Example
Dry-Run:
SQL
-- Test data
SELECT ROUND(3.14159, 2),
       ROUND(3.145678, 2),
       ROUND(3.999999, 2);

Expected output:

CODE
+---------------------+--------------------+----------------------+
| ROUND(3.14159, 2)    | ROUND(3.145678, 2)  | ROUND(3.999999, 2)   |
+---------------------+--------------------+----------------------+
| 3.14                | 3.15               | 4.00                 |
+---------------------+--------------------+----------------------+
Self-Check:

Correctness and completeness confirmed.

**Code:**

```sql
-- Example usage of ROUND function to round to two decimal places
SELECT ROUND(3.14159, 2),
       ROUND(3.145678, 2),
       ROUND(3.999999, 2);
```

**Example input / output:**

```text
-- Example data and output
+---------------------+--------------------+----------------------+
| ROUND(3.14159, 2)    | ROUND(3.145678, 2)  | ROUND(3.999999, 2)   |
+---------------------+--------------------+----------------------+
| 3.14                | 3.15               | 4.00                 |
+---------------------+--------------------+----------------------+
```

