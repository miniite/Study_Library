---
type: "practice"
difficulty: "Expert"
---

# In a multi-tenant SaaS with schema-per-tenant, how do you roll a schema change across 50,000 tenant schemas safely, track per-tenant progress, and handle tenants that fail or are mid-transaction?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Coordinated Multi-store Schema Changes](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Coordinated_Multi-store_Schema_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle the challenge of rolling out a schema change across 50,000 tenant schemas in a multi-tenant SaaS environment, we need to ensure safety, track progress, and handle failures gracefully. The core idea is to design a migration strategy that handles each tenant's schema update independently while maintaining transactional integrity and tracking the status of each migration.

The key steps are:

Identify the Schema Change: Clearly define the schema change.
Design the Migration Process: Develop a process for applying changes to individual schemas.
Track Progress and Status: Implement mechanisms to track which tenants have been updated successfully, are in progress, or failed.
Handle Failures Gracefully: Ensure that partial updates do not leave databases in an inconsistent state.

**Comparison:**

```
Tenant ID	Status	Last Updated At
1	completed	2023-10-05 14:30:00
2	failed	2023-10-05 14:31:00 (Error Msg)
```

**Code:**

```sql
-- Example of a schema change script
ALTER TABLE tenant_specific_data ADD COLUMN new_column INT;
```

**Example input / output:**

```
Input: A list of tenant IDs and the schema change script.
Output: Updated status for each tenant and any failed updates.
PYTHON
Copy
# Example input
tenants = [
    {"tenant_id": 1, "status": "not_started"},
    {"tenant_id": 2, "status": "not_started"}
]

# Example output after processing
updated_tenants = [
    {"tenant_id": 1, "status": "completed", "last_updated_at": datetime.now()},
    {"tenant_id": 2, "status": "failed", "error_message": "Some error message"}
]
```

**Data:**

```
tenants table:
JSON
Copy
{
  "tenant_id": int,
  "status": str,  // e.g., 'not_started', 'in_progress', 'completed', 'failed'
  "last_updated_at": datetime
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Tenant Schema Migrations?
7. Migrate Tenant Schemas
8. Handle Errors?
9. Mark as Completed
10. Log Error, Retry Later
11. End Migration Process
12. End Process

