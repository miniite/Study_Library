---
type: "practice"
difficulty: "Deep"
---

# A daily DAG occasionally double-loads data into the warehouse — how do you make the load task idempotent and what patterns (delete-insert MERGE partition overwrite) would you choose for a partitioned table?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Scheduling Retries and Trigger Rules](<../../../../TOOLS/Apache_Airflow/Airflow_Core/Scheduling_Retries_and_Trigger_Rules.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of occasional double-loading in Apache Airflow, we need to ensure that our load task is idempotent, meaning it can be run multiple times without changing the final state beyond its initial execution. For a partitioned table, this involves handling edge cases where data might already exist and ensuring efficient updates.

Firstly, let’s triage what the problem entails. The question asks us to make the load task idempotent for a partitioned table in an Apache Airflow DAG that runs daily. This means we need to consider how we can handle potential duplicates gracefully while minimizing unnecessary operations and ensuring data integrity.

#### Assumptions
The data being loaded is partitioned by some key (e.g., date).
We are using PostgreSQL or a similar relational database for the warehouse.
The load_data task in Airflow runs daily, and occasional failures can lead to double-loading.

#### First Principles Solution
Task Idempotence: Ensure that running the load task multiple times does not change the state beyond its first execution. This involves checking if rows already exist before inserting or updating them.
Handling Partitions: For partitioned tables, we need to ensure that each partition is processed idempotently and that updates are applied only where necessary.
Step-by-Step Solution

Partition Check and Load:

Before loading data into a specific partition, check if the data already exists in that partition.
If it does not exist, insert the new data; otherwise, update or merge existing records.

Using PostgreSQL MERGE Statement: The MERGE statement can be used to handle both inserts and updates efficiently. It allows you to specify conditions under which rows should be inserted or updated based on their existence in the target table.

#### Example
SQL for MERGE:

SQL
-- Example of a MERGE statement for PostgreSQL
MERGE INTO warehouse_table w
USING staging_table s ON (w.partition_key = s.partition_key)
WHEN NOT MATCHED THEN
  INSERT (partition_key, column1, column2) VALUES (s.partition_key, s.column1, s.column2);

Using UPSERT (ON CONFLICT Clause):

For databases that support UPSERT (e.g., PostgreSQL), use the ON CONFLICT clause to handle duplicate key conflicts.

#### Example
using ON CONFLICT for INSERT/UPDATE:
SQL
-- Example of an ON CONFLICT statement for UPSERT in PostgreSQL
INSERT INTO warehouse_table (partition_key, column1, column2)
VALUES ('2023-10-01', 'value1', 'value2')
ON CONFLICT (partition_key) DO UPDATE SET column1 = EXCLUDED.column1;

Airflow Task Implementation:

Define a custom Airflow operator that executes the above SQL.
Use PostgresOperator or BigQueryOperator to run the SQL command in Airflow.

Error Handling and Logging:

Implement logging to capture any errors or warnings during execution.
Add retry mechanisms for transient failures.

#### Closing Summary

By using idempotent patterns like MERGE statements or UPSERT, we can ensure that our data loading task does not double-load data into the warehouse. This approach is efficient, minimizes redundant operations, and ensures data integrity even in the presence of occasional failures.

#### Common Mistakes/Follow-Ups
Not Handling Partitions: Failing to check if a partition already exists before inserting can lead to unnecessary load on the system.
Overwriting Instead of Merging: Always prefer merging or upserting over simple overwriting, as it helps maintain data consistency and reduces the risk of losing existing records unintentionally.
Ignoring Performance Considerations: While MERGE and UPSERT are powerful, they can have performance implications. Carefully consider the schema and indexing to optimize these operations.

**Comparison:**

```
Not Matched
Matched
Staging Table
Partition Check
INSERT
MERGE Update
Warehouse Table
Completed Load
```

**Code:**

```python
# Example Airflow DAG task implementation using PostgresOperator

from airflow import DAG
from airflow.providers.postgres.operators.postgres import PostgresOperator
from datetime import datetime

default_args = {
    'owner': 'data_engineer',
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

dag = DAG(
    'idempotent_data_load_dag',
    default_args=default_args,
    description='An idempotent data load task for a partitioned table',
    schedule_interval='0 0 * * *',  # Daily at midnight
    start_date=datetime(2023, 10, 1),
)

# Custom operator to execute the MERGE statement
merge_task = PostgresOperator(
    task_id='load_data_into_partition',
    postgres_conn_id='your_postgres_connection_id',
    sql="""
        -- Example of a MERGE statement for PostgreSQL
        MERGE INTO warehouse_table w
        USING staging_table s ON (w.partition_key = s.partition_key)
        WHEN NOT MATCHED THEN
            INSERT (partition_key, column1, column2) VALUES (s.partition_key, s.column1, s.column2);
    """,
    dag=dag,
)

# Task dependencies and retries setup here
```

**Example input / output:**

| Input | Output |
|-------|--------|
| `staging_data` with new entries for `2023-10-01` and existing data for `2023-10-02` | Updated `warehouse_partitioned_data` to include the new entries for `2023-10-01`, ensuring no double-loading, and merged updates for `2023-10-02`. |

**Data:**

```json
{
  "staging_table": {
    "name": "staging_data",
    "columns": ["partition_key", "column1", "column2"]
  },
  "warehouse_table": {
    "name": "warehouse_partitioned_data",
    "columns": ["partition_key", "column1", "column2"],
    "partitions": [
      {"key": "2023-10-01"},
      {"key": "2023-10-02"}
    ]
  }
}
```

**Flow:**

1. +-----------------------------+
2. | Staging Data                |
3. | - partition_key             |
4. | - column1                   |
5. | - column2                   |
6. +-----------------------------+
7. |
8. v
9. +-----------------------------+
10. | Partition Check & Load       |
11. | - If not matched, insert     |
12. | - Else, merge existing rows  |
13. +-----------------------------+
14. |
15. v
16. +-----------------------------+
17. | Warehouse Table             |
18. | - partition_key: date       |
19. | - column1                   |
20. | - column2                   |
21. +-----------------------------+

