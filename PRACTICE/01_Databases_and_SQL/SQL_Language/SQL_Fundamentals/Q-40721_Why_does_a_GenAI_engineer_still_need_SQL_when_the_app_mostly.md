---
type: "practice"
difficulty: "Basic"
---

# Why does a GenAI engineer still need SQL when the app mostly calls an LLM API and a vector database?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Experiment Metrics in SQL](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Experiment_Metrics_in_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

As a GenAI engineer, understanding SQL is crucial even if your application primarily interacts with an LLM (Language Model) API and a vector database. SQL is essential because it provides a powerful, structured way to manage and query relational data. Whether the app calls an LLM or uses a vector database for embeddings, there will be instances where you need to work with traditional relational databases.

SQL allows you to:

Efficient Data Storage and Retrieval: Manage large datasets in a structured format.
Complex Querying: Perform intricate queries that are difficult or impossible with other methods.
Data Integrity and Consistency: Ensure data accuracy and consistency across your applications.

In this context, let's break down why SQL remains important:

**Comparison:**

```
ID	Name	Age
1	Alice	20
2	Bob	30
3	Charlie	40
```

**Code:**

```sql
-- Example of a simple SQL query to select users from a database
SELECT * FROM Users WHERE age > 18;
```

