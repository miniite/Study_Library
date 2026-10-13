---
type: "practice"
difficulty: "Deep"
---

# What are the risks of exposing raw database column names or internal IDs directly in your REST API responses?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Keeping Secrets out of URLs](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Keeping_Secrets_out_of_URLs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a REST API, exposing raw database column names or internal IDs can introduce several significant security and operational risks. This practice is often seen as a red flag by experienced engineers due to its potential implications on data integrity, maintainability, and security. In this answer, we'll explore the key risks involved in such an approach and discuss best practices for mitigating them.

**Code:**

```python
# Example of raw database column name exposure in API response
def get_user_profile(user_id):
    user = db.get_user(user_id)
    return {
        "userId": user.id,  # Raw internal ID
        "name": user.name,
        "email": user.email
    }
```

**Example input / output:**

```

Given the input get_user_profile(12345):

JSON
Copy
{
    "userProfile": {
        "userId": 12345,
        "name": "Alice Smith",
        "email": "alice@example.com"
    }
}
```

**Data:**

```json
{
    "userProfile": {
        "userId": 12345,      // Raw database ID
        "name": "Alice Smith",
        "email": "alice@example.com"
    }
}
```

**Flow:**

1. Yes
2. No
3. API Call
4. Is DB Column Exposed?
5. Security Risk
6. API Design Best Practices
7. Data Exposure, Insecure Logs,etc.
8. Secure API Design Principles

