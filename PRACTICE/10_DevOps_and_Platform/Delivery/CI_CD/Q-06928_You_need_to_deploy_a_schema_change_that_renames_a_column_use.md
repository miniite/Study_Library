---
type: "practice"
difficulty: "Expert"
---

# You need to deploy a schema change that renames a column used by three services; walk me through the full expand-and-contract sequence across code and DB so no service ever breaks.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Expand and Contract Migrations](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Idempotent_Deploy_Jobs/Expand_and_Contract_Migrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the problem of renaming a database column while ensuring zero downtime for three dependent services, we'll follow a CI/CD strategy that involves careful planning, coordination, and a well-orchestrated sequence of steps. This approach will ensure that no service breaks during the transition.

First, let's outline the high-level goals:

Minimize Downtime: Ensure that all services continue to operate without interruption.
Zero Data Loss: Preserve data integrity throughout the process.
Smooth Transition: Gradually introduce changes so that each step is reversible if issues arise.

#### Triage
Understanding the Problem and What's Being Probed

The core of this question revolves around understanding how to handle database schema migrations in a complex, multi-service environment using CI/CD principles. The interviewer will be probing your knowledge of:

Database migration strategies.
Service coordination and communication.
CI/CD pipelines.
Error handling and rollback mechanisms.

#### Assumptions
We are using Git for version control and continuous integration.
The database is PostgreSQL, but the approach can be adapted to other databases as well.
Each service has a corresponding test suite that runs on every commit.
Services communicate via REST APIs or message queues.
There are existing CI/CD pipelines in place.
Step-by-Step Solution

Plan the Migration:

Identify the column to rename and its usage across services.
Determine the impact of this change on each service.
Develop a plan that involves creating a new column, updating references, and finally removing the old column.

Create a New Column:

SQL
ALTER TABLE users ADD COLUMN renamed_column VARCHAR(100);

This step will create a new column with the desired name while retaining data integrity.

Update Service References:

Each service that uses this column needs to be updated.
Use feature flags or conditional logic to introduce the new column gradually.
For example, in Service A, you might use an environment variable USE_RENAMED_COLUMN to switch between using the old and new columns.

Write Integration Tests:

Develop tests for both the old and new code paths in each service.
Ensure that all services pass these tests before proceeding further.

Deploy Changes:

Push changes to Git repositories for all services and databases.
Trigger CI/CD pipelines to build, test, and deploy changes.
Use blue-green deployment or canary releases to ensure smooth transitions without downtime.

Monitor and Validate:

Monitor the system closely during the transition period.
Verify that data remains consistent across all services.
Ensure that no service breaks due to the change.

Complete the Migration:

Once everything is working as expected, remove the old column.
SQL
ALTER TABLE users RENAME COLUMN original_column TO renamed_column;
Finally, update any remaining references in the codebase and re-run tests.

Rollback Mechanism:

Have a rollback plan ready in case anything goes wrong during deployment.
Ensure that all services can revert to their previous state quickly if necessary.

#### Closing Summary

In summary, renaming a database column while ensuring no service breaks involves careful planning, gradual updates, and robust testing. By leveraging CI/CD pipelines, feature flags, and blue-green deployments, we can minimize downtime and ensure data integrity throughout the process.

#### Common Mistakes/Follow-Ups
Overlooking Feature Flags: Failing to use feature flags can lead to sudden outages.
Insufficient Testing: Not thoroughly testing new code paths can result in unexpected errors.
Lack of Rollback Plan: Without a rollback mechanism, any issue during deployment can cause significant downtime.

Would you like me to elaborate on any specific aspect of this process?

**Flow:**

1. Identify Column
2. Plan Migration
3. Create New Column
4. Update Service References
5. Write Integration Tests
6. Deploy Changes
7. Monitor and Validate
8. Check Data Integrity
9. Rename Column
10. Complete Migration
11. Rollback Plan

