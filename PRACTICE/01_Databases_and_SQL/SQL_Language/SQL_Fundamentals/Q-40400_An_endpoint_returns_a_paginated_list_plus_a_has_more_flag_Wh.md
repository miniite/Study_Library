---
type: "practice"
difficulty: "Intermediate"
---

# An endpoint returns a paginated list plus a 'has_more' flag. Why is fetching limit+1 rows better than a separate COUNT query for that flag, and when does even that break down?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Join Fan-out](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Join_Fan-out.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To efficiently determine if there are more pages of data in a paginated list using SQL, many developers might be tempted to use an additional COUNT query to check if any rows exist after the current page's limit. However, fetching limit + 1 rows is often a better approach because it avoids executing another round trip to the database and can provide more accurate results in certain scenarios.

#### Triaging the Question

The core of this question revolves around understanding the trade-offs between using an additional COUNT query versus fetching limit + 1 rows. The interviewer is probing your knowledge of SQL performance, pagination techniques, and how different implementations affect resource usage and accuracy.

#### Assumptions
We are dealing with a paginated list where each page has a certain number of items.
The goal is to determine if there is more data available for the next page.
We have control over both the SELECT statement that fetches the rows and any additional COUNT queries.

#### First Principles Solution
Why Fetching limit + 1 Rows is Better

Reduced Network Latency:

Executing a single query to fetch more rows than necessary reduces network latency compared to executing two separate queries (one for fetching the data and another for counting).

#### Cons
istency and Accuracy:

When you fetch limit + 1 rows, you can accurately determine if there are more pages left without needing to make a second query.
If the last row in your current fetch is not included in the count of visible rows (because it's one too many), then you know there are more items beyond this point.

Simplified Logic:

The logic required to implement pagination becomes simpler and less error-prone when using a single query.
You can directly use the number of fetched rows minus the page size to set your has_more flag.

#### Example
Implementation

Assume you have a table items, and you are fetching 10 items per page. Here’s how you would implement it in SQL:

SQL
SELECT * FROM items ORDER BY some_column LIMIT 10 OFFSET (page_number - 1) * 10;

To determine if there is more data, you would fetch one extra row and then check the count of returned rows:

SQL
WITH fetched AS (
    SELECT * FROM items ORDER BY some_column LIMIT 11 OFFSET (page_number - 1) * 10
)
SELECT COUNT(*) > 10 AS has_more FROM fetched;

If has_more is true, it means there are more than 10 rows in the result set, indicating that there are at least 11 items, and thus more pages.

Edge Cases and Common Pitfalls

Large Data Sets:

If your dataset is extremely large, fetching limit + 1 rows might still be inefficient. In such cases, using a separate COUNT query could be necessary.

Database Indexing:

Ensure that appropriate indexes are in place to optimize both the main query and any additional COUNT queries.

#### Closing Summary

Fetching limit + 1 rows is generally more efficient for determining if there is more data on the next page. However, this approach has its limitations, especially with very large datasets or complex query conditions. Understanding these trade-offs helps in making informed decisions about pagination strategies.

#### Common Mistakes
and Follow-Ups

Failing to Understand Latency Implications:

Developers often overlook the importance of minimizing network latency by reducing the number of database round trips.

Overlooking Large Data Sets:

For very large datasets, a separate COUNT query might be more appropriate despite the additional overhead.

Ignoring Indexing and Query Optimization:

Not considering how indexes affect performance can lead to suboptimal queries and poor application performance.

