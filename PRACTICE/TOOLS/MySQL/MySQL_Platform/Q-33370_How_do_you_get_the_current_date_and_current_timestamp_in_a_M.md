---
type: "practice"
difficulty: "Basic"
---

# How do you get the current date and current timestamp in a MySQL query?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Date Arithmetic](<../../../../TOOLS/MySQL/MySQL_Platform/Date_Arithmetic.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To retrieve the current date and timestamp in a MySQL query, we use specific functions provided by MySQL to get these values. The CURRENT_DATE() function returns today’s date without time, while NOW() or CURRENT_TIMESTAMP() provides both the current date and time.

Let's dive into the details:

**Code:**

```sql
-- Getting the current date (without time)
SELECT CURRENT_DATE();

-- Getting the current timestamp (date + time)
SELECT NOW();
```

