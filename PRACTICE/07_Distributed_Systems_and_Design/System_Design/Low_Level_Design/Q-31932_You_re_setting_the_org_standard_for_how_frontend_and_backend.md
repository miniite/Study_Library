---
type: "practice"
difficulty: "Deep"
---

# You're setting the org standard for how frontend and backend share a single source of truth for validation and types (schema-first codegen vs shared package vs duplicated); walk through the class/module model and the failure modes when the two drift in a fullstack team.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Domain Event Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Tenant_Isolation_in_Repositories/Domain_Event_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this deep dive, we'll explore the low-level design decisions for sharing a single source of truth between frontend and backend validation and types within a fullstack development context. Specifically, we will compare schema-first codegen with shared package vs duplicated schemas, detailing their class/module models and the failure modes that arise when these two approaches diverge.

#### Triaging the Question

The core question revolves around ensuring consistency in data validation and type definitions across both frontend and backend layers of a fullstack application. The key aspects include:

#### Cons
istency: Ensuring that the schema used for validation is the same on both sides.
Maintainability: Managing changes efficiently without introducing bugs or inconsistencies.
Codegen vs Shared Package Duplications: Exploring the trade-offs between generating code from a central definition and maintaining shared packages.

#### Assumptions
We are working with a modern JavaScript/TypeScript stack.
The application uses APIs for communication between frontend and backend.
There is a need to validate data both on the client side (frontend) and server side (backend).
Class/Module Model for Schema-First Codegen
Backend: Schema Definition

The schema is defined in a central location, often in YAML or JSON format:

YAML
# schemas/validation.yaml
apiVersion: 1.0
resources:
  - name: User
    properties:
      username: { type: string, required: true }
      email: { type: string, required: true, pattern: "^[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,4}$" }
Backend: Codegen Generation

Using a tool like openapi-generator to generate backend models:

SH
# Generate models from the schema
npm run gen-models --schema validation.yaml

This generates files like User.ts in TypeScript for API handling.

Frontend: Shared Package Usage

The frontend can import these generated types directly, ensuring consistency:

TYPESCRIPT
import { User } from '@api/models';

function createUser(user: User) {
  // Validation and logic
}
Class/Module Model for Shared Package
Centralized Schema Storage

Store the schema definitions in a shared package:

TYPESCRIPT
// @shared-schema/src/user.ts
export interface User {
  username: string;
  email: string;
}

#### cons
t validationPattern = "^[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,4}$";

function validateEmail(email: string): boolean {
  return new RegExp(validationPattern).test(email);
}
Frontend and Backend Consumption

Both frontend and backend consume the shared package:

TYPESCRIPT
// Backend usage
import { User } from '@shared-schema/src/user';

app.post('/api/users', (req, res) => {

#### cons
t user: User = req.body;
  if (!validateEmail(user.email)) return res.status(400).send("Invalid email");

  // Persist logic
});

// Frontend usage
import { User } from '@shared-schema/src/user';

function createUser(user: User) {
  if (!validateEmail(user.email)) throw new Error('Invalid email');
  // API call to create user
}
Failure Modes
Schema-First Codegen
Version Mismatch: If the backend generates a new version of the schema, the frontend might not be updated.
Tooling Issues: Dependency on external tools like openapi-generator can fail or introduce bugs.
Shared Package
Maintenance Overhead: Keeping the shared package up-to-date and ensuring all dependencies are consistent.
Circular Dependencies: Managing complex interactions between multiple schemas and packages.
Testing and Validation: Ensuring that changes in one part of the codebase don’t break others, especially when they share common types.

#### Closing Summary

The choice between schema-first codegen and shared package depends on the specific needs and constraints of your project. Schema-first codegen offers a straightforward approach to generating backend models but requires careful management of tooling and versioning. On the other hand, using a shared package provides better maintainability by centralizing type definitions but introduces complexity in managing dependencies.

#### Common Mistakes/Follow-Ups
Ignoring Tooling Issues: Tools like openapi-generator can be brittle; ensure they are well-tested.
Overlooking Shared Package Complexity: Managing large schemas and multiple packages can become cumbersome, especially in large teams.
Failing to Automate Version Control: Ensure that schema changes are tracked and managed effectively.

**Comparison:**

```
Approach	Schema-First Codegen	Shared Package
Tooling Dependency	Yes, external tools	No, central management
Centralized Storage	No, generated files	Yes, shared package
Complexity	Low	High
```

**Code:**

```
schemas/validation.yaml

apiVersion: 1.0 resources:

name: User properties: username: { type: string, required: true } email: { type: string, required: true, pattern: "^[a-z0-9._%+-]+@[a-z0-9.-]+.[a-z]{2,4}$" }
CODE
Copy

```typescript
// @api/models/User.ts
import { User } from '@shared-schema/src/user';

export class User {
  constructor(public username: string, public email: string) {}
}
```

**Example input / output:**

```text
// Input for createUser function
const user = new User('john.doe', 'invalid-email');

// Output for createUser function with validation error
if (!validateEmail(user.email)) {
  throw new Error("Invalid email");
}
```

**Data:**

```yaml
# schemas/validation.yaml
apiVersion: 1.0
resources:
  - name: User
    properties:
      username: { type: string, required: true }
      email: { type: string, required: true, pattern: "^[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,4}$" }
```

**Flow:**

1. Schema-First Codegen
2. Backend Model Generation
3. Frontend Type Import
4. Consistency
5. Tooling Overhead
6. Shared Package
7. Centralized Storage
8. Multiple Module Consumption
9. Maintainability
10. Circular DependencyManagement
11. E
12. Version Mismatch
13. J
14. Failure Modes
15. D
16. Automated Version Control
17. I
18. Maintenance Overhead

