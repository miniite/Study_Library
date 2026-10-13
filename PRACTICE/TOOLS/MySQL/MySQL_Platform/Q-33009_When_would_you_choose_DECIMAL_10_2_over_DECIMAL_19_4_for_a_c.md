---
type: "practice"
difficulty: "Intermediate"
---

# When would you choose DECIMAL(10,2) over DECIMAL(19,4) for a currency column and what does the second number control?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Date Arithmetic](<../../../../TOOLS/MySQL/MySQL_Platform/Date_Arithmetic.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between DECIMAL(10,2) and DECIMAL(19,4) for storing currency data in MySQL, the choice depends on the precision required, the range of values expected, and the trade-offs you are willing to make. Let’s break down each aspect step by step.

Firstly, understand what the numbers mean:

The first number (e.g., 10 or 19) specifies the total number of digits that can be stored in the field.
The second number (e.g., 2 or 4) specifies how many of those digits are to the right of the decimal point, which determines the precision.

Now, let's consider a scenario where you have a currency column for storing monetary values. For DECIMAL(10,2), you can store up to 99999999.99, and it offers two digits after the decimal point. On the other hand, DECIMAL(19,4) allows you to store values like 99999999999999999.9999, with four digits after the decimal point.

In most financial applications, a requirement for two decimal places is sufficient (e.g., $20.50). However, if your application deals with very large values or needs to handle fractional parts of a cent more precisely, DECIMAL(19,4) might be necessary.

To make the decision clearer:

If you are dealing with small amounts and high precision is not critical, use DECIMAL(10,2). This choice is simpler and uses less storage.
If your application requires handling large financial values or needs to store very precise fractional parts of a cent (e.g., 0.0001), go with DECIMAL(19,4).

#### Common mistakes
include choosing the wrong data type for the required precision, which can lead to data loss or inaccuracies, or using unnecessarily high precision, which wastes storage and performance resources.

**Code:**

```sql
-- Example of declaring DECIMAL columns in MySQL
CREATE TABLE example (
    id INT AUTO_INCREMENT PRIMARY KEY,
    amount DECIMAL(10,2),  -- Two decimal places for most currency values
    large_amount DECIMAL(19,4)  -- Four decimal places for precise financial data
);
```

**Example input / output:**

```

To illustrate, consider storing a value of $123.45:

SQL
Copy
INSERT INTO example (amount) VALUES (123.45);
SELECT * FROM example WHERE id = 1; -- Returns the inserted value with two decimal places

INSERT INTO example (large_amount) VALUES (1234567890.1234);
SELECT * FROM example WHERE id = 1; -- Returns the inserted value with four decimal places
```

**Data:**

```json
{
  "example_table": {
    "id": 1,
    "amount": 10.50,
    "large_amount": 2039876543.1234
  }
}
```

