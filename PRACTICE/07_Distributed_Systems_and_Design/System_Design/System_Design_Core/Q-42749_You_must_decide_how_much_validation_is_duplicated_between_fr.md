---
type: "practice"
difficulty: "Deep"
---

# You must decide how much validation is duplicated between frontend and backend. Set an org principle for what's client-only (UX), server-only (security), and mirrored, and design a way to share validation logic so the two never diverge silently. What's the shared-schema mechanism?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Coordinated Client Upgrades](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Coordinated_Client_Upgrades.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a system that requires validation logic on both the frontend and backend, one of the key challenges is ensuring consistency between these layers to prevent silent divergence. As a Fullstack Architect/Principal, my goal would be to set an organizational principle that clearly delineates what validation should be handled by each layer, while also establishing mechanisms for sharing validation logic effectively.

To achieve this, we'll define three categories: client-only (UX), server-only (security), and mirrored validation. The shared-schema mechanism will ensure that both layers use the same schema definitions to validate data, thereby preventing silent divergence.

#### Triage

The question is probing our ability to design a robust system where validation logic is consistent across frontend and backend, while also understanding the trade-offs between client-side performance and server-side security. We need to assume that we are working with a modern web application using RESTful APIs for communication.

#### First-Principles Solution
Client-only (UX) Validation: This includes validations that enhance user experience but do not affect the system's integrity or security. Examples include form field length limits, formatting requirements (e.g., email format), and minor UI-specific rules.
Server-only (Security) Validation: These are critical for maintaining the application’s security and integrity. They should never be omitted on the client side because they prevent potential vulnerabilities (e.g., SQL injection through dynamic query parameters).
Mirrored Validation: This involves shared logic that must be identical in both layers to ensure consistency.

To share validation logic, we can use a shared schema mechanism that defines common data models and validation rules. This approach ensures that any changes in the backend validation logic are automatically reflected on the frontend without manual synchronization.

Shared Schema Mechanism
Define Common Data Models: Create a set of common schemas for all entities (e.g., users, products) used across both layers.
Automate Code Generation: Use tools or scripts to generate client-side and server-side validation code from these shared schemas. This ensures that any updates in one layer are propagated to the other automatically.
Version Control: Implement version control for schema definitions to track changes and ensure traceability.

#### Closing Summary

By adhering to a clear principle of separating validation logic into client-only, server-only, and mirrored categories, we can achieve consistency while optimizing performance and security. The shared-schema mechanism is crucial in ensuring that the validation logic remains consistent across both layers without manual intervention.

#### Common Mistakes/Follow-Ups
Forgetting Client-only Validations: A common mistake is to overlook client-side validations entirely, leading to a poor user experience.
Manual Synchronization Issues: Not using automated tools for synchronization can lead to errors when one layer changes but the other does not.
Ignoring Version Control: Lack of version control in schema definitions makes it difficult to track and manage changes over time.

By addressing these challenges proactively, we can build a robust system that maintains consistent validation logic while optimizing performance and security.

**Comparison:**

```
Layer	Validation Type	Example
Client	UX	Field length, formatting
Server	Security	SQL injection prevention
Shared	Consistency	Password complexity validation
```

**Code:**

```javascript
// Example Schema Definition (JSON)
{
  "User": {
    "name": { "type": "string", "minLength": 3, "maxLength": 50 },
    "email": { "type": "string", "format": "email" },
    "password": { "type": "string", "minLength": 8 }
  }
}
```

**Example input / output:**

```

Input: POST /user/register

Request Body:

JSON
Copy
{
  "name": "JohnDoe",
  "email": "johndoe@example.com",
  "password": "weakpassword"
}

Response (Client-side Validation Errors):

JSON
Copy
[
  { "field": "password", "message": "Password must be at least 8 characters long" }
]

Server Response:

JSON
Copy
{
  "status": "422",
  "errors": [
    { "field": "password", "message": "Password must be at least 8 characters long" }
  ]
}
```

**Data:**

```json
{
  "User": {
    "name": "JohnDoe",
    "email": "johndoe@example.com",
    "password": "securePassword123"
  }
}
```

**Flow:**

1. Client Request
2. Validate Client-side
3. Send to Server
4. Server Validate
5. Store Data
6. Clean Up

