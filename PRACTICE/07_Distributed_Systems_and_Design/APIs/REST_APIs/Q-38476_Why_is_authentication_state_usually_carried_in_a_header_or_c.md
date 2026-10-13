---
type: "practice"
difficulty: "Intermediate"
---

# Why is authentication state usually carried in a header or cookie rather than the request body in a stateless REST API?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ API Keys vs Tokens](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/API_Keys_vs_Tokens.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When building a stateless REST API, one of the key principles is to ensure that each request contains all necessary information so that the server can process it independently. Carrying authentication state in headers or cookies aligns with this principle by keeping sensitive data out of the request body and making sure requests are self-contained and idempotent.

#### Triaging the Question

The core issue here is understanding why we typically prefer to keep the authentication token in a header or cookie rather than within the request body. The interviewer might be probing your knowledge of REST principles, security best practices, and how statelessness impacts API design.

#### Assumptions
Statelessness: The server does not retain any session information between requests.
Security Concerns: Sensitive data like authentication tokens need to be handled carefully.
HTTP Semantics: Understanding the semantics of HTTP headers and bodies is crucial.

#### First Principles Solution

Self-Containment Principle: In a stateless API, each request should contain all necessary information for processing. This means that every request should include its own authentication token without relying on session data stored elsewhere.

Security: Sensitive data such as tokens are better kept out of the body to prevent accidental exposure or tampering.

HTTP Semantics: Headers in HTTP requests can be used to carry metadata, including authentication tokens, while the body is reserved for payloads like JSON objects.

Idempotency and Cacheability: Headers allow for additional metadata that can help with idempotent operations and caching strategies without revealing sensitive information.

Step-by-Step Solution

Identify the Role of Authentication Tokens:

An authentication token (e.g., JWT) is needed to verify the user's identity and permissions.
This token should be sent with each request to maintain statelessness.

Headers vs. Body:

Headers: Typically used for metadata, including authentication tokens. They are a good place because they can be set by the client (browser) without tampering easily.
Body: Reserved for data payloads. Placing sensitive information here could expose it to vulnerabilities.

Cookie Mechanism:

Cookies can also be used as part of the HTTP response headers, allowing them to persist across requests and automatically included in subsequent requests via the Cookie header.
This mechanism is useful when you want to avoid sending tokens with every request, but still need statelessness.

#### Closing Summary

In a stateless REST API, keeping authentication tokens in headers or cookies ensures that each request contains all necessary information independently. Headers and cookies provide security benefits by keeping sensitive data out of the body and allow for better control over HTTP semantics such as idempotency and caching.

#### Common Mistakes/Follow-ups
Misplacing Tokens: Not placing tokens correctly (e.g., in the body instead of headers).
Security Vulnerabilities: Failing to implement secure handling practices for tokens.

#### Performance Considerations
Overusing cookies can lead to increased latency, especially with large amounts of data.
Self-CHECK

Correctness + completeness confirmed.

**Comparison:**

| Header      | Value                            |
|-------------|----------------------------------|
| Authorization | Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6U4GmsIIvZbOTk |

**Code:**

```plaintext
// Example code snippet to demonstrate setting a JWT in an HTTP header using Node.js and Express
const jwt = require('jsonwebtoken');
app.use((req, res, next) => {
  const token = req.header('Authorization');
  if (token) {
    jwt.verify(token, process.env.SECRET_KEY, (err, user) => {
      if (err) return res.sendStatus(403);
      req.user = user;
      next();
    });
  } else {
    // If no token is present, reject the request
    return res.status(401).send('Access Denied');
  }
});
```

**Example input / output:**

```text
Request:
GET /api/user-profile HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6U4GmsIIvZbOTk

Response:
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 12345,
  "username": "john_doe",
  "email": "john.doe@example.com"
}
```

**Data:**

```plaintext
{
  "requestHeader": {
    "Authorization": "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6U4GmsIIvZbOTk"
  }
}
```

**Flow:**

1. Yes
2. No
3. User Sends Request
4. Is Token in Header?
5. Request Includes Token
6. Server Verifies Token
7. Server Processes Request
8. Response Sent
9. Server Rejects Request

