---
type: "practice"
difficulty: "Basic"
---

# Explain why SELECT * can be slower than selecting only the columns you need.

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ How Indexes Are Used](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/How_Indexes_Are_Used/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with databases, a common practice is to use SELECT * in queries to retrieve all columns from a table. However, this approach can be less efficient compared to explicitly specifying the required columns. This difference in performance primarily stems from how database indexing and data retrieval work behind the scenes.

Firstly, let's understand the core idea: SELECT * retrieves all columns, whereas selecting specific columns can lead to more targeted data access. The key lies in understanding indexes and their role in query optimization.

