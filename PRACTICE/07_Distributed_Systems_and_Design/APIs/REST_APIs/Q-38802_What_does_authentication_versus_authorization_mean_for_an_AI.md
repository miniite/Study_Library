---
type: "practice"
difficulty: "Basic"
---

# What does authentication versus authorization mean for an AI chatbot API, for example verifying who a user is versus which documents they may query?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ API Keys vs Tokens](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/API_Keys_vs_Tokens.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Authentication and authorization are two critical security concepts that ensure both identity verification and permission management in APIs like an AI chatbot. Authentication verifies the identity of a user, while authorization determines what actions or resources that authenticated user can access.

In the context of an AI chatbot API, authentication might involve checking if the user is who they claim to be, possibly through methods like OAuth, JWT tokens, or basic HTTP authentication. Authorization would then determine which documents (or data) the verified user is allowed to query based on their role or permissions.

**Code:**

```python
# Example of a simple HTTP Basic Authentication in Python
import base64

def authenticate_user(username, password):
    auth_string = f"{username}:{password}"
    encoded_auth = base64.b64encode(auth_string.encode()).decode()
    return {"Authorization": f"Basic {encoded_auth}"}

auth_header = authenticate_user("user123", "pass456")
```

**Example input / output:**

```
HTTP
Copy
GET /documents HTTP/1.1
Host: example.com
Authorization: Basic dXNlcjEyMzozNDY=

This request would trigger authentication to verify the user's identity before checking their permissions.
```

**Data:**

```text
- **User Credentials**: `username: user123, password: pass456`
- **Encoded Auth Header**: `Basic dXNlcjEyMzozNDY=`
```

