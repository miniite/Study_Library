---
type: "practice"
difficulty: "Intermediate"
---

# How would you round, floor, and ceil a numeric value in a query, and where do these matter for money?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Storing Money](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Storing_Money.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with SQL databases to handle financial data, rounding, flooring, and ceiling operations are crucial to ensure accuracy and compliance with business rules. These operations can be performed using specific SQL functions depending on the database system you're using (e.g., PostgreSQL, MySQL, SQLite). For this explanation, let's focus on how these operations work in a typical SQL environment.

**Code:**

```sql
-- Rounding a numeric value to two decimal places in PostgreSQL
SELECT ROUND(123.456, 2) AS rounded_value; -- Output: 123.46

-- Flooring a numeric value down to the nearest whole number in MySQL
SELECT FLOOR(123.999) AS floored_value; -- Output: 123

-- Ceiling a numeric value up to the nearest whole number in SQLite
SELECT CEIL(123.001) AS ceiling_value; -- Output: 124
```

**Example input / output:**

| Input          | Rounding       | Flooring       | Ceiling      |
|----------------|----------------|---------------|--------------|
| 123.456        | 123.46         | 123           | 124          |
| 123.999        | 124            | 123           | 124          |
| -123.001       | -123           | -124          | -123         |

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Input Value
8. Rounding
9. Value >= 0.5?
10. Round Up
11. Round Down
12. Flooring
13. Value < 0?
14. Floor Down
15. Floor Same as Input
16. Ceiling
17. Value > 0.5?
18. Ceil Up
19. Ceil Same as Input

