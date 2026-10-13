---
type: "practice"
difficulty: "Expert"
---

# Design an authorization model (RBAC vs ABAC vs ReBAC) for a doc-sharing app where users grant per-document access. How do you check permission on every request in single-digit ms at scale?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Org-wide Lineage and PII Tracking](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Org-wide_Lineage_and_PII_Tracking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To design an authorization model for a document-sharing app that ensures per-document access and can handle millions of requests per second, we should consider using a hybrid approach combining Role-Based Access Control (RBAC) with Attribute-Based Access Control (ABAC). This ReBAC (Role-Enhanced ABAC) model will leverage roles for initial permissions while dynamically evaluating attributes during each request to minimize latency. The system will be optimized by caching and indexing strategies, ensuring that permission checks are performed in single-digit milliseconds.

#### Triage

The core of this question is understanding how to efficiently implement an authorization mechanism that can handle large-scale document-sharing scenarios with fine-grained access control. The interviewer is probing your knowledge on:

Different types of access control models (RBAC, ABAC).
Hybrid approaches like ReBAC.
Techniques for reducing latency and scaling the system.
Caching mechanisms and indexing strategies.

#### First Principles Solution
Step 1: Understand RBAC vs ABAC

Role-Based Access Control (RBAC):

Simple and scalable.
Permissions are based on roles, making it easier to manage.
Challenges with dynamic permissions.

Attribute-Based Access Control (ABAC):

Highly flexible and granular.
Policies can be complex and context-dependent.
Requires more processing power for evaluating policies dynamically.
Step 2: Introduce ReBAC

ReBAC combines the best of both worlds:

Role Hierarchy: Use RBAC to define roles and their associated permissions.
Dynamic Evaluation: For fine-grained control, leverage ABAC rules that are applied based on attributes like user properties, document metadata, time, etc.
Step 3: Design the System

Role Management:

Define roles (e.g., admin, editor, viewer) and their associated permissions.
Store role definitions in a database with proper normalization to avoid redundancy.

User-Role Assignment:

Map users to roles.
Cache this mapping for quick lookups.

Document Metadata:

Attach attributes (e.g., sensitivity level, access level) to documents.
Cache these attributes in a document metadata store.

Policy Definitions:

Define ABAC policies that define the conditions under which permissions are granted or denied based on user roles and document attributes.

Permission Checking:

When a request comes in, first check if the user has any specific role-based permissions.
If not, use ABAC to evaluate additional attributes (e.g., time of day, user group, etc.) for dynamic permission checks.

Caching and Indexing:

Cache role mappings and document metadata to minimize database lookups.
Use indexes on commonly queried fields in the database.

Optimization Techniques:

Implement an in-memory cache for frequently accessed roles and documents.
Use distributed caching solutions like Redis or Memcached to handle high concurrency.
Step 4: Performance Considerations
Aim for single-digit millisecond response times by minimizing database queries and leveraging caching.
Ensure that the permission check logic is optimized, potentially using precomputed results stored in a cache.

#### Closing Summary

By combining RBAC with ABAC in a ReBAC model and implementing efficient caching strategies, we can achieve scalable and fine-grained access control for a document-sharing application. This hybrid approach balances simplicity and flexibility while ensuring that permission checks are performed efficiently at scale.

#### Common Mistakes/Follow-Ups
Failing to consider the complexity of dynamic ABAC policies.
Overlooking role hierarchy management and caching strategies.
Not addressing potential security risks associated with complex policy evaluation.

**Comparison:**

```
User ID	Roles
1	admin, viewer
2
Document ID	Attributes
1	sensitivityLevel: public
2	sensitivityLevel: private
```

**Code:**

```python
# Example of Role-Based Permission Check
def check_role_permission(user_roles, required_role):
    return any(required_role in user_role for user_role in user_roles)

# Example of ABAC Policy Evaluation (simplified)
from abc import abstractmethod

class ABACPolicies:
    @abstractmethod
    def evaluate_policy(self, attributes):
        pass

class TimeBasedPolicy(ABACPolicies):
    def evaluate_policy(self, attributes):
        current_time = datetime.now()
        return 9 <= current_time.hour < 18  # Work hours only

# Usage Example
user_roles = ['admin', 'viewer']
required_role = 'editor'
if check_role_permission(user_roles, required_role):
    print("User has role-based permission.")
else:
    if TimeBasedPolicy().evaluate_policy({'time': datetime.now().hour}):
        print("User has ABAC permission during work hours.")
    else:
        print("User does not have the required permissions.")
```

**Example input / output:**

```text
Request:
- User ID: 1
- Document ID: 1

Response:
- Permission Check Result: User has role-based permission.
- Time-based ABAC Policy Evaluation: True (User can access public documents during work hours)
```

**Data:**

```json
{
  "users": [
    {"id": 1, "roles": ["admin", "viewer"]},
    {"id": 2, "roles": []}
  ],
  "documents": [
    {"id": 1, "attributes": {"sensitivityLevel": "public"}},
    {"id": 2, "attributes": {"sensitivityLevel": "private"}}
  ]
}
```

**Flow:**

1. Yes
2. No
3. True
4. False
5. User Request
6. Has User Role?
7. Grant Permission
8. Policy Evaluation
9. Time-based Policy?
10. Grant Permission
11. Deny Permission

