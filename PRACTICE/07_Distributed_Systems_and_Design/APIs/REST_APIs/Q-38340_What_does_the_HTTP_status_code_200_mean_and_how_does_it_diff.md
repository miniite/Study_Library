---
type: "practice"
difficulty: "Basic"
---

# What does the HTTP status code 200 mean, and how does it differ from 201 and 204?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ HTTP Status Codes](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/HTTP_Status_Codes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

HTTP status codes are crucial in understanding the outcome of an HTTP request in a REST API. A 200 OK status code indicates that the request has succeeded, meaning the server has fulfilled the client's request and provided the requested data or resource. This is the most common success response.

In contrast, 201 Created and 204 No Content serve distinct purposes:

201 Created signifies that the request has succeeded and a new resource was created as a result of it.
204 No Content indicates that the server successfully processed the request but is not returning any content in the response body.

This distinction helps clients understand what kind of action they should take next or how to interpret the data returned by the server. Let's break this down step-by-step from first principles and explore some common scenarios.

**Code:**

```python
# Example HTTP responses with status codes

# 200 OK response
response_200 = {
    "status": 200,
    "message": "Request successful",
    # Additional data or resource might be included here
}

# 201 Created response
response_201 = {
    "status": 201,
    "message": "Resource created successfully",
    "location": "/resources/123"  # URL of the newly created resource
}

# 204 No Content response
response_204 = {
    "status": 204,
    "message": "No content to return"
}
```

**Example input / output:**

```
Input (Example Request):
HTTP
Copy
POST /users HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "username": "john_doe",
  "email": "john@example.com"
}
Output (200 OK Response):
HTTP
Copy
HTTP/1.1 200 OK
Content-Type: application/json

{
  "status": 200,
  "message": "User created successfully",
  "user_id": 456
}
Output (201 Created Response):
HTTP
Copy
HTTP/1.1 201 Created
Location: /users/789

{
  "status": 201,
  "message": "User created successfully",
  "user_id": 789
}
Output (204 No Content Response):
HTTP
Copy
HTTP/1.1 204 No Content
Content-Type: application/json

# The response body is empty, with only the status line and headers.
```

