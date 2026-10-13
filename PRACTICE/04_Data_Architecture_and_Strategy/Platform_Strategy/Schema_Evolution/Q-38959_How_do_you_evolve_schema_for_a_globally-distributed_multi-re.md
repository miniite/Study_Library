---
type: "practice"
difficulty: "Expert"
---

# How do you evolve schema for a globally-distributed, multi-region active-active database (e.g. Spanner/CockroachDB/Yugabyte) where DDL is itself a distributed transaction? What new failure modes appear?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Schema Changes Under Long Transactions](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Schema_Changes_Under_Long_Transactions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To evolve the schema for a globally-distributed, multi-region active-active database like Google Cloud Spanner or CockroachDB, you need to address several key aspects including handling distributed transactions and ensuring data consistency across regions. The core challenge is to ensure that DDL operations are robust enough to maintain high availability while mitigating potential failure modes.

First, let's understand the question: You're asked how to safely evolve a database schema in an environment where updates must be propagated consistently across multiple regions using distributed transactions. This requires careful planning and execution, as any failure during this process could lead to inconsistent data or service outages.

#### Triaging the Question
What is being asked: How to manage schema changes while ensuring data consistency and high availability.
Key assumptions:
The database supports multi-region active-active operations.
DDL operations are themselves distributed transactions, meaning they must be completed atomically across all affected regions.
The main challenge: Ensuring that any schema change is propagated correctly to all replicas in different regions without data loss or inconsistencies.
Step-by-Step Solution
Identify the Schema Change:
Determine what changes are needed in the database schema, such as adding a column, altering a table structure, etc.
Plan the Change:
Use the database’s built-in tools to plan the change, ensuring it is designed to be applied atomically across all regions.
Test Locally:
Test the DDL operation in a local environment to ensure it works as expected before applying it globally.
Propagate the Change:
Use the database’s replication mechanism to propagate the change to all regions. This typically involves sending the DDL command over to other regions, which will then apply it atomically.
Monitor and Validate:
Monitor the process to ensure that no errors occur during propagation. Validate that the changes were applied correctly in each region.
Common Failure Modes
Transaction Failures: A distributed transaction may fail due to network issues or resource limitations, causing partial completion of the schema change.
Replication Lag: Differing replication lags between regions can lead to inconsistencies if not handled properly.
Data Loss: If a critical region fails during the operation, it could result in data loss unless there are robust backup and recovery mechanisms.

#### Closing Summary

Evolving the schema for a multi-region database like Spanner or CockroachDB requires meticulous planning and execution. You must ensure that DDL operations are applied atomically across all regions to maintain consistency. Failure modes such as transaction failures, replication lags, and data loss can occur if not properly managed.

#### Common Mistakes
Failing to test changes thoroughly in a local environment.
Not monitoring the propagation process closely to detect and address issues early.
Ignoring potential failure scenarios and their impacts on the system's availability and integrity.

**Comparison:**

| Region | Status | Last Update |
|--------|--------|------------|
| us-east1 | UP     | 2023-10-05 |
| asia-northeast1 | DOWN   | -          |
| europe-west1 | UP     | 2023-10-05 |

**Code:**

```sql
-- Example DDL command for adding a column
ALTER TABLE my_table ADD COLUMN new_column INT;
```

**Example input / output:**

```text
Input:
- DDL command to add a column

Output:
- Command propagated successfully in us-east1 and europe-west1, but failed due to network issues in asia-northeast1. Replication lags are observed.
```

**Data:**

```json
{
  "regions": ["us-east1", "asia-northeast1", "europe-west1"],
  "replicas": [
    {"region": "us-east1", "status": "UP"},
    {"region": "asia-northeast1", "status": "DOWN"},
    {"region": "europe-west1", "status": "UP"}
  ]
}
```

**Flow:**

1. Rendering diagram…

