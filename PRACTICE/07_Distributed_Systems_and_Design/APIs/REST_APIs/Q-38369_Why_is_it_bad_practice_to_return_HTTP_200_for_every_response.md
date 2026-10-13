---
type: "practice"
difficulty: "Deep"
---

# Why is it bad practice to return HTTP 200 for every response and put the real status inside the body?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ HTTP Status Codes](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/HTTP_Status_Codes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing REST APIs, returning an HTTP 200 (OK) status code for every response and embedding the real status information in the body can lead to several issues that undermine the clarity and reliability of API communication. This approach violates fundamental principles of HTTP status codes and can complicate error handling on the client side.

#### Triaging the Question

The core issue at hand is understanding why it’s generally considered poor practice to use a 200 OK status code for all responses and include the real status information within the response body. The interviewer likely probes your knowledge of HTTP status codes, REST API design principles, error handling mechanisms, and the importance of clear communication between clients and servers.

#### First-Principles Solution

HTTP status codes are a crucial part of web communication protocols. They provide essential information about the outcome of an HTTP request to both the client and the server. Using 200 OK for every response can obscure the actual nature of the operation, making it difficult for clients to distinguish between successful operations and errors. Here’s why:

Clarity and Consistency: HTTP status codes like 404 (Not Found), 403 (Forbidden), or 500 (Internal Server Error) provide clear and consistent indications about what went wrong, which is essential for both human readability and automated systems.
Error Handling: Embedding error details in the response body can lead to complex logic on the client side to parse these errors, increasing the potential for bugs. HTTP status codes allow clients to quickly determine if a request was successful or not without parsing the payload.
Caching: Using 200 OK consistently means that caching mechanisms can be relied upon more easily because they know when a response is successfully served.
Step-by-Step Solution

Understanding the Status Codes:

200 (OK): The request was successful. Use this only if you want to return data or content.
4xx Series: Client errors (e.g., 400 Bad Request, 403 Forbidden).
5xx Series: Server errors (e.g., 500 Internal Server Error).

Embedding Real Status in the Body:

Embedding real status information in the response body adds unnecessary complexity.
For example, returning a JSON object like { "status": "ERROR", "message": "Not Found" } with a 200 OK can be confusing and violate HTTP semantics.

Client-Side Implications:

Clients expect a 200 status code to mean success.
Handling mixed results (200 for success, non-200 for errors) requires additional logic in the client, increasing complexity and risk of bugs.

Server-Side Best Practices:

Use appropriate HTTP status codes based on the outcome of the request.
Return meaningful data or content with 200 OK only if applicable.
Ensure error messages are consistent and clear to aid debugging.

#### Closing Summary

In summary, using a 200 OK status code for every response and embedding real status information in the body can lead to confusion, complicate client-side logic, and violate REST principles. Instead, it’s best practice to use appropriate HTTP status codes that clearly indicate the outcome of each request.

#### Common Mistakes
and Follow-Ups
Mistake: Using 200 OK for all responses without considering the actual success or failure.

#### Follow-Up
Discuss edge cases where returning a non-200 status might be necessary (e.g., rate limiting, temporary failures) while still adhering to REST principles.

**Comparison:**

| HTTP Status Code | Description                   | Example Response Body                          |
|------------------|-------------------------------|------------------------------------------------|
| 200 OK           | Request was successful        | {"data": [...]}                                 |
| 404 NOT FOUND    | Resource not found            | {"status": "ERROR", "message": "Resource not found"} |
| 500 INTERNAL SERVER ERROR | Server error     | {"status": "ERROR", "message": "Internal server error"} |

**Code:**

```python
# Example of using appropriate HTTP status codes in Python Flask

@app.route('/api/data', methods=['GET'])
def get_data():
    try:
        data = fetch_data()
        return jsonify(data), 200
    except ResourceNotFound as e:
        return jsonify({"status": "ERROR", "message": str(e)}), 404
    except InternalServerError as e:
        return jsonify({"status": "ERROR", "message": str(e)}), 500
```

**Example input / output:**

```text
Request:
GET /api/data

Response (Success):
HTTP/1.1 200 OK
Content-Type: application/json

{
  "data": [
    {
      "id": 1,
      "name": "Item One"
    },
    {
      "id": 2,
      "name": "Item Two"
    }
  ]
}

Response (Resource Not Found):
HTTP/1.1 404 NOT FOUND
Content-Type: application/json

{
  "status": "ERROR",
  "message": "Resource not found"
}
```

**Data:**

```json
{
  "data": [
    {
      "id": 1,
      "name": "Item One"
    },
    {
      "id": 2,
      "name": "Item Two"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Client Sends Request
8. Server Processes Request
9. Request Successful?
10. 200 OK, Return Data
11. Error Type?
12. Resource Not Found?
13. 404 NOT FOUND, ErrorMessage
14. Server Error?
15. 500 INTERNAL SERVER ERROR,Error Message

