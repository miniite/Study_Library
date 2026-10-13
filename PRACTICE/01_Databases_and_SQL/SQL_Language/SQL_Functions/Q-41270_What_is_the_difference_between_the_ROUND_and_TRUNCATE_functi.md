---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between the ROUND and TRUNCATE functions when reducing decimal places?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Numeric Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/Numeric_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In SQL, both ROUND and TRUNCATE are used to reduce the number of decimal places in a numeric value, but they operate differently under the hood. This distinction is crucial when dealing with precision and rounding behavior.

#### Triage

The question asks for the difference between the ROUND and TRUNCATE functions when reducing decimal places. The interviewer likely wants to understand your grasp of how these two functions handle numeric values internally, their edge cases, and potential implications in real-world scenarios.

#### First-Principles Solution

1. ROUND Function:

Behavior: Rounds the number to a specified number of decimal places.

#### Example
ROUND(3.7856, 2) returns 3.79.
Internals: Internally, SQL rounds the number by adding 0.5 times 10^(-scale) to the value before truncating it. For example, in the case of 3.7856, adding 0.005 would result in 3.79.

2. TRUNCATE Function:

Behavior: Truncates (cuts off) a number at a specified decimal place without rounding.

#### Example
TRUNCATE(3.7856, 2) returns 3.78.
Internals: Simply removes the digits beyond the specified decimal place.
Key Differences

Rounding vs Truncation:

ROUND involves a rounding operation, which can alter the value.
TRUNCATE simply cuts off the extra digits without any rounding.

Implications in Real-World Scenarios:

If precision is critical and you don't want any alteration to the original number (e.g., financial transactions), use TRUNCATE.
For scenarios where you need to ensure data consistency across multiple operations or when dealing with floating-point errors, prefer ROUND.

#### Common Mistakes/Follow-Ups
Misunderstanding Rounding Rules: Failing to understand that rounding can sometimes lead to unexpected results.
Not Considering Precision Loss: Ignoring the fact that truncation simply removes digits without altering them, which might cause precision issues in some use cases.

In summary, ROUND and TRUNCATE serve different purposes. Use ROUND when you need a value rounded for readability or consistency, and use TRUNCATE when you want to avoid any rounding and simply remove the extra digits. Understanding these nuances is crucial for effective SQL development.

**Code:**

```sql
-- Example of ROUND function
SELECT ROUND(3.7856, 2); -- Returns 3.79

-- Example of TRUNCATE function
SELECT TRUNCATE(3.7856, 2); -- Returns 3.78
```

**Example input / output:**

```text
Input: ROUND(3.7856, 2)
Output: 3.79

Input: TRUNCATE(3.7856, 2)
Output: 3.78
```

