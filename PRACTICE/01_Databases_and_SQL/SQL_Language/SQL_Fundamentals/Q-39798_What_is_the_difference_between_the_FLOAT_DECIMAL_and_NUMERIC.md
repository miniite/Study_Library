---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between the FLOAT, DECIMAL, and NUMERIC data types, and which is exact?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Storing Money](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Storing_Money.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with databases, understanding the differences between FLOAT, DECIMAL, and NUMERIC data types is crucial to ensure accurate data representation and efficient query performance. In this answer, I will explain what each of these data types means in SQL, their main characteristics, and which one provides exact arithmetic.

TL;DR
FLOAT: An approximate numeric data type used for scientific calculations with large values.
DECIMAL and NUMERIC: Fixed-point decimal numbers that ensure precision and exactness. DECIMAL and NUMERIC are interchangeable in SQL standards, meaning they store the same kind of data.

#### Triage

The question is probing your understanding of numeric data types in SQL databases. The main concepts to cover include:

Precision vs. Range: How each type handles precision and range.
Storage Requirements: Memory usage for storing these values.
Arithmetic Operations: Precision during arithmetic operations.
Use Cases: Situations where one type might be preferred over another.

#### First Principles Solution
FLOAT

FLOAT is a floating-point number, which means it can represent both integer and non-integer values. It uses an approximate algorithm to store numbers, making it suitable for scenarios requiring scientific or engineering calculations with large values. However, due to the nature of floating-point arithmetic, operations involving FLOAT may result in rounding errors.

DECIMAL and NUMERIC

Both DECIMAL and NUMERIC are fixed-point decimal types that ensure exactness during arithmetic operations. They store numbers as precise decimals without any loss of precision or rounding errors. These data types are ideal for financial calculations, where accuracy is critical.

Detailed Solution

Precision:

FLOAT: Stores values in binary format using a floating exponent. This can lead to small inaccuracies when representing decimal fractions.
DECIMAL and NUMERIC: Store numbers as exact decimals, ensuring no rounding errors during arithmetic operations.

Storage Requirements:

The storage requirements for FLOAT are generally smaller compared to DECIMAL because it uses a binary floating-point format. However, the trade-off is accuracy.
For DECIMAL and NUMERIC, the storage requirement depends on the specified precision (number of digits). Each digit requires 4 bytes in some databases.

Arithmetic Operations:

FLOAT: Arithmetic operations can introduce small inaccuracies due to the nature of floating-point representation.
DECIMAL and NUMERIC: These data types guarantee exact arithmetic, making them suitable for scenarios where precision is critical.
Use Cases
FLOAT: Suitable for scientific calculations, simulations, or any application that involves large numbers with approximate results.
DECIMAL and NUMERIC: Ideal for financial applications like banking, accounting, or any scenario requiring exact decimal operations without rounding errors.

#### Closing Summary

In summary, FLOAT is an approximate numeric type used in scenarios where precision can be traded off for performance and storage efficiency. On the other hand, DECIMAL and NUMERIC provide exact arithmetic operations suitable for applications that require precise calculations, such as financial systems.

#### Common Mistakes/Follow-Ups
Common Mistake: Using FLOAT when DECIMAL or NUMERIC would be more appropriate in financial contexts.

#### Follow-Up
How do you handle overflow/underflow issues with each data type? (For FLOAT, it can lead to NaN values; for DECIMAL, the maximum value is defined by the database schema.)

**Code:**

```sql
-- Example of using DECIMAL in SQL
CREATE TABLE financial_transactions (
    id INT PRIMARY KEY,
    amount DECIMAL(10, 2) -- Precision of 10 digits total, 2 after decimal point
);
```

