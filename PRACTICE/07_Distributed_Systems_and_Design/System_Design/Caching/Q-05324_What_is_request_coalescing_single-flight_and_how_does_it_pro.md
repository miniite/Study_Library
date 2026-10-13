---
type: "practice"
difficulty: "Deep"
---

# What is 'request coalescing' (single-flight) and how does it protect the database during a stampede?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Cache Stampede Prevention](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Cache_Stampede_Prevention.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Request coalescing, also known as single-flight or batched requests, is a caching technique designed to mitigate the impact of high traffic on databases by ensuring that multiple client requests are combined into a single request before they reach the database. This mechanism helps protect the database from being overwhelmed and can significantly reduce the load during a "stampede," where a large number of simultaneous client requests occur due to an external trigger, such as user panic or sudden viral content.

In essence, request coalescing works by temporarily buffering incoming requests until it receives enough requests to form a batch. Once sufficient requests are accumulated, they are combined into a single request that is then sent to the database. This strategy ensures that the database does not receive an overwhelming number of individual queries at once, thereby reducing the risk of overloading and potential failures.

**Comparison:**

```
Request	Combined Query
1	SELECT * FROM users WHERE id=1
2	INSERT INTO orders (user_id, product_id) VALUES (2, 3)
3	UPDATE products SET stock=stock-1 WHERE id=4
```

**Code:**

```python
def coalesce_requests(requests_queue: List[Request], threshold: int) -> Optional[str]:
    combined_request = None
    while len(requests_queue) >= threshold:
        batched_queries = [req.query for req in requests_queue[:threshold]]
        combined_request = " ".join(batched_queries)
        # Process the combined request
        process_request(combined_request)
        # Clear the processed requests from the queue
        requests_queue = requests_queue[threshold:]

    return combined_request
```

**Example input / output:**

```

Given the requests list and a threshold of 2:

Input:

JSON
Copy
{
  "requests": [
    {"query": "SELECT * FROM users WHERE id=1"},
    {"query": "INSERT INTO orders (user_id, product_id) VALUES (2, 3)"},
    {"query": "UPDATE products SET stock=stock-1 WHERE id=4"}
  ],
  "threshold": 2
}

Output:

JSON
Copy
{
  "combined_request": "SELECT * FROM users WHERE id=1 INSERT INTO orders (user_id, product_id) VALUES (2, 3)"
}
```

**Data:**

```json
{
  "requests": [
    {"query": "SELECT * FROM users WHERE id=1"},
    {"query": "INSERT INTO orders (user_id, product_id) VALUES (2, 3)"},
    {"query": "UPDATE products SET stock=stock-1 WHERE id=4"}
  ]
}
```

**Flow:**

1. No
2. Yes
3. Client Request
4. Is Queue Full?
5. Cache Buffer
6. Collect Requests UntilThreshold
7. Threshold Reached?
8. Combine Requests
9. Send Combined Request to DB
10. Process Response
11. Clear Buffer
12. Wait for Next Batch

