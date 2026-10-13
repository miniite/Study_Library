---
type: "practice"
difficulty: "Intermediate"
---

# Your integration tests hit a shared staging database and two concurrent pipeline runs corrupt each other's fixtures; how do you give each run isolation without provisioning a full database per run?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Idempotent Deploy Jobs](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Idempotent_Deploy_Jobs/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of concurrent CI/CD pipeline runs corrupting each other’s test fixtures in a shared staging database, we need to ensure that each run has its own isolated environment. Given your current setup and constraints, I'll outline a solution using database snapshots and transactional testing.

Firstly, let's restate what the question is really asking: We need to provide isolation between concurrent CI/CD pipeline runs while minimizing resource overhead. The interviewer is probing our understanding of database management, concurrency control, and how to efficiently manage shared resources in a staging environment.

#### Assumptions

You are using a relational database (e.g., PostgreSQL, MySQL).
The database supports snapshot functionality or can be easily backed up.
Each CI/CD run should start with the same initial state of data for consistency.

Step-by-step solution:

1. Use Database Snapshots

One effective approach is to take a snapshot of the staging database at the beginning of each pipeline run and restore this snapshot before running tests. This ensures that the test environment starts in a known, clean state.

#### Implementation
Before Pipeline Start: Take a full backup or snapshot of the staging database.
Before Test Execution: Restore this snapshot to create an isolated database for the current run.
After Tests Complete: Optionally, drop the database or restore it to its previous state (if needed).
2. Transactional Testing

Another approach is to use transactional testing where each test runs inside a transaction and rolls back any changes after execution. This way, tests do not interfere with each other.

#### Implementation
Before Test Execution: Start a database transaction.
After Test Execution: Rollback the transaction (undo all changes).
Advantage: Minimal overhead; no need to restore or drop databases between runs.
3. Database Sandbox

A more advanced approach is to use database sandboxes, where each CI/CD run has its own isolated instance of a database. This can be achieved using containerization technologies like Docker.

#### Implementation
Before Pipeline Start: Spin up a new Docker container with the staging database.
During Tests: Run tests within this container.
After Tests Complete: Stop and remove the container to clean up resources.
4. Hybrid Approach

For a balanced solution, you can combine snapshotting and transactional testing. Use snapshots for initial setup but rely on transactions during test execution.

#### Implementation
Take a snapshot of the staging database at the start.
Restore this snapshot before running tests.
Each test runs inside a transaction that rolls back after completion.

#### Closing Summary

In summary, we can ensure isolation between concurrent CI/CD pipeline runs by using database snapshots, transactional testing, or containerized sandboxes. The choice depends on your specific constraints and resources available. For practical implementation, a hybrid approach combining snapshotting and transactions is often effective and efficient.

#### Common Mistakes

Not considering the overhead of full database restoration.
Ignoring the importance of consistency across runs.
Failing to account for transaction management complexity in large-scale deployments.

**Comparison:**

```
Snapshot	Action
Initial State	Back up staging_db
Pre-Test	Restore snapshot
Post-Tests	Rollback or discard changes
```

**Code:**

```python
# Example Python function to restore a database snapshot before tests
import subprocess

def restore_database_snapshot(snapshot_path):
    subprocess.run(["pg_restore", "--dbname=staging_db", snapshot_path])
```

**Example input / output:**

```text
Input:
1. Pipeline start -> Take database snapshot

Output:
- Database snapshot saved to /path/to/snapshot.dump
```

**Data:**

```json
{
  "backup_command": "pg_dump -Fc --no-owner --no-privileges --no-tablespaces -h localhost -U staging_user -d staging_db > backup.dump",
  "restore_command": "pg_restore --dbname=staging_db backup.dump"
}
```

**Flow:**

1. Yes
2. No
3. Pipeline Start
4. Take Database Snapshot?
5. Restore Snapshot
6. RUN TESTS
7. Rollback Changes
8. Pipeline End
9. Spin Up Docker Container withStaging DB
10. Run Tests in IsolatedEnvironment
11. Stop and Remove DockerContainer

