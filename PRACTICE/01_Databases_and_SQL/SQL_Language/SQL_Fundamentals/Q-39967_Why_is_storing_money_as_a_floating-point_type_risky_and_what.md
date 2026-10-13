---
type: "practice"
difficulty: "Deep"
---

# Why is storing money as a floating-point type risky, and what data type should you use instead?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Storing Money](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Storing_Money.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Storing monetary values like money in a floating-point format can introduce precision issues and rounding errors, which are critical in financial applications where accuracy is paramount. To mitigate these risks, it’s recommended to use an integer type that stores the value without fractional parts (e.g., cents), thereby avoiding floating-point arithmetic altogether.

To break this down from first principles:

Understanding Floating-Point Representation: Floating-point numbers represent values using a binary format similar to scientific notation. This can lead to precision issues because not all decimal fractions can be represented exactly in binary.
Common Issues with Floating-Point Arithmetic:
Rounding errors: When you perform arithmetic operations, the result may lose some precision.
Inconsistent results: The same operation might yield slightly different results across different platforms or even within the same application under different conditions.

Let's take a step-by-step look at why floating-point is risky and what to use instead:

#### Triage

The core of the question is about understanding why using float or double for monetary values can be problematic and exploring better alternatives. The interviewer will likely be probing your knowledge of numerical representation, potential pitfalls in financial calculations, and best practices.

Step-by-Step Solution from First Principles

Floating-Point Precision Issues:

Representation: Floating-point numbers are represented using a sign bit, exponent, and mantissa ( significand). This means that not all decimal fractions can be precisely represented.

#### Example
The fraction 0.1 cannot be exactly represented as a floating-point number; it results in an approximation.

Rounding Errors:

Arithmetic operations on these approximations introduce small rounding errors, which accumulate over multiple calculations. For example, adding and subtracting the same small value can lead to significant discrepancies.

#### Example
0.1 + 0.2 does not equal 0.3 exactly due to rounding.

Practical Impact:

In financial systems where accuracy is crucial (e.g., accounting, banking), these tiny errors can have real-world consequences such as incorrect balance calculations or discrepancies in transactions.

#### Example
A small error accumulating over millions of transactions could result in significant financial imbalances.

Alternative Data Types:

Integer Type: Storing monetary values using an integer type (e.g., cents) avoids these issues altogether. By converting dollars to cents, you can perform precise arithmetic.
SQL
-- Example: Convert dollars to cents
SELECT amount * 100 AS cents FROM transactions;
BigInt or Decimal: Using a DECIMAL type in SQL ensures that the value is stored with exact precision. This data type does not suffer from the same rounding issues as floating-point types.

#### Closing Summary
and Common Mistakes

Using an integer type for storing monetary values is safer and more reliable because it avoids the inherent imprecision of floating-point arithmetic. The key takeaway is to avoid using float or double for financial calculations; instead, use integer representations (like cents) or exact numeric types (DECIMAL).

#### Common Mistakes
Assuming that float or double can be used interchangeably with int or decimal without considering the implications.
Ignoring rounding errors and their potential impact on financial systems.

**Code:**

```sql
-- Example of storing money in cents as an integer
CREATE TABLE transactions (
    id INT PRIMARY KEY,
    amount DECIMAL(10, 2),
    cents INT -- Store amounts in cents for exact precision
);
```

**Example input / output:**

```text
Input:
INSERT INTO transactions (id, amount) VALUES (1, 1.5);

Output:
SELECT cents FROM transactions WHERE id = 1;

Result:
cents -> 150
```

