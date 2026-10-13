---
type: "practice"
difficulty: "Deep"
---

# How do you communicate and schedule a risky migration with on-call, product, and downstream teams? What's in your runbook?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Schema Migration Strategy](<../../../../01_Databases_and_SQL/Database_Operations/Migrations/Concurrent_Index_Builds/Schema_Migration_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When handling a risky database migration, clear communication and meticulous planning are crucial to ensure minimal disruption and risk mitigation. This involves coordinating with on-call engineers, product managers, and downstream teams. The primary focus is on developing a comprehensive runbook that covers every aspect of the migration process.

The core idea here is to break down the complexity into manageable steps, ensuring each stakeholder understands their role and responsibilities. We'll start by defining what we mean by a "risky" migration, identify key stakeholders, and outline a structured approach for planning and execution.

**Comparison:**

| Stakeholder Role       | Responsibilities                                          |
|------------------------|----------------------------------------------------------|
| Product Manager        | Define project scope, manage timelines and dependencies  |
| On-call Engineer       | Monitor systems, respond to issues during migration     |
| Downstream Teams       | Adjust their services based on schema changes            |

**Code:**

```python
# Example of a simple runbook script to illustrate the structure

def plan_migration(migration_plan):
    # Function to plan and schedule the migration steps
    print("Planning migration with:", migration_plan)

def execute_migration(steps):
    # Execute each step in the migration process
    for step in steps:
        print(f"Executing step: {step}")

def rollback_migrate(rollback_steps, critical=False):
    if not critical:
        print("Rolling back non-critical changes")
    else:
        print("Performing a full rollback due to critical issues")

# Example runbook structure
migration_plan = {
    "stakeholders": ["product", "on-call", "downstream"],
    "steps": [
        {"step_name": "Backup databases", "responsible_party": "database_admin"},
        {"step_name": "Apply new schema", "responsible_party": "dev_team"},
        {"step_name": "Migrate data to new schema", "responsible_party": "data_migration_team"}
    ]
}

plan_migration(migration_plan)
execute_migration([step["step_name"] for step in migration_plan["steps"]])
```

**Example input / output:**

| Input: Migration Plan | Output: Steps Execution |
|-----------------------|------------------------|
| Backup databases, Apply new schema, Migrate data to new schema | Executing step: Backup databases, Executing step: Apply new schema, Executing step: Migrate data to new schema |

**Data:**

```json
{
  "stakeholders": ["product", "on-call", "downstream"],
  "steps": [
    {"name": "Backup databases", "responsible_party": "database_admin"},
    {"name": "Apply new schema", "responsible_party": "dev_team"},
    {"name": "Migrate data to new schema", "responsible_party": "data_migration_team"}
  ]
}
```

**Flow:**

1. No
2. Yes
3. Yes
4. No
5. Plan Migration
6. Define Plan
7. Is Migration Risky?
8. Execute Steps
9. Review Risks
10. Create Contingency Plan
11. Communicate withStakeholders
12. Schedule Migrations
13. Monitor and Execute
14. Handle Issues?
15. Rollback (if critical)
16. Complete Migration
17. Notify Completion to Product

