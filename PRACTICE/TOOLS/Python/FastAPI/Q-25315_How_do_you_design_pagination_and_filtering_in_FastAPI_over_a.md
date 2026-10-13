---
type: "practice"
difficulty: "Deep"
---

# How do you design pagination and filtering in FastAPI over a 500M-row table so that deep pages stay fast and the query planner does not fall off a cliff?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Cursor Pagination](<../../../../TOOLS/Python/FastAPI/Cursor_Pagination.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design efficient pagination and filtering for a 500 million row table in FastAPI, we need to ensure both performance and scalability. The core challenge is to avoid overwhelming the query planner with complex queries that can lead to poor performance as the data grows. Here’s how we approach this:

Summary: We will use advanced techniques like SQL query optimization, indexing, and leveraging FastAPI's powerful dependency injection and validation capabilities. By implementing a combination of database-level optimizations and API design best practices, we can ensure efficient pagination and filtering even for deep pages.

#### Triage
The question is probing how you handle large datasets efficiently in FastAPI while ensuring the query planner remains effective. We need to make sure our queries are optimized, and we minimize the load on the database by fetching only necessary data. Key assumptions include using a relational database like PostgreSQL and leveraging its advanced features.

#### First Principles Solution

Database Optimization: Use indexes strategically to speed up query execution.
Query Design: Write parameterized queries that can handle dynamic filtering efficiently.
Pagination: Implement efficient pagination techniques that minimize the data fetched from the database.
Caching: Cache results to reduce database load for frequently accessed data.

Detailed Steps:

Indexing Strategy: Create indexes on columns used in WHERE and ORDER BY clauses. For a 500 million row table, we might index user_id, created_at, or other commonly filtered fields.
SQL
CREATE INDEX idx_user_filter ON users(user_id);
Query Parameterization: Use parameterized queries to handle dynamic filtering conditions.
PYTHON
from fastapi import APIRouter, Depends, Query

router = APIRouter()

@router.get("/users")
async def get_users(
    user_id: int | None = None,
    created_at: str | None = Query(None),
    offset: int = 0,
    limit: int = 10
):
    query = "SELECT * FROM users"
    params = []
    if user_id:
        query += " WHERE user_id = %s"
        params.append(user_id)
    if created_at:
        query += " AND created_at > %s"
        params.append(created_at)

    return await execute_query(query, params, offset, limit)
Pagination Implementation: Use LIMIT and OFFSET to fetch a specific subset of data. For deep pages, we can implement server-side pagination.
PYTHON
async def execute_query(query, params, offset, limit):
    # Execute the query with parameters and return results
    result = await database.fetch_all(query=query, values=params)
    return list(result)[offset:offset + limit]
Caching: Use caching mechanisms like Redis to store frequently accessed data.
PYTHON
from fastapi import FastAPI
from redis import StrictRedis

app = FastAPI()
cache = StrictRedis()

@app.get("/users")
async def get_users(
    user_id: int | None = None,
    created_at: str | None = Query(None),
    offset: int = 0,
    limit: int = 10
):
    key = f"users:{user_id}:{created_at}:{offset}:{limit}"
    data = cache.get(key)
    if data:
        return json.loads(data)

    query, params = build_query(user_id, created_at)
    result = await execute_query(query, params, offset, limit)
    cache.set(key, json.dumps(result))
    return result

#### Closing Summary
By using indexes, parameterized queries, and caching, we can ensure that pagination and filtering are efficient even for large datasets like a 500 million row table in FastAPI.

#### Common Mistakes/Follow-Ups

#### Common mistakes
include not indexing frequently queried fields or using inefficient pagination techniques.

#### Follow-up
questions might involve discussing how to handle concurrent reads and writes, implementing rate limiting, or optimizing queries for specific use cases like real-time data updates.

