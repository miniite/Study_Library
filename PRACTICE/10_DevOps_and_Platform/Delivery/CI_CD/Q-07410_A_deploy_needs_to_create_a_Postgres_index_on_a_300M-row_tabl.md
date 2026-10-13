---
type: "practice"
difficulty: "Expert"
---

# A deploy needs to create a Postgres index on a 300M-row table and a naive CREATE INDEX locked writes for 20 minutes; how would you run it concurrently and gate the app deploy on the index being ready without holding the pipeline hostage?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Expand and Contract Migrations](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Idempotent_Deploy_Jobs/Expand_and_Contract_Migrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

To create a Postgres index on a large table while minimizing downtime, we can use concurrent operations with proper gating mechanisms. The key idea is to parallelize index creation and ensure that the application only starts using the new index after it's fully created.

#### Triaging the Question

The core of this question revolves around creating a Postgres index on a 300 million-row table in such a way that it does not block the deployment process. The challenge is to run the index creation concurrently while ensuring the application can be deployed safely only when the index is fully created.

#### Assumptions
and Context:
Table Size: 300 million rows.
Index Creation Time: Naive CREATE INDEX command locks writes for approximately 20 minutes.
Deployment Process: The app deployment must wait until the index creation is completed to avoid data inconsistencies or performance issues.

#### First Principles Solution

Parallelize Index Creation:

Use Postgres’s parallel query feature (parallel_index_create = on) to create multiple indexes concurrently, which can significantly reduce the overall time required for index creation.

Gating Mechanism:

Implement a gating mechanism using SQL queries or external scripts that monitor the progress of index creation.
Ensure that the application deployment waits until all index creation processes are completed and the new index is fully ready.

Postgres Features:

Use CREATE INDEX CONCURRENTLY to avoid locking writes during the index creation process.
Leverage Postgres’s pg_stat_progress_create_index view to monitor the progress of index creation.

Application-Level Gating:

Implement a health check in the application deployment pipeline that verifies if the new index is available before proceeding with further steps.
Step-by-Step Solution:

Parallel Index Creation:

SQL
-- Enable parallel query feature
SET enable_parallel_hash = on;
SET enable_parallel_sort = on;

-- Create multiple indexes concurrently
CREATE INDEX CONCURRENTLY idx_name ON your_table(column_name);

Monitoring Progress: Use pg_stat_progress_create_index to monitor the index creation progress:

SQL
SELECT
  pid,
  relid::regclass AS relation,
  indexrelid::regclass AS index,
  opertypname AS operation,
  opertime AS runtime,
  totaltime/1000.0 AS total_time_ms,
  (currenttimetz() - opertime) / 1000.0 AS elapsed_time_ms
FROM pg_stat_progress_create_index;

Application-Level Gating: Implement a health check in the application deployment pipeline to ensure that the new index is ready before proceeding:

BASH
# Example script for checking if the index exists and is up-to-date
psql -c "SELECT COUNT(*) FROM your_table WHERE column_name IN (SELECT column_name FROM idx_name);"

while [[ $? != 0 ]]; do
  sleep 5s
  psql -c "SELECT COUNT(*) FROM your_table WHERE column_name IN (SELECT column_name FROM idx_name);"
done

# Proceed with deployment if the index is ready

#### Closing Summary

By leveraging Postgres’s parallel query and CREATE INDEX CONCURRENTLY features, along with a robust gating mechanism in the application pipeline, we can ensure that index creation does not block the deployment process. This approach minimizes downtime and ensures data consistency.

#### Common Mistakes/Follow-Ups
Overlooking Parallel Index Creation: Not using parallel_index_create = on or CREATE INDEX CONCURRENTLY.
Failing to Monitor Progress: Not implementing a mechanism to monitor the progress of index creation.
Ignoring Application-Level Gating: Deploying the application prematurely, leading to data inconsistencies.

**Code:**

```

-- Example SQL for creating an index concurrently CREATE INDEX CONCURRENTLY idx_name ON your_table(column_name);

CODE
Copy

```bash
# Example script for checking if the index exists and is up-to-date
psql -c "SELECT COUNT(*) FROM your_table WHERE column_name IN (SELECT column_name FROM idx_name);"

while [[ $? != 0 ]]; do
  sleep 5s
  psql -c "SELECT COUNT(*) FROM your_table WHERE column_name IN (SELECT column_name FROM idx_name);"
done
```

