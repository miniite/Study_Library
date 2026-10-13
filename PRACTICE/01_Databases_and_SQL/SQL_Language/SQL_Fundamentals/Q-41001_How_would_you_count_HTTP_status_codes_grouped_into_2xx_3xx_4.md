---
type: "practice"
difficulty: "Intermediate"
---

# How would you count HTTP status codes grouped into 2xx/3xx/4xx/5xx classes from an access-log table using CASE?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Conditional Aggregation](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Conditional_Aggregation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To count HTTP status codes grouped into 2xx, 3xx, 4xx, and 5xx classes from an access-log table using SQL, we'll use a CASE statement to categorize the status codes, followed by aggregating the counts for each category. This approach ensures that we can effectively group and count the HTTP statuses according to their respective ranges.

