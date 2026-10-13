---
type: "practice"
difficulty: "Basic"
---

# In one sentence, how would you explain caching to a non-technical teammate?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Caching Fundamentals](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Caching_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Caching is like keeping copies of frequently used information in your backpack so you don't have to carry all that information with you everywhere you go. Whenever you need to use that information, you just grab it from your backpack instead of going back to where you stored the original.

To break down what we're doing:

What is caching? Caching involves storing copies of data in a place that can serve it faster than fetching it directly.
Why do we use caching? We use caching to speed up access to frequently requested or slowly generated data. Think of it like having a quick-reference book for things you use often, rather than always going back to the original source.

Let's dive into the first principles: Caching works by storing the results of expensive function calls and returning the cached result when the same inputs occur again. This reduces latency and computational load because instead of recomputing or re-fetching data from a slower source, we use the cached version which is much faster to access.

For example, imagine you're building an online store where product details are fetched from a database. If a customer frequently visits the product page for a specific item, fetching that data every time could be slow and inefficient. Instead, the first time the data is requested, we fetch it and cache it in memory or another fast storage system (like Redis). On subsequent requests, instead of hitting the database again, we serve the cached version which is much faster.

There are a few key things to consider:

Where do you store the cache? Common choices include in-memory caches like Redis, distributed caches, and even browser-based caching.
When should data be invalidated or refreshed? For example, if the underlying data changes, we need to ensure our cache is updated. This could happen through expiration policies, manual invalidation, or using timestamps.

In summary, caching helps us make systems faster by storing frequently accessed data in a fast-access location and reusing it when needed.

**Comparison:**

```
Cache Type	Storage Location	Example Use Case
In-Memory	RAM	User sessions
Distributed	Multiple Servers	API responses
Browser	Local Storage	Web page caching
```

**Code:**

```python
# Example of basic caching in Python with Flask and Redis
from flask import Flask, make_response
import redis

app = Flask(__name__)
redis_client = redis.Redis(host='localhost', port=6379, db=0)

@app.route('/product/<int:id>')
def get_product(id):
    key = f'product:{id}'

    # Check if the product is cached
    data = redis_client.get(key)
    if data:
        return make_response(data.decode('utf-8'))

    # If not, fetch from database (simplified example)
    data = "Some slow and expensive query to get product details"
    redis_client.set(key, data, ex=3600)  # Cache for 1 hour
    return make_response(data)
```

**Example input / output:**

```

Request:

CODE
Copy
GET /product/42

Response (First Request):

CODE
Copy
"Product Details for ID 42"

Response (Subsequent Requests within Cache TTL):

CODE
Copy
"Product Details for ID 42"
```

**Data:**

```json
{
  "cache_key": "product:42",
  "cache_value": "Product Details for ID 42"
}
```

**Flow:**

1. Request for Data
2. Check Cache
3. If Cache Miss, Fetch fromSource and Store in Cache
4. Return Cached Data
5. If Cache Hit, Return CachedData

