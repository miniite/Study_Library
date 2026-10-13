---
type: "practice"
difficulty: "Expert"
---

# As principal, how do you build a migration off a vendor OLAP appliance (Teradata/Exadata) to open-format lakehouse without a multi-year stall, and how do you keep OLTP feeds stable through it?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Separating OLTP and Reporting](<../../../../02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Separating_OLTP_and_Analytics/Separating_OLTP_and_Reporting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To migrate from a vendor's OLAP appliance like Teradata or Exadata to an open-format lakehouse while keeping your OLTP feeds stable, we need to take a holistic approach that balances data integrity, performance, and cost. This migration involves multiple phases including planning, data extraction, transformation, and loading (ETL), and post-migration testing. The key steps are:

Assess the Current Environment: Understand the current schema, data distribution, and how the OLAP appliance is being used.
Design a Data Model for the Lakehouse: Plan the new schema in a way that supports both real-time querying (OLTP) and analytical queries (OLAP).
Develop ETL Pipelines: Create ETL pipelines to move data from the existing system to the lakehouse, ensuring minimal downtime.
Implement Real-Time Feeds for OLTP Systems: Ensure that live transactions continue uninterrupted throughout the migration process.
Perform Testing and Validation: Rigorously test the new setup before full deployment.

Let’s dive into each step in detail:

#### Triaging the Question

The core of this question is about transitioning from a vendor-specific, closed system to an open-format lakehouse while maintaining ongoing operations. The interviewer is likely probing your ability to plan and execute complex migrations, manage dependencies between different systems, and ensure data consistency.

Step-by-Step Solution

Assess the Current Environment

Analyze the existing Teradata/Exadata schema and performance characteristics.
Identify key performance bottlenecks that might affect migration times.
Understand user queries and their execution patterns to prioritize which parts of the database need most attention.

Design a Data Model for the Lakehouse

Design a star or snowflake schema in your lakehouse architecture, ensuring it can handle both transactional and analytical workloads efficiently.

#### Cons
ider using a hybrid model where certain tables are optimized for OLTP and others for OLAP, leveraging Apache Iceberg or Delta Lake to provide ACID transactions.

Develop ETL Pipelines

Use tools like Apache Airflow or Kubernetes pipelines to automate data movement.
Develop incremental ETL processes that only transfer changes made since the last run.
Implement a staging layer within the lakehouse for initial transformation and validation before loading into final tables.

Implement Real-Time Feeds for OLTP Systems

Use change data capture (CDC) tools like Debezium orDebeiso to extract transactional data in real time.
Set up a buffer zone between your OLTP systems and the lakehouse to avoid direct contention.
Ensure that write operations are logged and can be replayed during initial setup if needed.

Perform Testing and Validation

Conduct thorough testing using mock data to simulate production environments.
Use tools like Apache Spark for rapid prototyping and validation of complex queries.
Perform load testing to ensure the system meets performance SLAs under both normal and peak conditions.

#### Closing Summary

The key to a successful migration is meticulous planning, robust ETL processes, and careful management of dependencies. By following these steps, we can minimize disruption to ongoing operations while ensuring that the new architecture supports future growth and flexibility.

#### Common Mistakes/Follow-ups
Failing to Plan for Data Consistency: Ensuring data integrity during the transition is critical; any discrepancies could lead to significant issues.
Overlooking Real-Time Performance: Inadequate handling of real-time feeds can cause delays in transactional operations, impacting customer experience.
Underestimating Testing Requirements: Insufficient testing can result in unforeseen bugs or performance issues that only surface after deployment.

**Comparison:**

| Schema              | OLTP             | OLAP          |
|---------------------|------------------|---------------|
| Tables              | SalesTransactions| AnalyticsData |
| Storage Type        | Relational DB    | File System   |
| Query Performance   | High            | Optimized     |
| ACID Compliance     | Yes             | No            |

**Code:**

```python
from airflow.operators.python_operator import PythonOperator
from airflow.models import DAG

def extract_data():
    # Example ETL logic using CDC tool
    pass

dag = DAG(
    'migration_dag',
    schedule_interval='@daily'
)

extract_task = PythonOperator(
    task_id='extract_data',
    python_callable=extract_data,
    dag=dag
)
```

**Example input / output:**

```text
Input:
- Source data in Teradata/Exadata format
- Target schema designed for lakehouse

Output:
- Staged data transformed and loaded into the lakehouse
- Validated and tested data ready for production use
```

**Data:**

```json
{
  "schema": [
    {"name": "customer_id", "type": "int"},
    {"name": "product_id", "type": "int"},
    {"name": "quantity", "type": "int"}
  ],
  "data": [
    [1, 2001, 5],
    [2, 2002, 3]
  ]
}
```

**Flow:**

1. Current OLTP
2. CDC Extraction
3. ETL Pipeline Staging Layer
4. Lakehouse Load
5. Maintenance & Validation
6. Post-Migration Testing
7. Production Ready

