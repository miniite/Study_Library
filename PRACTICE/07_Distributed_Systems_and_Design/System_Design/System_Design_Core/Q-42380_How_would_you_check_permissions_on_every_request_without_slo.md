---
type: "practice"
difficulty: "Deep"
---

# How would you check permissions on every request without slowing down the API?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Read-through and Write-behind Caching](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that permissions are checked on every request without significantly slowing down the API, we need to design a robust and efficient permission system. At a high level, the goal is to minimize overhead while ensuring that all requests are properly authenticated and authorized. We'll break this down step-by-step using first principles.

The key components in our solution include:

Authentication: Verifying the user's identity.
Authorization: Checking if the user has the required permissions.
Caching Mechanisms: To reduce redundant checks on subsequent requests from the same user or with a similar context.

In this response, I'll outline the steps and design decisions to achieve efficient permission checks without slowing down the API.

**Comparison:**

```
Step	Action
1	Extract token from headers.
2	Decode the JWT and extract user_id.
3	Check if the user has required permissions.
4	Return appropriate response based on success or failure of authentication/authorization.
```

**Code:**

```python
# Example code for an authentication middleware in Python using Flask

from flask import request, jsonify
import jwt  # Assuming we use JWT tokens for simplicity

def authenticate_and_authorize(f):
    def wrapper(*args, **kwargs):
        auth_header = request.headers.get('Authorization')
        if not auth_header:
            return jsonify({"error": "Unauthorized"}), 401

        token = auth_header.split(" ")[1]
        try:
            # Decode the JWT token
            payload = jwt.decode(token, 'secret_key', algorithms=['HS256'])
            user_id = payload['user_id']
            request.user_id = user_id

            # Authorize based on roles or permissions
            if not has_permission(user_id):
                return jsonify({"error": "Unauthorized"}), 401
        except jwt.ExpiredSignatureError:
            return jsonify({"error": "Token expired"}), 401
        except jwt.InvalidTokenError:
            return jsonify({"error": "Invalid token"}), 401

        return f(*args, **kwargs)
    return wrapper
```

**Example input / output:**

```

Given the following input:

Request Headers:

CODE
Copy
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoxMjMsInByb2ZpbGVfaWQiOlsncmVkaXJlY3QnLCcicHdkZWdpbiJdLCJleHAiOjE2NTk5OTcyNzR9.7r-108qC7u8vD6Ux4eMhIw4FVZoPfXtKl3nYsG6H3aTg

Expected Output:

PYTHON
Copy
{
  "user_id": 123,
  "permissions": ["read", "write"]
}
```

**Data:**

```json
{
  "user_id": 123,
  "permissions": ["read", "write"]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. Request Received
7. Is Token Present?
8. Decode Token
9. User ID Valid?
10. Return 401 Unauthorized
11. Has Required Permissions?
12. Proceed with Request

