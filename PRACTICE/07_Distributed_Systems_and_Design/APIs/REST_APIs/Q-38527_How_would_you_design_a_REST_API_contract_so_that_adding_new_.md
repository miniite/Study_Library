---
type: "practice"
difficulty: "Deep"
---

# How would you design a REST API contract so that adding new required fields to a request does not break existing clients?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ API Versioning and Deprecation](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/API_Versioning_and_Deprecation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that adding new required fields to a request in a REST API doesn't break existing clients, we need to adopt a design principle that is both forward-compatible and robust. This involves using optional parameters with default values, leveraging HTTP methods effectively, and possibly employing JSON schemas or type definitions for validation. Let’s explore each aspect step-by-step.

First, the core idea here is to maintain backwards compatibility while still allowing future extensibility. For a backend SDE-2/3, we need to consider both the practical implementation details and strategic considerations such as performance and ease of use for clients.

Optional Parameters with Default Values: When defining new fields in a request, they should be optional by default unless explicitly required at some point. This can be achieved using query parameters or JSON properties that have sensible default values.

HTTP Methods and Semantics: Use HTTP methods wisely to align with the semantics of your operations. For instance, GET requests should not modify state; use POST, PUT, PATCH, etc., appropriately based on what you want to achieve.

JSON Schema Validation: Introduce JSON schema validation for incoming requests. This will ensure that all required fields are present and correctly formatted, even if some are optional. JSON schemas can be defined using standards like JSON Schema or OpenAPI.

Versioning the API: Implement API versioning to manage changes over time. Clients can stick with older versions while newer functionality is introduced incrementally without disrupting existing clients.

Let’s break this down in more detail:

#### Triage

The question is essentially asking how to design a REST API that gracefully handles updates to request structures without breaking compatibility with pre-existing clients. The key here is to ensure flexibility and forward compatibility through thoughtful design choices.

#### First-Principles Solution

Define Optional Fields: Use optional fields in your request payloads, ensuring they have sensible default values. For example:

JSON
{
  "name": "John Doe",
  "age": 30,    // Optional with a default value of 0 if not provided.
  "email": "john@example.com"
}

Use JSON Schemas: Define a schema for your request payloads. This helps in validating incoming requests and ensuring that all required fields are present:

JSON
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "name": { "type": "string" },
    "age": { "type": "integer", "default": 0 },
    "email": { "type": "string" }
  },
  "required": ["name", "email"]
}

Leverage HTTP Methods: Ensure that your API methods are semantically correct and align with REST principles:

GET: Retrieve data.
POST: Create new resources.
PUT: Update existing resources.
PATCH: Partially update resources.

Versioning the API: Version your API to manage changes over time. This allows you to introduce breaking changes in a controlled manner:

HTTP
GET /api/v1/users/123

#### Closing Summary
and Common Mistakes/Follow-Ups

By following these principles, we can design a REST API that is both flexible and robust. Key takeaways include the importance of optional parameters with default values, JSON schema validation for robustness, appropriate use of HTTP methods, and versioning to manage changes over time.

#### Common mistakes
often involve making all fields required without providing sensible defaults or not considering how new API versions will impact existing clients. Always think about backward compatibility and the long-term maintainability of your API design.

