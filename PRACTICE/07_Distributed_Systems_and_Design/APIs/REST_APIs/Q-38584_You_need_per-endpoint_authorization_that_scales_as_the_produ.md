---
type: "practice"
difficulty: "Expert"
---

# You need per-endpoint authorization that scales as the product grows. Compare RBAC, ABAC, and a policy engine like OPA for a REST API, and how you'd keep authorization decisions testable and centralized.

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Authorization Design](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Authorization_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address per-endpoint authorization in a growing backend system using RBAC (Role-Based Access Control), ABAC (Attribute-Based Access Control), or an OPA policy engine, we need to consider several factors including scalability, flexibility, testability, and maintainability. Let's break down each approach and then discuss how to ensure testable and centralized authorization.

RBAC is a straightforward model that categorizes users into roles based on their job functions. Each role has permissions on specific resources or actions. ABAC, on the other hand, evaluates access decisions based on attributes such as user data, context, time of day, etc., making it more flexible but potentially complex to implement and maintain. A policy engine like OPA allows for dynamic policies that can be updated without code changes, providing a balance between flexibility and manageability.

When choosing among these approaches, we need to consider the following:

Scalability: How easily can we add new roles, permissions, or attributes?
Flexibility: Can we adjust access rules based on context (e.g., time of day, geographical location)?
Testability: How easy is it to write and verify authorization logic?
Centralization: Where are decisions made—close to the resource or in a centralized policy engine?

#### Triaging the Question

The interviewer is probing our understanding of different authorization models, their trade-offs, and how we ensure that authorization decisions are both testable and centrally managed. We need to make assumptions about the system's size, complexity, and expected growth.

#### First-Principles Solution
RBAC

RBAC simplifies permission management by assigning roles to users and then defining permissions for each role. This is straightforward but can become cumbersome as the number of roles grows or when fine-grained controls are required.

#### Example

PLAINTEXT
User Role:
  - Admin
  - User
  - Guest

Endpoint Permissions:
- /admin-dashboard -> [Admin]
- /user-profile -> [Admin, User]
- /public-content -> [Guest, User, Admin]
ABAC

ABAC is more flexible as it allows access decisions to be based on attributes. This can include user roles, but also other contextual information like time, location, or device type.

#### Example

PLAINTEXT
Policy:
  - allow if (user.role == "Admin" and current.time < "09:00") -> /admin-dashboard
  - allow if (user.role == "User" and location.city == "New York") -> /user-profile
OPA Policy Engine

OPA allows for complex policy evaluation based on JSON data. It can be used to implement both RBAC and ABAC, providing a flexible yet centralized way to manage authorization.

#### Example

JSON
# policies/opahelpers.json
{
  "rules": {
    "admin-only-dashboards": "input.user.role == 'Admin'"
  }
}

# opa-policy.json
package policies
default allow = false

allow {
  input.method == "GET"
  data.policies.admin-only-dashboards()
}
Keeping Authorization Decisions Testable and Centralized

To ensure that authorization decisions are testable and centralized:

Use a Policy Engine: OPA or similar tools provide a single place to manage policies, making it easier to maintain and update them.
Test Policies Independently: Write unit tests for each policy rule. For example, in OPA, you can use the opa eval command to run test scenarios.
Separate Concerns: Keep authorization logic separate from business logic by defining clear interfaces between services that need access control.

#### Closing Summary
and Common Mistakes/Follow-Ups

RBAC is simple but inflexible for complex policies; ABAC provides flexibility at the cost of complexity; OPA offers a balance. For scalability, consider using OPA or a similar policy engine to manage dynamic permissions efficiently. Ensure testability by writing comprehensive tests for each policy rule and keeping authorization logic centralized.

#### Common mistakes
include:

Overcomplicating RBAC with too many roles.
Ignoring context in ABAC, leading to overly complex policies.
Not testing authorization rules thoroughly, especially in dynamic environments like OPA.

#### Follow-ups
could involve discussing specific implementation details or trade-offs between different approaches based on the system's requirements.

**Comparison:**

| Policy Type | Scalability | Flexibility | Testability | Centralization |
|-------------|-------------|-------------|-------------|---------------|
| RBAC        | Medium      | Low         | Good        | Good          |
| ABAC        | High        | High        | Good        | Poor          |
| OPA         | High        | High        | Excellent   | Excellent     |

**Code:**

```
MERMAID
Copy
graph TD;
    A[RBAC] --> B{Is it scalable?};
    B -- Yes --> C[Use RBAC for simple scenarios];
    B -- No --> D[Consider ABAC or OPA for more complex needs];

    E[ABAC] --> F{Is it flexible enough?};
    F -- Yes --> G[Use ABAC when context is important];
    F -- No --> H[Consider using a policy engine like OPA for dynamic decisions];

    I[OPA Policy Engine] --> J{Can you manage policies centrally?};
    J -- Yes --> K[Use OPA for centralized and flexible authorization];
    J -- No --> L[Reconsider your approach if centralization is critical];
```

**Example input / output:**

```text
Input:
- User ID: 1 (Admin)
- Endpoint: /admin-dashboard

Output:
- Allow Access

Input:
- User ID: 2 (User)
- Endpoint: /public-content

Output:
- Allow Access

Input:
- User ID: 1 (Admin)
- Endpoint: /user-profile

Output:
- Deny Access
```

**Data:**

```json
{
  "users": [
    {"id": 1, "username": "admin", "role": "Admin"},
    {"id": 2, "username": "user1", "role": "User"}
  ],
  "permissions": {
    "/admin-dashboard": ["Admin"],
    "/user-profile": ["Admin", "User"]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. RBAC
8. Is it scalable?
9. Use RBAC for simple scenarios
10. Consider ABAC or OPA formore complex needs
11. ABAC
12. Is it flexible enough?
13. Use ABAC when context isimportant
14. Consider using a policy enginelike OPA for dynamic decisions
15. OPA Policy Engine
16. Can you manage policiescentrally?
17. Use OPA for centralized andflexible authorization
18. Reconsider your approach ifcentralization is critical

