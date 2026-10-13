---
type: "practice"
difficulty: "Expert"
---

# A model trained on customer-support transcripts is now shared across business units. How do you enforce purpose limitation so the model isn't used beyond consented scope?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Privacy by Default](<../../../../11_AI_and_ML/Responsible_AI/Privacy_and_PII/Privacy_by_Default.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enforce purpose limitation for a model trained on customer support transcripts and ensure it's not used beyond its intended scope, we need to implement a robust data governance strategy that includes clear policies, technical controls, and monitoring mechanisms. This approach ensures compliance with legal and ethical standards while maintaining the utility of the model.

The key steps involve defining the purpose of the model, restricting access based on roles and responsibilities, implementing encryption and anonymization techniques, setting up audit trails, and ensuring ongoing compliance checks. These measures will help maintain data privacy and prevent misuse.

**Comparison:**

```
Role	Read	Write	Audit	Administer
Analyst	X
Manager	X	X	X
Admin		X	X	X
```

**Code:**

```python
# Example Python code to demonstrate role-based access control (RBAC)
class AccessControl:
    def __init__(self):
        self.permissions = {
            "analyst": ["read", "analyze"],
            "manager": ["read", "write", "audit"],
            "admin": ["read", "write", "administer"]
        }

    def check_permission(self, user_role, action):
        if user_role in self.permissions and action in self.permissions[user_role]:
            return True
        else:
            return False

# Example usage
ac = AccessControl()
print(ac.check_permission("manager", "read"))  # True
print(ac.check_permission("analyst", "administer"))  # False
```

**Example input / output:**

```

Input:

User role: manager
Action to check: write

Output: True

CODE
Copy

Input:
- User role: analyst
- Action to check: administer

Output:
False
```

**Data:**

```json
{
    "data": {
        "user_role": ["analyst", "manager", "admin"],
        "permissions": [
            {"action": "read"},
            {"action": "write", "audit": true},
            {"action": "administer"}
        ]
    }
}
```

**Flow:**

1. Yes
2. No
3. User logs in
4. Is user role allowed to performaction?
5. Grant access
6. Deny access and log attempt
7. Audit successful accessattempt
8. Audit failed access attempt

