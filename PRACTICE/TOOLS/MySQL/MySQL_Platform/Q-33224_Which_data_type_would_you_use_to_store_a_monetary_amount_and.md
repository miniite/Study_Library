---
type: "practice"
difficulty: "Intermediate"
---

# Which data type would you use to store a monetary amount and why not FLOAT or DOUBLE?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Date Arithmetic](<../../../../TOOLS/MySQL/MySQL_Platform/Date_Arithmetic.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To store a monetary amount in MySQL, we should avoid using FLOAT or DOUBLE types due to their inherent imprecision when dealing with decimal values. Instead, the recommended data type is DECIMAL. Here’s why:

TL;DR: Use DECIMAL for storing monetary amounts because of its precision and accuracy.

#### Triage
The question asks about which MySQL data type to use for monetary storage, while also probing your understanding of numeric types' limitations and best practices in database design.
Why Not FLOAT or DOUBLE?

Precision Issues: Both FLOAT and DOUBLE are floating-point types that store approximate values. They can introduce rounding errors when dealing with decimal places, which is unacceptable for financial calculations.

For example:

SQL
SELECT 0.1 + 0.2; -- May result in 0.30000000000000004 instead of 0.3

Performance and Readability: While FLOAT and DOUBLE can handle a wide range of values, they are generally slower and less precise than fixed-point types.

Why Use DECIMAL?

Precision and Accuracy: The DECIMAL type allows for exact storage and arithmetic operations. It is designed to store numbers with a fixed number of digits before and after the decimal point.

For instance:

SQL
SELECT CAST('10.99' AS DECIMAL(10,2)); -- Exactly 10.99

Control Over Precision: DECIMAL can be specified with a precision (total number of digits) and scale (number of digits after the decimal point), giving you fine-grained control over the storage requirements.

#### Example
Schema
SQL
CREATE TABLE orders (
    id INT AUTO_INCREMENT PRIMARY KEY,
    order_date DATE,
    amount DECIMAL(10, 2)
);
Precision Specified: DECIMAL(10, 2) means we can store up to 10 digits in total with 2 digits after the decimal point. This is suitable for most financial applications.

#### Common Mistakes
and Follow-Ups
Mistake: Using FLOAT or DOUBLE for monetary amounts without understanding their limitations.

#### Follow-Up
Considerations like transactional consistency, database normalization, and performance optimization when dealing with large datasets.

In summary, DECIMAL is the preferred data type for storing monetary amounts due to its precision and accuracy. Understanding these nuances ensures reliable financial calculations in your applications.

**Code:**

```sql
-- Example schema creation
CREATE TABLE orders (
    id INT AUTO_INCREMENT PRIMARY KEY,
    order_date DATE,
    amount DECIMAL(10, 2)
);
```

