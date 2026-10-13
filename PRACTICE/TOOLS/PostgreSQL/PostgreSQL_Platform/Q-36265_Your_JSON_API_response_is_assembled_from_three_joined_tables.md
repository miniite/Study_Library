---
type: "practice"
difficulty: "Intermediate"
---

# Your JSON API response is assembled from three joined tables and Node serialization is the bottleneck; when would you push row_to_json / jsonb_build_object into the query, and what's the risk of moving formatting into the DB?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ UUID vs Serial Keys](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/UUID_vs_Serial_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When optimizing a JSON API response in PostgreSQL, one key decision is whether to perform row serialization (e.g., using row_to_json or jsonb_build_object) within the database query versus doing so after fetching rows with Node.js. For an intermediate developer working on a PostgreSQL backend, understanding the performance implications and choosing the right approach involves considering factors like query complexity, network latency, and resource management.

#### Triage

The question is asking about when to push row_to_json or similar JSON formatting functions into the SQL query versus handling it in Node.js. The core of the issue revolves around balancing database load with network performance and ensuring efficient data transmission. Key assumptions include understanding that Node.js might be a bottleneck, implying high request volumes or complex processing.

#### First-Principles Solution

To determine when to push JSON serialization into the query, consider these steps:

Query Complexity: Evaluate if the query is simple enough to handle in SQL and whether it can produce the required JSON structure directly.
Network Latency: Consider network latency between the database server and application servers. Reducing the amount of data transmitted over the network by doing more work in the database can improve overall performance.
Resource Management: Analyze if pushing serialization into the query reduces load on the Node.js environment, allowing it to focus on other tasks.
Detailed Solution
Simple Queries:
If the queries are simple and return a small number of fields (e.g., a single row or a few columns), row_to_json can be pushed directly into the query.
SQL
SELECT row_to_json(t) FROM table_name t WHERE id = 1;
Complex Queries:
For more complex queries involving joins, aggregates, or nested structures, performing serialization in Node.js might be necessary to maintain performance.
Network Considerations:
If the response size is large, pushing JSON serialization into the query can reduce network bandwidth usage and improve client-side processing speed.
Risks of Moving Formatting Into the DB
Performance Impact: Complex queries with row_to_json or jsonb_build_object can become slow if not optimized.
Scalability Issues: As data grows, complex queries in the database might degrade performance and increase response times.
Maintainability: Mixing business logic in the database can make the code harder to maintain and test.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Pushing row_to_json or similar functions into the query is beneficial when dealing with simple, lightweight queries that reduce network load. However, for complex scenarios involving multiple joins or aggregations, it might be better to handle serialization in Node.js. The primary risk of moving JSON formatting into the database lies in performance degradation and increased complexity.

#### Common mistakes
include:

Overusing row_to_json without considering the query's complexity.
Ignoring network latency when data size is significant.
Failing to optimize queries that use extensive JSON functions.
Common Follow-Ups
What are some optimization techniques for complex queries?
How do you balance database and application layer caching in such scenarios?
Can you provide an example of a query that would benefit from moving serialization into the DB vs Node.js?

**Code:**

```sql
-- Example SQL Query with row_to_json
SELECT row_to_json(t)
FROM (
  SELECT column1, column2
  FROM table_name
  WHERE id = 1
) t;
```

