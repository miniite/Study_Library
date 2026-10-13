---
type: "practice"
difficulty: "Basic"
---

# What does the SIGNED versus UNSIGNED attribute on an integer column change about its range?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Constraint Definitions](<../../../../TOOLS/MySQL/MySQL_Platform/Constraint_Definitions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When defining an integer column in a MySQL database, you can choose between SIGNED and UNSIGNED. The key difference lies in the range of values that each can hold. Let's dive into what these attributes mean and how they affect your data storage.

First, let's clarify what "signed" and "unsigned" mean:

SIGNED means the column can store both positive and negative integers.
UNSIGNED means the column is restricted to storing only non-negative integers (i.e., zero and positive numbers).

Next, we'll look at the specific ranges for each type:

SQL
-- SIGNED integer range in MySQL
SIGNED INT: -2^31 to 2^31 - 1 = -2147483648 to 2147483647

-- UNSIGNED integer range in MySQL
UNSIGNED INT: 0 to 2^32 - 1 = 0 to 4294967295

The choice between SIGNED and UNSIGNED impacts how you define the column. Here's a step-by-step reasoning from first principles:

Step-by-Step Reasoning

Understanding Binary Representation:

In binary, signed integers use one bit to indicate the sign (positive or negative) of the number.
For unsigned integers, all bits are used for representing the magnitude of the number.

#### Example
with INT Type:

An INT type in MySQL is 32 bits wide.
With a signed integer (SIGNED INT), one bit is reserved as a sign bit (0 for positive and 1 for negative).
This leaves 31 bits to represent the magnitude, leading to the range from -2^31 to 2^31 - 1.
For an unsigned integer (UNSIGNED INT), all 32 bits are used to represent the number.
The range is from 0 to 2^32 - 1.
Practical Considerations

Storage Space:

Using SIGNED can save one bit of storage per row, which might be useful in certain scenarios where space is a critical concern.

Value Range:

If your application requires the use of negative numbers, you must use SIGNED. If only non-negative values are needed, UNSIGNED is more appropriate and can provide a larger range for positive integers.

#### Common Mistakes
Overlooking Range: Developers might choose SIGNED or UNSIGNED without considering the full range required by their application.
Ignoring Storage Cost: Failing to consider that using SIGNED saves one bit of storage, which can be significant in large datasets.

#### Closing Summary
and Follow-Ups

In summary, choosing between SIGNED and UNSIGNED depends on your specific data requirements. SIGNED allows for both positive and negative numbers within a smaller range, while UNSIGNED provides a larger range of non-negative values but at the cost of one bit per row.

When designing your database schema, always consider the full implications of these choices to ensure optimal performance and correct data representation.

**Code:**

```sql
-- Example SQL statement for defining SIGNED vs. UNSIGNED integers
CREATE TABLE example (
    signed_column INT SIGNED,
    unsigned_column INT UNSIGNED
);
```

