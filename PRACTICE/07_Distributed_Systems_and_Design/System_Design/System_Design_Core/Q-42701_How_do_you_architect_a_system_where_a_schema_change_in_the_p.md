---
type: "practice"
difficulty: "Deep"
---

# How do you architect a system where a schema change in the primary database must safely ripple through the ORM, the GraphQL/REST layer, generated client types, and the frontend, all in a single coordinated release across teams? What breaks the atomicity?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Coordinated Client Upgrades](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Coordinated_Client_Upgrades.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this system design challenge, we need to ensure that any schema change in our primary database can be seamlessly propagated through multiple layers of infrastructure—ORM, GraphQL/REST layer, generated client types, and frontend—all while maintaining atomicity across teams. The key idea here is to implement a coordinated release strategy that minimizes downtime and ensures consistency.

To achieve this, we need to break down the problem into its core components: database schema changes, ORM mapping, API layer handling, client-side type definitions, and frontend updates. Each of these layers interacts with each other in complex ways, so we must carefully design our system to handle them cohesively. The potential pitfalls include versioning mismatches, data inconsistency, and coordination challenges.

#### Triaging the Question

The core question here is how to maintain atomicity when making schema changes across various layers. We need to ensure that all components are updated simultaneously or not at all, which requires robust versioning and synchronization mechanisms. The interviewer will be probing for your understanding of database migrations, API versioning strategies, and client-side updates.

#### First-Principles Solution
Database Schema Versioning: Implement a versioned schema system where the current database state is tagged with a version number. This allows us to track which parts of the application are compatible with each version.
ORM Layer Updates: Ensure that ORM mappings reflect these schema changes correctly. We need mechanisms to handle backward compatibility and forward migration strategies.
API Versioning: Introduce API versioning on the GraphQL/REST layer so that old clients can continue functioning while new ones start using updated APIs.
Generated Client Types: Regenerate client types from a central source to ensure they are in sync with the latest schema versions.
Frontend Update Strategy: Implement a strategy where frontend updates happen only when all backend components are ready for them.
Detailed Solution
Database Schema Versioning
Use a database migration tool like Alembic, Flyway, or Liquibase to manage schema changes and version them.
Each deployment will increment the schema version number and apply the necessary migrations.
ORM Layer Updates
Modify ORM mappings based on new schema definitions.
Implement a migration strategy where existing models can be updated over time without breaking current applications.
API Versioning
Introduce versioned endpoints for APIs to support both old and new clients simultaneously.
Use URL or header-based versioning (e.g., https://api.example.com/v1/users vs. v2/users).
Generated Client Types
Use a service like GraphQL Codegen to automatically generate client-side type definitions from the API schema.
Frontend Update Strategy
Ensure that frontend updates are part of a coordinated release with backend changes.
Implement a mechanism for testing new frontend versions in a staging environment before rolling out.

#### Common Mistakes
and Follow-Ups
Mistake 1: Failing to handle backward compatibility during ORM or API versioning. Backward compatibility issues can cause downtime and user disruption.
Mistake 2: Not using a proper database migration tool, leading to schema inconsistencies across deployments.
Mistake 3: Scheduling frontend updates without adequate testing, which can lead to bugs in production.

#### Closing Summary

By implementing versioned schemas, ORM mappings, API layers with versioning, and synchronized client-side types, we can ensure that all components are updated atomically. The key is to carefully manage the transition phases and maintain robust synchronization mechanisms across teams.

**Comparison:**

```
Schema Version	Description
v1.0	Initial schema with basic fields
v2.0	Updated schema including new fields
```

**Code:**

```python
# Example of a database migration using Alembic
def upgrade():
    op.add_column('users', sa.Column('new_field', sa.String(), nullable=True))
```

**Example input / output:**

```text
{
  "old_api_version": "/v1/users",
  "new_api_version": "/v2/users"
}
```

**Data:**

```yaml
versioned_schema:
  - version: v1.0
  - version: v2.0
```

**Flow:**

1. Database Migration
2. ORM Mapping Update
3. API Versioning
4. Client Type Generation
5. Frontend Update

