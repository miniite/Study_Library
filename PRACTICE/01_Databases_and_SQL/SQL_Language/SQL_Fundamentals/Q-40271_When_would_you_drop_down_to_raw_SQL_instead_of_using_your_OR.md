---
type: "practice"
difficulty: "Intermediate"
---

# When would you drop down to raw SQL instead of using your ORM's query builder?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ SQL vs NoSQL](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/SQL_vs_NoSQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In an intermediate Fullstack SDE role, you might encounter scenarios where the built-in capabilities of ORMs (Object-Relational Mappers) fall short in terms of performance or flexibility. Dropping down to raw SQL can be necessary when working with complex queries, optimizing database interactions, or leveraging advanced database features that aren't easily accessible through ORM query builders.

#### Triage

The question is essentially asking under what circumstances and for what reasons you would choose to write direct SQL statements instead of using the ORM's query builder. This involves understanding both the strengths and limitations of ORMs versus raw SQL, as well as considering factors like performance, database-specific features, and developer experience.

Triaging the Scenario

#### Assumptions

You are familiar with a popular ORM (e.g., Sequelize for Node.js).
The project requirements might include complex queries or advanced database operations.
Performance is a critical factor in your application's success.

Reasoning from First Principles:

Complex Queries: ORMs often abstract the underlying SQL logic, making it difficult to construct highly optimized and complex queries (e.g., multi-table joins with intricate conditions).
Performance Optimization: When you need fine-grained control over query execution plans or want to avoid the overhead of ORM abstractions.
Advanced Database Features: To leverage features specific to a particular database system that are not supported by your chosen ORM (e.g., window functions in PostgreSQL, raw SQL for full-text search).
Readability and Maintainability: For simpler queries, ORMs can produce more readable code. However, complex logic might be harder to follow when written in SQL.
Step-by-Step Solution

Identify the Requirement:

Determine if a specific query is too complex for your ORM.
Evaluate whether the performance gains from raw SQL outweigh the overhead of switching contexts.

#### Performance Considerations

Measure the performance impact of using both ORMs and raw SQL to ensure that the former does not significantly degrade application performance.

Database-Specific Features:

Check if there are specific database features or functions (e.g., window functions, full-text search) that can be leveraged with raw SQL but not through your ORM.

Developer Experience and Maintenance:

#### Cons
ider the readability and maintainability of code written in both styles. For complex queries, a mix of ORMs for simple logic and raw SQL for critical parts might be a better approach.

#### Closing Summary

In summary, you would drop down to raw SQL when dealing with highly complex or performance-critical queries that are not easily managed by your ORM. It’s essential to weigh the trade-offs between readability, maintainability, and performance. A balanced approach often involves using ORMs for simpler logic and raw SQL where necessary.

#### Common Mistakes
and Follow-Ups
Mistake: Overusing raw SQL in every query without considering the overhead of switching contexts.

#### Follow-Up
Always profile and benchmark to ensure that your choice yields tangible performance benefits.

**Comparison:**

```
User ID	Average Score
1	87.5
```

**Code:**

```sql
-- Example of a complex raw SQL query
SELECT
    u.id AS user_id,
    AVG(s.score) AS average_score
FROM
    users u
JOIN
    scores s ON u.id = s.user_id
WHERE
    u.status = 'active'
GROUP BY
    u.id;
```

**Example input / output:**

```text
Input:
- Raw SQL query for calculating average scores of active users.

Output:
- A table with user IDs and their corresponding average scores.
```

**Data:**

```json
{
  "users": [
    {"id": 1, "status": "active"},
    {"id": 2, "status": "inactive"},
    {"id": 3, "status": "active"}
  ],
  "scores": [
    {"user_id": 1, "score": 85},
    {"user_id": 1, "score": 90},
    {"user_id": 2, "score": 75}
  ]
}
```

**Flow:**

1. No
2. Yes
3. Identify Requirement
4. Is ORM Sufficient?
5. Use Raw SQL
6. Optimize ORMs
7. Measure Performance Gains
8. Implement Changes

