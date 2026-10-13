---
type: "practice"
difficulty: "Intermediate"
---

# How would you use SQL to sample 100 random conversations for a manual quality review, and why avoid ORDER BY RANDOM on a huge table?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Scalar Subqueries](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Scalar_Subqueries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To perform a random sampling of 100 conversations from a large database using SQL, we need to carefully choose a method that ensures fairness and efficiency. In this context, we will use techniques like Common Table Expressions (CTEs) and window functions to achieve our goal. Let's dive into the solution step by step.

First, it is important to understand why ORDER BY RANDOM can be problematic on large tables. When you use ORDER BY RANDOM, the database has to sort all rows in the table before returning the first 100. This operation is both time-consuming and resource-intensive, especially as the size of the table grows.

We will instead leverage window functions, specifically ROW_NUMBER() or RANK(), to achieve a more efficient and fair sampling.

**Code:**

```sql
WITH OrderedConversations AS (
    SELECT
        conversation_id,
        message,
        ROW_NUMBER() OVER (ORDER BY random()) as row_num
)
SELECT
    conversation_id,
    message
FROM
    OrderedConversations
WHERE
    row_num <= 100;
```

