---
type: "practice"
difficulty: "Basic"
---

# What is a user-defined function (UDF) and when would you create one?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Scalar vs Table-valued Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/Scalar_vs_Table-valued_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

User-defined functions (UDFs) are custom functions created by database developers to perform specific tasks or computations within SQL queries. They allow users to encapsulate logic that can be reused across multiple statements, improving code readability and maintenance. UDFs are particularly useful when a set of operations needs to be performed repeatedly in different parts of an application.

To understand when you would create a UDF, we need to consider the problem domain, performance implications, and reusability benefits. UDFs are especially valuable when you have complex logic that can be reused multiple times or when you want to hide implementation details from users.

