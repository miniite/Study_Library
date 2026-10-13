---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd handle a "thundering herd" of cache misses all issuing the same expensive MySQL query at once, at both the app and DB layer.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ MySQL Performance Diagnosis](<../../../../TOOLS/MySQL/MySQL_Platform/MySQL_Performance_Diagnosis.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When handling a thundering herd scenario in a backend application where many requests simultaneously miss the cache and hit an expensive MySQL query, we need to address this issue from both the application and database layers to ensure efficient and scalable performance.

At the application layer, our primary goal is to minimize redundant SQL queries by leveraging caching mechanisms effectively. At the database level, we can optimize the query execution and ensure proper resource management to handle the load without overwhelming the server. Let's dive into a detailed step-by-step solution starting from first principles.

**Comparison:**

```
Time	Request ID	Cache Hit	Query Execution (ms)
08:00	A	No	100
08:01	B	No	100
08:02	C	Yes	-
08:03	D	Yes	-
```

**Code:**

```python
# Example of an application-level cache implementation using Flask and Redis
from flask import Flask
import redis

app = Flask(__name__)
cache = redis.Redis(host='localhost', port=6379, db=0)

def get_data_from_db(query):
    # Check if data is already cached
    key = f"dbquery:{query}"
    cached_data = cache.get(key)

    if cached_data:
        return cached_data.decode('utf-8')

    # Execute the query and store in cache
    db_result = execute_query(query)
    cache.setex(key, 3600, db_result)  # Cache for one hour

    return db_result

def execute_query(query):
    # Simulate database execution
    import sqlite3
    conn = sqlite3.connect('database.db')
    cursor = conn.cursor()
    cursor.execute(query)
    result = cursor.fetchall()
    conn.close()
    return result
```

**Example input / output:**

```text
Input:
- Simultaneous requests to `/get_user/123`

Output:
- First request retrieves data from the database and stores it in Redis.
- Subsequent requests hit the cache until expiration.
```

**Data:**

```json
{
  "query": "SELECT * FROM users WHERE id = ?"
}
```

**Flow:**

1. Yes
2. No
3. Cache Miss
4. Execute Query?
5. Store Result in Cache
6. Retrieve from Cache
7. Return Data to Requestor

