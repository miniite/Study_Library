---
type: "practice"
difficulty: "Deep"
---

# How does the database decide whether to use your index or ignore it, in simple terms?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ How Indexes Are Used](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/How_Indexes_Are_Used/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a simple terms approach, when a query is executed, the database optimizer evaluates several factors to decide whether to use an index or not. The key idea here is that using indices can speed up queries by allowing the database to quickly locate the required data without scanning entire tables. However, if improperly used, they can also slow down insert and update operations due to additional overhead.

To address this question, we need to break it down into its core components: understanding how the query optimizer makes decisions, the costs associated with using or not using an index, and practical scenarios where each approach might be more beneficial.

**Code:**

```sql
-- Example SQL query for demonstration purposes
SELECT * FROM customers WHERE age > 30 AND city = 'New York';
```

**Example input / output:**

```text
- **Input:** `SELECT * FROM customers WHERE age > 30 AND city = 'New York';`
- **Output:** The database decides whether to use an index or not based on the cost analysis.
```

