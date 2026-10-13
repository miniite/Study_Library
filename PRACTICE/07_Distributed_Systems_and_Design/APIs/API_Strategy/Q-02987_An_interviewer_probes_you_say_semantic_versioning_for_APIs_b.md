---
type: "practice"
difficulty: "Deep"
---

# An interviewer probes: you say "semantic versioning for APIs," but what actually constitutes a MAJOR vs MINOR vs PATCH change in a REST API where clients are lenient parsers? Define the contract precisely and where semver breaks down for web APIs.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ API Versioning and Deprecation Governance](<../../../../07_Distributed_Systems_and_Design/APIs/API_Strategy/API_Versioning_and_Deprecation_Governance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're diving deep into the nuances of semantic versioning (SemVer) in the context of REST APIs, particularly focusing on how to delineate major, minor, and patch changes when client parsers are lenient. SemVer is a convention for versioning software that helps us understand what has changed between versions and allows clients to decide whether they should upgrade or not. However, semver's applicability can be tricky in web APIs where client parsers might handle differences in request/response structures more flexibly.

The core of this question lies in defining the API contract precisely and understanding the limitations of SemVer when dealing with lenient clients. Let’s break down the concept step by step.

**Comparison:**

| API Version | Changes                                 | Impact on Clients |
|-------------|-----------------------------------------|-------------------|
| Major       | New resources or behaviors              | Must handle changes |
| Minor       | Enhancements to existing resources      | Lenient parsers can adapt |
| Patch       | Backwards-compatible bug fixes          | No impact on clients |

**Code:**

```
MERMAID
Copy
graph TD;
    A[Initial API Design] --> B{Is it a new resource or behavior?};
    B -->|Yes| C[New Resource or Behavior - Major Version]
    B -->|No| D[Behavioral Changes - Minor Version]
    D --> E[Small Fixes or Backwards Compatible Changes - Patch Version]
```

**Example input / output:**

```

Example SemVer changes and their impact on client behavior:

Major Version Change: If we add a new resource or introduce a new major behavior, e.g., POST /users to create users. A client must handle this change by either implementing the new endpoint or maintaining compatibility.

JSON
Copy
# New API contract with major version bump
apiVersion: 2.0.0

resources:
  users:
    - id: user_id
      fields: [name, email]
    - id: group_id
      fields: [name]

operations:
  get_user:
    response: { success: true, data: {name: string, email: string} }

  create_user:
    request: { name: string, email: string }
    response: { success: true, id: int }

  # New operation
  add_group:
    request: { group_name: string }
    response: { success: true, id: int }

Minor Version Change: If we enhance the existing create_user endpoint by adding a new optional field like phone, it's a minor change because clients can choose to ignore or use this new information.

JSON
Copy
# New API contract with minor version bump
apiVersion: 1.1.0

resources:
  users:
    - id: user_id
      fields: [name, email]

operations:
  create_user:
    request: { name: string, email: string }
    response: { success: true, id: int }

  # New optional field in request and response
  create_user_with_phone:
    request: { name: string, email: string, phone: string? }
    response: { success: true, id: int, phone: string? }

Patch Version Change: If we fix a bug that doesn’t alter the contract, e.g., improving error handling.

JSON
Copy
# New API contract with patch version bump
apiVersion: 1.1.1

resources:
  users:
    - id: user_id
      fields: [name, email]

operations:
  create_user:
    request: { name: string, email: string }
    response: { success: true, id: int }

  # Improved error handling
  create_user_with_phone:
    request: { name: string, email: string, phone: string? }
    response: { success: true, id: int, phone: string? }
```

**Data:**

```yaml
# Example API contract schema
apiVersion: 1.2.3
baseUri: https://example.com/api/v1

resources:
  users:
    - id: user_id
      fields: [name, email]

operations:
  # GET /users/{user_id}
  get_user:
    response: { success: true, data: {name: string, email: string} }

  # POST /users
  create_user:
    request: { name: string, email: string }
    response: { success: true, id: int }

# SemVer definition example
major[1.0.0] - New resource or behavior introduced.
minor[1.1.0] - Minor enhancements to existing resources or behaviors (e.g., adding a new field without breaking the contract).
patch[1.1.1] - Bug fixes and backwards-compatible changes.
```

**Flow:**

1. Yes
2. No
3. Major Version Change
4. New Resource or Behavior?
5. Maintain Compatibility - Major
6. Check for Leniency in Parsers -Minor
7. Enhancements, New Features -Minor
8. Bug Fixes,Backwards-compatibleChanges - Patch

