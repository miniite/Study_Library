---
type: "practice"
difficulty: "Basic"
---

# What does the ABS function return and when would you use it?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ NULL vs Zero in Aggregates](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/NULL_vs_Zero_in_Aggregates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The ABS function in SQL is a simple yet powerful tool that returns the absolute value of a number. It takes a single numeric argument and always returns a non-negative result, effectively converting negative numbers to positive ones while leaving positive numbers unchanged.

To use it effectively, we need to understand its basic syntax and practical applications.

**Code:**

```sql
-- Example usage
SELECT ABS(-10) AS AbsoluteValue;  -- Returns 10
```

