---
type: "practice"
difficulty: "Intermediate"
---

# How do you write a query to compute the ratio of two columns as a feature while safely handling division by zero?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Data Validation Queries](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Data_Validation_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute the ratio of two columns in SQL, we need to ensure that we handle division by zero gracefully. This is important because dividing by zero can result in an error or return a special value like infinity, which might not be meaningful in our context. Here’s how you would approach this problem step-by-step:

Understanding the Problem: We want to compute column_A / column_B for each row.
Handling Division by Zero: If column_B is zero, we should return a predefined value (e.g., 0) or handle it in another way that makes sense for our use case.

Let's dive into the SQL query solution.

