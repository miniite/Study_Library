---
type: "practice"
difficulty: "Basic"
---

# What is the difference between authentication and authorization in the context of an API?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ API Keys vs Tokens](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/API_Keys_vs_Tokens.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In the context of a REST API, understanding the differences between authentication and authorization is crucial for ensuring security and protecting your application's resources. Authentication involves verifying the identity of the user or client attempting to access your API, while authorization focuses on determining what actions that authenticated entity can perform once they are identified.

TL;DR: Authentication is about "who you are," and authorization is about "what you can do."

**Comparison:**

```
User	Role	Allowed Actions
admin	admin	read, write
user1	user	read
```

**Code:**

```python
# Example code snippets for authentication and authorization concepts

def authenticate_user(username, password):
    # Simple example of user authentication using hardcoded credentials
    if username == 'admin' and password == 'password':
        return True
    return False

def authorize_request(user_id, resource_type, action):
    # Example authorization logic where only authenticated users can read/write to their own resources
    if user_id:
        allowed_actions = {'read': ['user'], 'write': ['admin']}
        if user_id in allowed_actions[action] or (action == 'read' and user_id == resource_type):
            return True
    return False
```

**Example input / output:**

```text
# Input and output for the example code

# Authentication input
authenticated = authenticate_user('admin', 'password')  # Output: True

# Authorization input
authorized = authorize_request(2, "typeA", "write")   # Output: False (user1 cannot write to typeA)
authorized_admin = authorize_request(1, "typeA", "write")  # Output: True (admin can write to typeA)
```

**Data:**

```json
{
  "users": [
    { "id": 1, "username": "admin", "role": "admin" },
    { "id": 2, "username": "user1", "role": "user" }
  ],
  "resources": {
    "typeA": ["read", "write"],
    "typeB": ["read"]
  }
}
```

**Flow:**

1. Yes
2. No
3. No
4. User Authenticates
5. Is User Authenticated?
6. Retrieve User Role
7. Is User Authorized to PerformAction?
8. Deny Access
9. Allow Access

