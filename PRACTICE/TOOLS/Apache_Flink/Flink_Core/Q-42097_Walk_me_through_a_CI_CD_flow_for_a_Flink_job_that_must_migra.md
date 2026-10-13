---
type: "practice"
difficulty: "Deep"
---

# Walk me through a CI/CD flow for a Flink job that must migrate state via savepoints on every deploy of the transformation logic.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Zero-loss Job Upgrades](<../../../../TOOLS/Apache_Flink/Flink_Core/Zero-loss_Job_Upgrades.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When setting up a Continuous Integration and Deployment (CI/CD) pipeline for an Apache Flink streaming job, one crucial aspect is ensuring the state migration during deployments. This involves creating savepoints to capture the current state of the job before updating its logic. Here’s a deep dive into how this can be achieved:

TL;DR

This CI/CD flow ensures that your Flink job's state is migrated via savepoints whenever you deploy updated transformation logic, maintaining fault tolerance and data consistency.

#### Triage

The question asks for a detailed CI/CD workflow to migrate the state of an Apache Flink job using savepoints. The key aspects are:

Savepoint Creation: Ensuring that a savepoint is taken before updating the job.
State Migration: Using these savepoints during deployment.
Fault Tolerance and Consistency: Maintaining data integrity across updates.

#### First Principles Solution

Initial Setup

Define a Flink job with persistent state.
Configure the job to use checkpointing or savepoints, depending on your requirements for consistency levels (at-least-once or exactly-once).

CI/CD Pipeline

Stages:
Build and Test: Compile, run tests, and verify that the new version of the Flink job works as expected.
Savepoint Capture: Take a savepoint of the current running state.
Deployment: Deploy the new version using the captured savepoint.

Savepoint Management

Use Apache Flink’s savePoint API to manually or automatically capture savepoints before updates.
Store these savepoints in a reliable storage system such as HDFS, S3, or a cloud object store.

Deployment Strategy

During deployment, the new version of the job is started using the latest savepoint.
This ensures that no data is lost during the transition and the application remains fault-tolerant.

Monitoring and Rollback

Monitor the health of the updated Flink job to ensure it’s functioning correctly.
Implement a rollback plan if issues arise, reverting to the previous savepoint.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Summary: The CI/CD flow for an Apache Flink job involves capturing and using savepoints during deployments. This ensures fault tolerance and data consistency, making updates safer and more reliable.

#### Common Mistakes
Forgetting to capture a savepoint before deploying the new version of the job.
Not configuring checkpointing or savepoints correctly in your Flink job.
Ignoring proper monitoring and rollback plans.

#### Follow-Ups
What are the trade-offs between using checkpoints vs. savepoints?
How do you handle large state sizes with frequent deployments?
Can you explain how this approach ensures exactly-once processing?

**Comparison:**

| Step | Action |
|------|--------|
| 1    | Build and test new version of the Flink job. |
| 2    | Capture a savepoint from the current running state. |
| 3    | Deploy the new version using the captured savepoint. |
| 4    | Monitor the health of the updated Flink job. |

**Code:**

```python
from pyflink.dataset import ExecutionEnvironment
from pyflink.table import BatchTableEnvironment, TableConfig

env = ExecutionEnvironment.get_execution_environment()
t_config = TableConfig()
t_env = BatchTableEnvironment.create(env, t_config)

# Define a Flink job that reads from a source and writes to a sink
t_env.execute_sql("""
CREATE TABLE input_source (
  id INT,
  value STRING
) WITH ('connector'='filesystem', 'format'='csv', ...)
""")

t_env.execute_sql("""
CREATE TABLE output_sink (
  id INT,
  processed_value STRING
) WITH ('connector'='filesystem', 'format'='csv', ...)
""")

# Define the processing logic
t_env.execute_sql("""
INSERT INTO output_sink
SELECT id, value FROM input_source
""")
```

**Example input / output:**

```text
- **Initial State**:
  - `input_source` contains some initial data.

- **After Savepoint Capture**:
  - A savepoint is captured from the current running state of the Flink job.

- **Deployment**:
  - New version of the Flink job starts using the latest savepoint, ensuring no data loss and fault tolerance.
```

**Data:**

```yaml
input_source:
  connector: filesystem
  format: csv
  path: /path/to/input

output_sink:
  connector: filesystem
  format: csv
  path: /path/to/output
```

**Flow:**

1. Build and Test
2. Capture Savepoint
3. Deploy New Version
4. Monitor Health
5. Rollback if Issues

