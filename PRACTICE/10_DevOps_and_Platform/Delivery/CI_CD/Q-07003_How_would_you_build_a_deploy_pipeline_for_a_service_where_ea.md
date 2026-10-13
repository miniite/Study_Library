---
type: "practice"
difficulty: "Expert"
---

# How would you build a deploy pipeline for a service where each tenant can be on a different version (multi-version production), and how do you keep the shared database compatible across all live versions?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Shared Pipeline Changes](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build a deploy pipeline for a multi-tenant application with separate versions, we need to ensure that each tenant can operate on its own version while maintaining compatibility in the shared database. This involves several key steps:

Isolation of Tenant Environments: Ensure that each tenant's environment is isolated.
Versioned Deployment Strategy: Implement a deployment strategy that allows deploying different versions for each tenant without disrupting other tenants.
Database Compatibility: Manage database migrations and ensure backward compatibility across all live versions.

The overall approach involves using a CI/CD pipeline to automate the deployment process, ensuring smooth updates and minimal downtime. We will break down these steps in detail.

**Comparison:**

```
Tenant	Current Version	New Version
A	1.0	1.2
B	2.0	2.0
C	1.5	1.6
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Code Repository] --> B[CI Trigger]
    B --> C[Build & Test]
    C --> D[Deploy Staging]
    D --> E[Stress Testing]
    E --> F[Promote to Production]
```

**Example input / output:**

```
Input: A new update is pushed to the code repository for tenantA (version 1.2) and tenantC (version 1.6).
Output:
YAML
Copy
tenants_after_update:
  - tenantA: {version: 1.2}
  - tenantB: {version: 2.0} # unchanged
  - tenantC: {version: 1.6}
```

**Data:**

```yaml
tenants:
  - tenantA: {version: 1.0}
  - tenantB: {version: 2.0}
  - tenantC: {version: 1.5}
database_schema_versions:
  - version1_0
  - version2_0
```

**Flow:**

1. Code Push
2. CI Server
3. Build & Test
4. Deploy Staging
5. Stress Test
6. Publish to Prod TenantA
7. Publish to Prod TenantC

