---
type: "practice"
difficulty: "Deep"
---

# How would you implement pagination for a large query result set in a Python backend, and what is keyset pagination?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Streaming Query Results](<../../../../TOOLS/Python/Python_Language/Streaming_Query_Results.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Pagination is essential when dealing with large datasets to ensure that your application can handle requests efficiently without overwhelming resources or the database. In this context, we will focus on implementing pagination in a Python backend and explore keyset pagination as one of its forms.

TL;DR:

Keyset pagination involves fetching records based on a specific field's value (usually an ID) rather than using offset-based counting. This method is more scalable for large datasets because it avoids the performance issues associated with offset-based pagination, such as expensive LIMIT and OFFSET queries in SQL.

#### Triage

The question asks about implementing pagination in Python for a backend system dealing with large query result sets. The interviewer is probing your understanding of different pagination techniques, particularly keyset pagination, to assess your ability to handle complex database operations efficiently. Assumptions include that the dataset can be paginated based on some order (e.g., by ID), and that the data is stored in a relational database.

#### First-Principles Solution
1. Understanding the Problem:
We need to fetch multiple pages of results from a large dataset.
The goal is to do this efficiently without overwhelming the database or memory resources.
2. Offset-Based Pagination (Common Approach):
Offset-based pagination uses LIMIT and OFFSET clauses in SQL queries. For example:
SQL
SELECT * FROM users LIMIT 10 OFFSET 90;
This method is straightforward but can lead to performance issues as the dataset grows, especially if OFFSET values are large.
3. Keyset Pagination:
Keyset pagination uses a key (usually an ID) from the previous page to fetch subsequent pages.
It ensures that each request retrieves only new data without needing to know the exact number of records or resorting to inefficient queries.
4. Implementation Steps in Python:

Database Query with Key Parameter:

PYTHON
def get_users(page_size=10, last_user_id=None):
    query = "SELECT * FROM users WHERE id > %s LIMIT %s"
    params = (last_user_id, page_size)
    # Execute the SQL query and fetch results
    return execute_query(query, params)

Client-Side Logic:

The client sends a request with page_size and an optional last_user_id.
On receiving a response, it extracts the last_user_id from the last item of the fetched list to use in subsequent requests.

Error Handling and Edge Cases:

Handle cases where no more records are available (e.g., return empty results).
Ensure that the implementation can handle large datasets without performance degradation.

#### Example
Workflow:

PYTHON
# Initial request
page_size = 10
last_user_id = None
response = get_users(page_size, last_user_id)

while response:
    for user in response['users']:
        process(user)  # Process each user as needed

    if 'next_last_user_id' in response:  # Assuming the response structure contains next_last_user_id
        last_user_id = response['next_last_user_id']
        response = get_users(page_size, last_user_id)
5. Performance Considerations and Scalability:
Keyset pagination is more scalable for large datasets because it reduces the load on the database by avoiding LIMIT and OFFSET.
It can be further optimized using indexes on the key field to ensure efficient retrieval of subsequent pages.

#### Closing Summary

Keyset pagination provides a robust solution for handling large datasets in Python backends. By leveraging a specific key (like an ID), we can efficiently fetch new data without the performance issues associated with offset-based pagination. This approach is particularly valuable when dealing with real-time, high-traffic applications where query efficiency and resource management are critical.

#### Common Mistakes
and Follow-Ups:
Mistake: Using LIMIT and OFFSET for every request, which can lead to suboptimal performance.

#### Follow-Up
Discussing how to handle edge cases such as no more data available or dealing with concurrent requests that might overwrite the key.

