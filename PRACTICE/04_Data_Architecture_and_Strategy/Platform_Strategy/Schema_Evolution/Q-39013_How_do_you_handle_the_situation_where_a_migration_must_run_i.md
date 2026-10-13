---
type: "practice"
difficulty: "Deep"
---

# How do you handle the situation where a migration must run inside application deploy CD but the DB change and code change have different rollback semantics? Describe your deploy-and-migrate ordering contract.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Schema Changes Under Long Transactions](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Schema_Changes_Under_Long_Transactions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with schema evolution and migration strategies in an environment where migrations need to occur within a Continuous Deployment (CD) pipeline, especially when there are conflicting rollback semantics between database changes and code changes, it is essential to carefully orchestrate the deployment and migration processes. This involves defining clear contracts or ordering rules that ensure both components can be rolled back independently if necessary.

The core challenge here is ensuring that both the application and the database migrations can revert to a previous state in case of failure, without interfering with each other. Let's break down this problem from first principles.

**Comparison:**

```
Step	Description	Rollback Actions
A	Apply code changes	Skip
B	Migrate database schema	Allow migration
C	Deploy updated application	Roll back
D	Start DB migration	Continue
E	DB Migration Failure	Rollback App
F	App Rollback Failure	No rollback
G	App Rollback Success	Rollback Database
H	App Rollback Fail - DB	Continue
I	DB Rollback	Roll back
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Application Code Change] --> B[Migrate DB Schema];
    C[Deploy Application Code] --> D[Start DB Migration];
    E[DB Migration Failure] --> F(Rollback Application);
    G[App Rollback Failure] --> H(DB Migrates Anyway);
    I[App Rollback Success] --> J[DB Rollback];
```

**Example input / output:**

```text
Given a scenario where an application needs to be deployed and its database schema updated, the steps are as follows:
1. **Step A**: Apply code changes to the application.
2. **Step B**: Migrate the database schema to match the new version (this might involve adding or removing columns, indexes, etc.).
3. **Step C**: Deploy the updated application code.

During deployment, if step D is initiated and a failure occurs at any point:
- If `E` (DB migration fails), rollback the application (`F`) but allow the database to proceed with its migration (`H`). This ensures that even though the application fails, the database can still be upgraded.
- If `G` (App rollback fails after DB migration), attempt to roll back the app (`I`), and if successful, perform a rollback on the database (`J`).

This setup ensures that both components are isolated during failure scenarios but also coordinated to avoid data loss or corruption.
```

**Data:**

```json
{
  "migration_steps": [
    {"step": "A", "description": "Apply code changes to the application."},
    {"step": "B", "description": "Migrate the database schema to match the new version."},
    {"step": "C", "description": "Deploy the updated application code."},
    {"step": "D", "description": "Initiate a database migration during the app deployment phase."}
  ],
  "rollback_scenarios": [
    {"scenario": "E", "steps": ["F", "H"]},
    {"scenario": "G", "steps": ["I", "J"]}
  ]
}
```

**Flow:**

1. App Code Change
2. Migrate DB Schema
3. Deploy App Code
4. Initiate DB Migration
5. DB Migration Failure
6. Rollback App
7. App Rollback Failure
8. DB Migrates Anyway
9. App Rollback Success
10. Rollback DB

