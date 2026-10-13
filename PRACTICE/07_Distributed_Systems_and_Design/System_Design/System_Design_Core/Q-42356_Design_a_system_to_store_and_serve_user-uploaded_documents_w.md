---
type: "practice"
difficulty: "Intermediate"
---

# Design a system to store and serve user-uploaded documents with access control.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Chat History Storage](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Chat_History_Storage.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a system that stores and serves user-uploaded documents with robust access control, we need to focus on several key aspects: storage, security, scalability, and performance. The goal is to ensure efficient document management while maintaining strict access controls.

Firstly, let’s break down what the question is probing:

Storage Management: How do we handle storing large volumes of documents efficiently?
Access Control: What mechanisms will be used to enforce user permissions on these documents?
Scalability and Performance: How can we ensure that the system scales well with increasing loads, both in terms of number of users and document storage size?

#### Assumptions

Users have unique identities (e.g., email or username).
Documents are version-controlled.
Access control is role-based or attribute-based.

Now, let's delve into the solution step-by-step:

Document Storage:

Use a distributed file system like AWS S3 or Google Cloud Storage for storing documents. This ensures high availability and scalability.
Implement an object storage service to reduce costs and improve performance by distributing data across multiple servers.

Metadata Management:

Store metadata about each document in a database (e.g., PostgreSQL, MongoDB) that includes details like owner, creation date, last modified, and version history.
This helps in quick retrieval and updating of documents.

Access Control Layer:

Implement role-based access control (RBAC) or attribute-based access control (ABAC). For example, users can be assigned roles such as “viewer,” “editor,” or “admin” based on their permissions.
Use OAuth 2.0 or OpenID Connect for authentication to ensure that only authorized users can access the documents.

API Layer:

Develop a RESTful API layer using frameworks like Node.js, Django, or Ruby on Rails to handle CRUD operations and access control checks.
Ensure that each request includes necessary headers (e.g., JWT token) for authentication and authorization before proceeding with document access requests.

Caching Mechanism:

Implement caching at various levels (e.g., Redis for frequently accessed documents, CDN for static content) to reduce latency and improve performance.
Use a reverse proxy like NGINX or Envoy for managing traffic distribution and caching.

Monitoring and Logging:

Set up monitoring tools like Prometheus and Grafana to track system performance metrics such as document load times, storage usage, and error rates.
Implement logging mechanisms (e.g., ELK stack) to capture access patterns and security events for auditing purposes.

Security Measures:

Encrypt documents both at rest and in transit using industry-standard protocols like TLS.
Regularly perform security audits and vulnerability assessments to ensure the system remains secure against evolving threats.

#### Closing Summary
The design involves a combination of distributed storage, robust metadata management, an advanced access control layer, efficient API handling, caching mechanisms, monitoring tools, and stringent security measures. This ensures that documents are stored securely, accessed efficiently, and managed effectively while meeting the needs for scalability and performance.

#### Common Mistakes/Follow-Ups

Failing to consider edge cases such as document versioning or concurrent access.
Not implementing proper logging and monitoring, leading to difficulty in troubleshooting issues.
Overlooking security best practices like encryption and regular audits.

**Comparison:**

| Document ID | Owner       | Created Date         |
|-------------|-------------|---------------------|
| 123         | user1@example.com | 2023-10-01T14:00:00Z |

**Code:**

```javascript
// Example of a simple authentication middleware for Node.js using JWT
const jwt = require('jsonwebtoken');

function authenticateToken(req, res, next) {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];

  if (token == null) return res.sendStatus(401);

  jwt.verify(token, process.env.ACCESS_TOKEN_SECRET, (err, user) => {
    if (err) return res.sendStatus(403);
    req.user = user;
    next();
  });
}
```

**Example input / output:**

```text
Input:
GET /documents/123?access_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

Output:
HTTP 200 OK
{
  "document": {
    "id": 123,
    "title": "Sample Document",
    "content": "This is the content of the document.",
    "metadata": {
      "owner": "user1@example.com",
      "createdDate": "2023-10-01T14:00:00Z",
      "versions": [
        {"version": 1, "modifiedBy": "admin@example.com"}
      ]
    }
  }
}
```

**Data:**

```json
{
  "metadata": [
    {"id": 1, "owner": "user1@example.com", "createdDate": "2023-10-01T14:00:00Z", "versions": [{"version": 1, "modifiedBy": "admin@example.com"}]}
  ]
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. No
5. User Request
6. Is Token Valid?
7. Authenticate User
8. Check Document AccessRights
9. Fetch and Return Document
10. Return 403 Forbidden
11. Return 401 Unauthorized

