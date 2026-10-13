---
type: "practice"
difficulty: "Intermediate"
---

# How would you represent a nested or related resource, e.g. all orders belonging to a specific user?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Filtering Sorting and Pagination](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Filtering_Sorting_and_Pagination.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To represent a nested or related resource in a REST API, such as all orders belonging to a specific user, we need to ensure that our design is both intuitive and efficient. At a high level, this involves using appropriate HTTP methods and status codes, handling pagination if necessary, and ensuring proper error responses.

#### Triage

The core of the question revolves around how to structure API endpoints for retrieving related data. Specifically, it tests your understanding of REST principles, HTTP methods, query parameters, and response formats.

#### Assumptions
We are working with a full-stack SDE role focusing on backend development.
The user ID is known in advance (e.g., passed as a parameter or extracted from authentication tokens).
Pagination might be required if the number of orders exceeds a reasonable limit per request.

#### First-Principles Solution
Endpoint Design: Use an appropriate HTTP method and path to indicate that we are fetching related resources.
Query Parameters: Leverage query parameters for pagination, filtering, sorting, etc.
HTTP Status Codes: Ensure proper use of status codes (e.g., 200 OK for successful requests).
Response Format: Use JSON or another suitable format to return the data.
Step-by-Step Solution

Endpoint Path:

The path should clearly indicate that we are fetching orders related to a user.
HTTP
GET /users/{userId}/orders

Query Parameters:

Use query parameters for pagination and filtering.
HTTP
GET /users/1/orders?limit=10&offset=0&sort=-createdAt

HTTP Status Codes:

200 OK: Success, the request was successful.
400 Bad Request: Invalid parameters or missing required fields.
401 Unauthorized: Authentication failed (e.g., invalid or expired token).
404 Not Found: User not found.

Response Format:

Return an array of order objects, with relevant information such as ID, createdAt, status, etc.
JSON
{
  "orders": [
    {
      "id": 1,
      "userId": 1,
      "createdAt": "2023-10-01T10:00:00Z",
      "status": "completed"
    },
    ...
  ],
  "totalPages": 5
}

#### Closing Summary
and Common Mistakes/Follow-Ups

The key is to ensure that the API design aligns with REST principles while being practical for real-world use. Common mistakes include:

Failing to validate input parameters.
Not handling pagination efficiently, leading to performance issues.
Returning incomplete or inconsistent data in responses.

#### Follow-ups
might involve discussing how to handle errors robustly, caching strategies, and performance optimization techniques like lazy loading of related resources.

**Comparison:**

```
Parameter	Type	Description
userId	integer	User ID
limit	integer	Number of orders to return
offset	integer	Offset for pagination
sort	string	Sorting field and direction
```

**Code:**

```python
# Example Python code for a Flask backend route
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/users/<int:user_id>/orders', methods=['GET'])
def get_orders(user_id):
    limit = int(request.args.get('limit', 10))
    offset = int(request.args.get('offset', 0))
    sort = request.args.get('sort', '-createdAt')

    # Dummy data for demonstration
    orders = [
        {"id": i, "userId": user_id, "createdAt": f"2023-10-{i:02d}T{i:02d}:00Z", "status": "completed"}
        for i in range(5)
    ]

    # Apply sorting
    orders.sort(key=lambda x: x['createdAt'], reverse=sort.startswith('-'))

    # Apply pagination
    orders = orders[offset:offset+limit]

    return jsonify({"orders": orders, "totalPages": 1})
```

**Example input / output:**

```text
Request:
GET /users/1/orders?limit=10&offset=0&sort=-createdAt

Response:
{
  "orders": [
    {
      "id": 1,
      "userId": 1,
      "createdAt": "2023-10-05T05:00:00Z",
      "status": "completed"
    },
    ...
  ],
  "totalPages": 5
}
```

**Data:**

```json
{
  "userId": 1,
  "orders": [
    {
      "id": 1,
      "userId": 1,
      "createdAt": "2023-10-01T01:00:00Z",
      "status": "completed"
    },
    ...
  ],
  "totalPages": 5
}
```

**Flow:**

1. Invalid
2. Valid
3. User Requests Orders
4. Extract User ID
5. Check Authentication Token
6. Return 401 Unauthorized
7. Fetch Related Orders
8. Paginate and Sort Orders
9. Format Response with Ordersand Pagination Info
10. Return JSON Response to User

