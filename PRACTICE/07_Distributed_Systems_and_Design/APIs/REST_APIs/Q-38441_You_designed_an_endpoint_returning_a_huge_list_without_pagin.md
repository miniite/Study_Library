---
type: "practice"
difficulty: "Deep"
---

# You designed an endpoint returning a huge list without pagination and it timed out in production. How would you fix it?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Long-running Operations](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Long-running_Operations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a timeout when serving a huge list via a REST API, we need to focus on optimizing both the backend logic and the frontend handling. The core challenge is efficiently retrieving and returning data without overwhelming either endpoint.

#### Triage

The question revolves around understanding why an endpoint might time out and what strategies can be employed to prevent such timeouts in production. The interviewer is likely probing for knowledge of best practices in API design, efficient database querying, caching mechanisms, and the overall architecture. We assume that the current implementation does not handle large datasets efficiently.

#### First-Principles Solution
Optimize Query Efficiency: Ensure the SQL query fetching data from the database is optimized.
Implement Pagination or Chunking: Paginate the response to avoid sending all data at once.
Caching: Use caching mechanisms to reduce the load on the backend and improve performance for repeated requests.
Rate Limiting: Implement rate limiting to prevent abuse and ensure fair usage.
Step-by-Step Solution:
1. Optimize Query Efficiency

First, we need to examine the SQL query used to fetch data from the database:

SQL
SELECT * FROM users WHERE ...

This might be inefficient if it doesn’t use proper indexes or joins, which can cause timeouts due to slow execution times.

Improvement:

SQL
SELECT id, name, email FROM users WHERE ... LIMIT 100 OFFSET 0;

Using LIMIT and OFFSET, we can fetch data in smaller chunks. Ensure the query includes necessary indexes for faster performance.

2. Implement Pagination or Chunking

Instead of returning all data at once, return it in manageable chunks:

HTTP
GET /users?page=1&size=50

Handle this request on the backend by fetching and sending only the relevant chunk:

PYTHON
def get_users(request):
    page = int(request.query_params.get('page', 1))
    size = int(request.query_params.get('size', 50))

    # Fetch data from database using query parameters
    users = User.objects.paginate(page, size)

    return Response(users)
3. Caching Mechanisms

Use caching to store frequently accessed data:

PYTHON
from django.core.cache import cache

def get_users(request):
    key = f'users_{page}_{size}'
    users = cache.get(key)

    if not users:
        # Fetch from database and cache the result
        users = User.objects.paginate(page, size)
        cache.set(key, users, timeout=3600)  # Cache for 1 hour

    return Response(users)
4. Rate Limiting

Implement rate limiting to control the number of requests within a given time frame:

PYTHON
from django_ratelimit.decorators import ratelimit

@ratelimit(key='user_or_ip', rate='5/m')
def get_users(request):
    # Handle request as usual

#### Closing Summary
+ Common Mistakes/Follow-Ups

By implementing these strategies, we can significantly reduce the likelihood of timeouts. It's crucial to balance between optimizing for performance and ensuring that the system remains responsive under various loads.

#### Common Mistakes

Not using proper query optimization.
Failing to implement pagination or chunking.
Overlooking caching mechanisms, leading to unnecessary database hits.
Ignoring rate limiting, which can cause abuse and overload the server.

For follow-ups, the interviewer might ask about additional considerations such as:

How would you handle concurrent requests?
What about handling large files or streaming responses?

