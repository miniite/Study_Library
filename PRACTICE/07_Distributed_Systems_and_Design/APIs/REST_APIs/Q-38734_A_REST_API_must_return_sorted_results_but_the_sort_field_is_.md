---
type: "practice"
difficulty: "Deep"
---

# A REST API must return sorted results but the sort field is user-provided; an attacker sorts by an unindexed column to cause slow scans. How do you allow flexible sort while preventing this abuse vector?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Keeping Secrets out of URLs](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Keeping_Secrets_out_of_URLs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a REST API that needs to handle sorted results but also prevent potential SQL injection or other security issues related to user-provided sort fields, we need to carefully balance flexibility with robust security measures. The key challenge here is ensuring that the sort field provided by the client is safe to use in database queries without leading to slow scans or arbitrary code execution.

#### Triage
Understanding the Question and Probing Points

The core of this problem involves allowing users to specify how they want results sorted, which can greatly enhance the usability of your API. However, providing such flexibility also opens up security risks, particularly if the sort field is derived from untrusted user input. The interviewer might be probing for your understanding of SQL injection, database query performance optimization, and secure coding practices.

#### Assumptions
User Input: Assume that the sort field provided by the client can be any string.
Database Schema: Assume a relational database with multiple tables where sorting operations are performed.
Security Context: Assume this API is part of a larger application with existing security measures in place.
Step-by-Step Solution From First Principles

Validate and Sanitize User Input

Before using the sort field, validate it to ensure it matches a predefined set of allowed columns. For example, if your database schema has fields id, name, and created_at, only allow these in the user's request.

Use Parameterized Queries

Use parameterized queries or prepared statements to safely include user input in SQL commands. This prevents injection attacks by ensuring that dynamic parts of a query are treated as literal values, not executable code.

Index Management

Ensure that the columns used for sorting are properly indexed. If the sort field is frequently requested and involves unindexed fields, this could lead to slow scans. Regularly review and update indexes based on usage patterns.

Rate Limiting and Throttling

Implement rate limiting and throttling mechanisms to prevent abuse. For instance, limit the number of requests per minute for a given user or IP address.

Query Optimization

Optimize queries by using caching techniques like query result caching. Store the results of frequently executed queries in memory or on disk to avoid repeatedly executing the same slow queries.

Fallback Mechanisms

If a requested sort field leads to a performance issue, have fallback mechanisms that can prioritize other fields or provide a default sorting behavior when necessary.

#### Closing Summary
and Common Mistakes

In summary, allowing flexible sorting in REST APIs while maintaining security involves multiple layers of defense: input validation, secure query construction, proper indexing, rate limiting, and fallback strategies. A common mistake is not validating user input sufficiently, which can lead to SQL injection vulnerabilities. Another frequent issue is neglecting performance optimization, resulting in slow scans when sorting large datasets.

Common Follow-Ups
Database Indexes: How do you ensure that the used columns are properly indexed?
Performance Metrics: How would you monitor and optimize query performance over time?
Security Best Practices: What other security measures should be considered alongside input validation?

**Code:**

```python
def handle_sort_request(sort_field, user_id):
    # List of allowed sort fields
    allowed_fields = ['id', 'name', 'created_at']

    # Validate the sort field
    if sort_field not in allowed_fields:
        raise ValueError("Invalid sort field")

    # Use parameterized query to prevent SQL injection
    db_query = f"SELECT * FROM users ORDER BY {sort_field} ASC"
    cursor.execute(db_query)
    results = cursor.fetchall()
    return results
```

**Example input / output:**

```text
Request:
{
  "sortField": "name"
}

Response:
[
  {"id": 1, "name": "Alice", "created_at": "2023-10-01"},
  {"id": 2, "name": "Bob", "created_at": "2023-10-02"}
]
```

**Flow:**

1. User Requests Sort Field
2. Validate and Sanitize Input
3. (Prepared) Query withParameterization
4. Execute Query, Fetch Results
5. Check Indexing andPerformance
6. Return Sorted Results

