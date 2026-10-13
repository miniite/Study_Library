---
type: "practice"
difficulty: "Deep"
---

# What is a CORS preflight request, and which requests trigger one?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ CORS](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/CORS.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A Cross-Origin Resource Sharing (CORS) preflight request is a special kind of HTTP request that checks whether a browser should be allowed to make an actual cross-origin request. This mechanism ensures security by confirming with the server that it’s safe to send sensitive data from one domain to another.

The question is asking for two main things: what CORS preflight requests are and which types of requests trigger them. The interviewer might also want to assess your understanding of how browsers handle CORS, security implications, and potential use cases or design considerations.

We assume the context here includes a REST API that involves cross-origin requests from different domains. CORS is necessary when an HTTP request's origin (domain) does not match the domain of the server receiving the request.

**Comparison:**

```
CORS Request Type	Preflight Required?
GET, HEAD	No
POST, PUT, DELETE	Yes
PATCH	Yes
```

**Code:**

```
MERMAID
Copy
sequenceDiagram
    participant Browser
    participant Server
    Browser ->> Server: OPTIONS /api/resource?origin=http://example.com
    Server -->> Browser: 200 OK, Access-Control-Request-Headers: Content-Type
    Browser ->> Server: POST /api/resource HTTP/1.1
    Server -->> Browser: 200 OK
```

**Example input / output:**

```text
Browser to Server:
OPTIONS /api/resource?origin=http://example.com

Server response:
HTTP/1.1 200 OK
Access-Control-Request-Headers: Content-Type

Browser sends actual request:
POST /api/resource HTTP/1.1
Content-Type: application/json

Server response:
HTTP/1.1 200 OK
```

**Data:**

```json
{
  "request": {
    "method": "OPTIONS",
    "url": "/api/resource",
    "headers": {
      "Origin": "http://example.com"
    }
  },
  "response": {
    "status": 200,
    "headers": {
      "Access-Control-Request-Headers": "Content-Type"
    }
  }
}
```

**Flow:**

1. Policy allows request
2. Policy disallows request
3. Policy allows request
4. Policy disallows request
5. Browser sends request
6. Request origin doesn't matchserver
7. Server checks CORS policy
8. Browser allowed, proceed withactual request
9. Server responds with 403Forbidden or 401Unauthorized
10. CORS preflight request sent(OPTIONS)
11. Server checks CORS policy
12. Browser allowed, proceed withactual request
13. Server responds withappropriate error

