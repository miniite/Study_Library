---
type: "practice"
difficulty: "Deep"
---

# You must decide the org stance on whether index definitions live in the application repo (as migrations) or in a separate schema-registry service owned by the platform team. Frame the trade-off for auditability, deploy coupling, and drift, over a multi-year horizon.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Index Strategy and Governance at Scale](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Index_Tuning_in_Practice/Index_Strategy_and_Governance_at_Scale.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding whether to place index definitions within the application repository (as migrations) or in a separate schema-registry service owned by the platform team, we need to carefully weigh the trade-offs in terms of auditability, deploy coupling, and data drift over a multi-year horizon. The goal is to ensure that our database remains flexible, maintainable, and compliant with evolving business requirements.

Firstly, let's clarify what these terms mean:

Auditability: Ensuring that changes are tracked, reviewed, and approved before being implemented.
Deploy Coupling: How tightly the application code is coupled to the deployment processes of the database schema.
Drift: Unintended changes in data or schema that can occur over time.
TL;DR

Placing index definitions in a separate schema-registry service owned by the platform team offers better auditability and decoupling but may introduce complexity. Application migrations, on the other hand, provide tighter integration with application development processes, reducing drift issues but risking increased coupling and reduced visibility into changes.

#### Triaging the Question

The core of this question revolves around database schema management, specifically index definitions. The primary concerns are:

Auditability: Ensuring all schema changes are properly documented and controlled.
Deploy Coupling: How dependencies between application code and database schema affect deployment processes.
Drift: Avoiding unexpected changes in the data or schema over time.

#### Assumptions
The organization has a multi-year horizon for planning.
Both options will be implemented with best practices to mitigate risks.
We are considering both short-term (immediate) and long-term (multi-year) impacts.

#### First-Principles Solution
Option 1: Index Definitions in Application Repository (Migrations)

#### Pros

Tight Integration: Migrations are part of the application code, making changes straightforward.
Easier Debugging: Changes can be directly traced through the version control system.
Reduced Coupling with Platform Team: No need to coordinate with platform teams for every schema change.

#### Cons

Increased Risk of Drift: Migrations might not always be tracked or approved, leading to uncontrolled changes.
Complexity in Maintenance: Managing migrations across multiple applications can become cumbersome over time.
Option 2: Index Definitions in Separate Schema-Registry Service

#### Pros

Enhanced Auditability: All schema changes are managed centrally and reviewed before deployment.
Lower Risk of Drift: Centralized management ensures all changes are tracked, reducing the chance of unintended alterations.
Decoupling from Application Deployment: Platform teams can manage database schema changes independently.

#### Cons

Additional Complexity: Requires additional coordination with platform teams for every change.
Potential Delay in Migrations: Centralized approval processes might introduce delays during critical deployments.

#### Closing Summary
and Common Mistakes

Ultimately, the choice between these options depends on the organization’s specific needs and constraints. A balanced approach that leverages both methods could be ideal: using migrations within applications for minor changes while centralizing significant schema modifications in a platform-managed registry. This hybrid model ensures maintainability, flexibility, and compliance without overcomplicating deployment processes.

#### Common Mistakes/Follow-Ups

Overlooking the importance of consistent change management practices.
Ignoring the potential impact of increased coupling on application development workflows.
Failing to establish clear communication channels between development and platform teams.
Not considering the long-term scalability and maintainability of chosen solutions.

**Comparison:**

| Migration Approach | Schema Registry Service |
|--------------------|-------------------------|
| Tight integration with app code | Decoupled from application development |
| Easier debugging | Centralized auditability |
| Increased risk of drift | Lower risk of unintended changes |
| No additional complexity for platform team | Additional coordination required |

**Code:**

```sql
-- Example SQL for a migration in an application repository

CREATE INDEX idx_user_name ON users (name);
```

**Example input / output:**

```text
Input: Application developer wants to add an index on the 'name' column in the 'users' table.
Output:
1. Using migration approach:
   - Developer writes a migration script in the app repo and runs it during deployment.
2. Using schema-registry service approach:
   - Developer submits a request to platform team for adding the index, which gets reviewed and applied by the registry.
```

**Data:**

```json
{
  "schemaRegistryService": {
    "serviceURL": "https://registry.example.com",
    "indexDefinitions": [
      {
        "tableName": "users",
        "columnName": "name",
        "indexName": "idx_user_name"
      }
    ]
  }
}
```

**Flow:**

1. Application Developer
2. Writes Migration Script
3. Commits to App Repo
4. Deploys Application withMigrations
5. Platform Team
6. Reviews and Applies SchemaChanges through Registry
7. Centralized Audit Log
8. Auditability Enhanced

