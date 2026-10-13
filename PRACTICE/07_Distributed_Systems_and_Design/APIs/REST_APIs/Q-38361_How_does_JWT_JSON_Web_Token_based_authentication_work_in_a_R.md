---
type: "practice"
difficulty: "Intermediate"
---

# How does JWT (JSON Web Token) based authentication work in a REST API?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ API Keys vs Tokens](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/API_Keys_vs_Tokens.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

JWT-based authentication is a common method used to secure REST APIs by allowing applications to securely transmit tokens over public networks. The core idea behind JWT is that it’s a compact, URL-safe means of representing claims to be transferred between two parties. In this context, the key components are the token generation on the server-side and verification on the client-side.

The question is asking for an explanation of how JWT-based authentication works in a REST API setup. The interviewer might be probing your understanding of the flow from generating tokens to their validation during API calls. Key assumptions include: we’re using HTTPS, which provides secure communication; there’s a server that generates and verifies tokens; and clients make authenticated requests.

Here’s the step-by-step process:

Token Generation: When a user logs in successfully, the server generates a JWT token containing claims like user ID, expiration time, etc.
Token Storage: The client stores this token, typically in local storage or cookies.
API Calls: On each subsequent request to a protected API endpoint, the client sends the JWT as part of the Authorization header (commonly prefixed with "Bearer").
Token Validation: The server receives the request and validates the JWT using a secret key or public key before processing the request.

Let's go through this in detail:

Token Generation

When a user logs in, you have:

A username/password pair.
A backend service that checks these credentials against a database (or OAuth provider).

Upon successful validation, the server generates a JWT token. This token is encoded using Base64 and contains claims like:

JSON
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022,
  "exp": 1516240022
}
sub: Subject (user ID).
name: User’s name.
iat and exp: Issued at and expiration time, respectively.
Token Storage

The token is stored in the client-side session using local storage or cookies. For example:

JAVASCRIPT
localStorage.setItem('token', 'your.jwt.token');
API Calls

On every request to a protected resource (e.g., /api/profile), the client includes the JWT in the Authorization header:

HTTP
GET /api/profile HTTP/1.1
Host: api.example.com
Authorization: Bearer your.jwt.token
Token Validation

The server receives the request, extracts the token from the Authorization header, and validates it using a secret key (or public key if JWT is signed with RSA or ECDSA). If valid, the server processes the request; otherwise, it returns an error.

This process ensures that only authenticated users can access protected resources without exposing sensitive data over HTTP.

#### Closing Summary

JWT-based authentication simplifies stateless session management and reduces server-side storage requirements. However, JWTs should be treated with caution due to potential security risks like token replay attacks or brute-forcing of short-lived tokens.

#### Common mistakes
include:

Using weak keys or no validation checks.
Storing sensitive information in the token (use claims minimally).
Not rotating tokens regularly.

**Comparison:**

```
Step	Action
1	Generate JWT token on server-side.
2	Store the token in client storage (localstorage/Cookie).
3	Include the token in Authorization header for API requests.
4	Validate the token on the server using a secret key.
```

**Code:**

```

// Example JWT generation using Node.js and JSONWebToken library const jwt = require('jsonwebtoken'); const secretKey = 'your-secret-key';

function generateToken(user) { const payload = { sub: user.id, name: user.name }; return jwt.sign(payload, secretKey, { expiresIn: '1h' }); }

CODE
Copy

```javascript
// Example JWT validation
async function validateToken(token) {
  try {
    const decoded = await new Promise((resolve, reject) => {
      jwt.verify(token, secretKey, (err, decoded) => {
        if (err) reject(err);
        resolve(decoded);
      });
    });

    return { user: decoded.sub, name: decoded.name };
  } catch (error) {
    throw error;
  }
}
```

**Example input / output:**

```text
// Example request for a protected resource
GET /api/profile HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

**Data:**

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"
}
```

**Flow:**

1. User logs in
2. Server generates JWT
3. Client stores token
4. Client makes API request
5. API receives and validates JWT
6. Server processes request ifvalid
7. Return response or error

