---
type: "practice"
difficulty: "Basic"
---

# What is the result of an arithmetic operation involving a NULL value in MySQL?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ MySQL Query Basics](<../../../../TOOLS/MySQL/MySQL_Platform/MySQL_Query_Basics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In MySQL, when you perform an arithmetic operation involving a NULL value, the result will always be NULL. This behavior is consistent with the way SQL handles NULL, which represents missing or unknown values. Let's break down this concept from first principles and explore some common scenarios.

**Code:**

```sql
-- Example 1: Addition involving NULL
SELECT 5 + NULL;  -- Result: NULL

-- Example 2: Subtraction involving NULL
SELECT 8 - NULL;  -- Result: NULL

-- Example 3: Multiplication involving NULL
SELECT 4 * NULL;  -- Result: NULL

-- Example 4: Division by NULL (special case)
SELECT 10 / NULL; -- Result: NULL

-- Example 5: Modulo operation with NULL (special case)
SELECT 9 % NULL;  -- Result: NULL
```

**Example input / output:**

```text
Input:
- Query 1: SELECT 5 + NULL;
- Query 2: SELECT 8 - NULL;
- Query 3: SELECT 4 * NULL;
- Query 4: SELECT 10 / NULL;
- Query 5: SELECT 9 % NULL;

Output:
- Result of Query 1: NULL
- Result of Query 2: NULL
- Result of Query 3: NULL
- Result of Query 4: NULL
- Result of Query 5: NULL
```

