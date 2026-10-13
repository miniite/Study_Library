---
type: "practice"
difficulty: "Deep"
---

# You need to backfill 3 years of a redesigned fact table without impacting the nightly SLA. How do you sequence and throttle it?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Data Warehouse Fundamentals](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Data_Warehouse_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of backfilling a redesigned fact table spanning three years while maintaining an existing nightly SLA, we need to carefully plan and implement a strategy that ensures data integrity, minimizes resource consumption, and avoids any potential service disruptions. The key is to sequence tasks logically and throttle them appropriately.

#### Triaging the Question

The question primarily asks how to efficiently manage a large-scale data backfill task without impacting regular operations. This involves understanding the current system's constraints, such as nightly SLA requirements, and designing a plan that can handle the backfill in an optimal manner.

#### Assumptions
The existing fact table has been redesigned with significant changes.
A nightly ETL process is currently running, which must continue without interruption.
There are limited resources available for the backfill operation.
We need to ensure that the backfill does not violate any data consistency rules.

#### First-Principles Solution

The solution involves breaking down the backfill task into manageable chunks and strategically scheduling these tasks during off-peak hours or leveraging a separate infrastructure to handle the backfill without affecting regular operations.

Data Identification: Identify the range of data that needs to be backfilled (three years).
Task Chunking: Divide the backfill process into smaller, parallelizable tasks.
Resource Management: Ensure sufficient resources are allocated for both the nightly ETL and backfill processes.
Throttling Strategy: Implement a throttling mechanism to control the rate of data processing during the backfill.
Step-by-Step Solution

Identify Data Range:

Determine the start and end dates for the three-year period that needs to be backfilled.
For example, if today is 2023-11-01, then the range would be from 2020-11-01 to 2023-11-01.

Task Chunking:

Break down the three-year period into smaller chunks that can be processed in parallel.
For instance, split the data into monthly or weekly intervals.

#### Example
2020-11-01 to 2021-10-31, 2021-11-01 to 2022-10-31, and so on.

Resource Allocation:

Ensure that the nightly ETL process has adequate resources.
Allocate additional resources specifically for backfill tasks, either by increasing capacity or running them during off-hours.

Throttling Mechanism:

Use a throttling mechanism to control the rate of data processing.
This could be implemented through configuration settings in your ETL tool (e.g., Apache Airflow) or through resource management policies in your database system.

#### Implementation
Example

Identify Data Range:

PLAINTEXT
Start Date: 2020-11-01
End Date: 2023-11-01

Task Chunking:

Break down into monthly intervals.

#### Example
Tasks: 2020-11-01 to 2021-10-31, 2021-11-01 to 2022-10-31, 2022-11-01 to 2023-10-31.

Resource Allocation:

Ensure nightly ETL has a steady state.
Allocate additional resources for backfill tasks, e.g., using a dedicated cluster or running during off-hours.

Throttling Mechanism:

Configure Airflow to limit the number of concurrent tasks.

#### Example
Set max_active_tis_per_dag in Airflow configuration to control parallel task execution.

#### Closing Summary

By carefully breaking down the backfill process and managing resources effectively, we can ensure that the nightly ETL SLA remains intact while still completing the necessary data backfill. Throttling mechanisms help prevent resource overloads and maintain system stability.

#### Common Mistakes/Follow-Ups
Overloading Resources: Allocating too many tasks simultaneously can overwhelm the system.
Ignoring Data Dependencies: Failing to consider dependencies between different data sources.
Not Using Off-Hours: Running backfill during regular business hours can impact performance and SLAs.

**Comparison:**

| Chunk | Start Date | End Date |
|-------|------------|----------|
| 1     | 2020-11-01 | 2021-10-31 |
| 2     | 2021-11-01 | 2022-10-31 |
| 3     | 2022-11-01 | 2023-10-31 |

**Code:**

```python
# Example Airflow DAG configuration for throttling
dag = DAG(
    'backfill_dag',
    max_active_tis_per_dag=5  # Limit concurrent tasks to 5
)

task1 = PythonOperator(
    task_id='process_first_chunk',
    python_callable=lambda: process_data('2020-11-01', '2021-10-31')
)
```

**Example input / output:**

```text
Input:
Start Date: 2020-11-01
End Date: 2023-11-01
Chunk Size: monthly

Output:
Backfill tasks scheduled for:
1. '2020-11-01' to '2021-10-31'
2. '2021-11-01' to '2022-10-31'
3. '2022-11-01' to '2023-10-31'

Airflow DAG configured with max_active_tis_per_dag=5
```

**Data:**

```json
{
    "start_date": "2020-11-01",
    "end_date": "2023-11-01",
    "chunk_size": "monthly",
    "resources": {
        "cpu": 4,
        "memory_gb": 8
    }
}
```

**Flow:**

1. Yes
2. Yes
3. Identify Data Range
4. Break into Chunks?
5. Chunk Data Monthly
6. Allocate Resources
7. Throttle Processing Rate?
8. Configure Throttling in Airflow
9. Schedule Backfill Tasks
10. Monitor and Adjust as Needed

