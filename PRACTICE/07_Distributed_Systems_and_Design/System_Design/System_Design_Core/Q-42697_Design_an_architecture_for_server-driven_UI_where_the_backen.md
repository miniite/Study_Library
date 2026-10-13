---
type: "practice"
difficulty: "Expert"
---

# Design an architecture for server-driven UI where the backend dictates screen layout to mobile clients you can't force-update. How do you evolve the schema without bricking old app versions, and where's the line between flexibility and an untestable render engine?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Coordinated Client Upgrades](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Coordinated_Client_Upgrades.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

In designing a server-driven UI architecture for mobile clients, the key is to create a flexible yet robust system that allows updates without breaking older client versions. We need to decouple presentation from business logic and ensure backward compatibility through incremental schema changes and versioning strategies.

#### Triaging the Question

The core of this question revolves around evolving a backend-driven user interface (UI) architecture for mobile clients, where the server dictates the layout but the client cannot be forced to update immediately. The challenge is twofold: ensuring that updates do not brick older app versions while maintaining flexibility and testability in the rendering engine.

#### Assumptions
Client-Specific Versions: Different app versions are supported, with each version having its own schema.
Server-Driven Layouts: The backend dictates the screen layout to ensure consistency and compliance across all clients.
Incremental Updates: Client updates can be applied incrementally, allowing for gradual adoption of new features.

#### First-Principles Solution

Versioned Schemas:

Define a versioned schema system where each client app version has its own schema file. For example, schema-v1.json, schema-v2.json, etc.
Ensure that newer schemas are backward compatible with older ones to maintain compatibility.

Dynamic Rendering Engine:

Implement a dynamic rendering engine on the backend that can interpret different schema versions and render them appropriately.
This engine should be modular, allowing for easy updates without affecting existing functionality.

Incremental Updates:

For new features or schema changes, introduce versioned API endpoints (e.g., /api/v1/layout, /api/v2/layout). New clients can opt-in to the latest APIs while older versions continue using legacy endpoints.
Use a phased rollout strategy where only a percentage of users are directed to newer APIs initially. This allows for real-world testing and bug fixing before full deployment.

Fallback Mechanisms:

If an update introduces breaking changes, provide fallback mechanisms in the rendering engine that can handle those scenarios gracefully.

#### Example
For a new layout element not supported by older schemas, default to a previous known good state or show a placeholder until the schema is updated.

Testing and Validation:

Use automated testing frameworks to validate both frontend and backend components. Ensure tests cover all versions of the schema and edge cases.
Implement canary releases to test new features in a controlled environment before full deployment.

Documentation and Communication:

Maintain clear documentation for both front-end developers and backend engineers, detailing how different schemas are interpreted and rendered.
Use version control systems like Git to manage schema files and track changes over time.

#### Closing Summary

By adopting a versioned schema approach and implementing a dynamic rendering engine, we can ensure that new features and updates do not break older client versions. This strategy balances flexibility with robustness and maintainability, ensuring that the system remains testable and easy to evolve over time.

#### Common Mistakes/Follow-Ups
Over-Engineering: Avoid making the architecture overly complex by adding unnecessary layers of abstraction.
Breaking Changes Without Fallbacks: Ensure that any new changes are backward compatible or have proper fallback mechanisms in place.
Insufficient Testing: Rigorous testing across all versions is crucial to catch and resolve issues early.
Lack of Clear Communication: Poor documentation can lead to misunderstandings and bugs, so clear communication channels are essential.

#### Example
Workflow

Let's consider a simple example where we need to add support for a new card layout:

Schema Change:

Introduce schema-v2.json with the new layout definition.

Backend Update:

Modify the backend rendering engine to recognize and render both v1 and v2 schemas.

Client Versioning:

Tag client app versions appropriately (e.g., v1.0, v2.0) and update their schema files accordingly.

Incremental Rollout:

Use A/B testing to gradually roll out the new layout to a subset of users before full deployment.

Monitoring and Feedback:

Monitor user feedback and performance metrics to fine-tune the rollout and address any issues that arise.

By following these steps, we can maintain a flexible and scalable architecture while ensuring compatibility with older client versions.

**Comparison:**

| Version | Layouts Supported           | Rendering Engine Features                  | Testing Strategy          |
|---------|----------------------------|-------------------------------------------|--------------------------|
| v1      | Home, Profile               | Basic Card Types (text, image)             | Unit tests, Integration  |
| v2      | Home, Profile, New Feature  | Enhanced Card Types + Customization       | A/B testing, Stress Tests|

**Code:**

```
MERMAID
Copy
graph LR;
    A[Client App] --> B[Server API / schema-v1.json]
    C[Backend Rendering Engine] --> D[Screen Layout Rendered]
    E[Client Update] --> F[New Version Client App]
    F --> G[Schema Migration to v2]
    G --> H[Server API / schema-v2.json]
```

**Example input / output:**

| Input | Output |
|-------|--------|
| `{ "schemaVersion": "v1", "layoutId": 1 }` | `<div>Welcome to the app!</div>` |
| `{ "schemaVersion": "v2", "layoutId": 2 }` | `<div><img src="https://example.com/profile.jpg" /></div>` |

**Data:**

```json
{
  "schemaVersion": "v1",
  "layouts": [
    {
      "id": 1,
      "name": "Home Layout",
      "cards": [
        { "type": "text", "content": "Welcome to the app!" }
      ]
    },
    {
      "id": 2,
      "name": "Profile Layout",
      "cards": [
        { "type": "image", "url": "https://example.com/profile.jpg" }
      ]
    }
  ]
}
```

**Flow:**

1. No
2. Yes
3. Client App
4. Is Schema Updated?
5. Use Legacy API / schema-v1
6. Choose v2 API Endpoint
7. Fetch New Layout & Schemafrom Server
8. Render New Layout UsingDynamic Engine

