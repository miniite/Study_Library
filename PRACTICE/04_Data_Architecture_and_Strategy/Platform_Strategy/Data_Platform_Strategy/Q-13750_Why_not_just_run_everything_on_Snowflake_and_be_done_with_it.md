---
type: "practice"
difficulty: "Deep"
---

# Why not just run everything on Snowflake and be done with it — what forces you to introduce a second compute engine into the platform and how do you avoid two-engine sprawl?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Open vs Proprietary Formats](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Data_Platform_Strategy/Open_vs_Proprietary_Formats.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When considering whether to run an entire data platform on just one compute engine like Snowflake, there are several factors that compel us to introduce a second compute engine, particularly in complex and large-scale environments. These include cost optimization, performance considerations, data governance needs, and the need for flexibility. However, it's crucial to avoid "two-engine sprawl," which can lead to increased complexity and maintenance overhead.

Firstly, Snowflake is an excellent choice for its managed nature, ease of use, and powerful data warehousing capabilities. It excels in handling large-scale analytical queries and provides a seamless experience with features like automatic scaling and zero administration. However, it isn't designed for all types of workloads.

Secondly, the cost structure of Snowflake is based on storage and compute usage, which can become prohibitive when running complex ETL processes or real-time streaming workloads. For these scenarios, introducing a second compute engine such as Apache Spark on an Elastic MapReduce (EMR) cluster or Kubernetes-based solutions like Dataflow can be more cost-effective.

Thirdly, performance is another critical factor. While Snowflake is optimized for SQL-heavy query processing and interactive analytics, some data engineering tasks require batch processing with complex transformations, which might not perform optimally within the Snowflake architecture. For instance, ETL jobs that involve heavy data ingestion and transformation can benefit from a distributed compute engine like Spark or Flink, which are designed to handle such workloads efficiently.

Fourthly, data governance and security requirements often necessitate additional compute engines. Different environments may require specific policies for data lineage, audit trails, and access controls. While Snowflake offers robust security features, integrating other tools can provide more granular control over data usage and compliance enforcement.

Finally, flexibility is key in modern data platforms. The ability to choose the right tool for each task ensures that the platform remains adaptable as new technologies emerge or as business requirements change.

To avoid "two-engine sprawl," we need a clear strategy for integrating these engines. This includes defining roles and responsibilities, ensuring seamless data movement between systems, and maintaining consistent security and governance practices across all components of the platform. A well-defined integration layer using tools like Apache Airflow can help manage workflows that span multiple compute engines.

In summary, while Snowflake offers numerous benefits, its limitations in certain areas necessitate the use of additional compute engines. Careful planning and architecture are essential to leverage each tool's strengths while minimizing complexity and overhead.

**Comparison:**

```
Step	Task	Engine	Description
1	Extract	SNOWFLAKE	Retrieve data from the source database.
2	Transform	EMR	Perform complex transformations using Spark/Flink.
3	Load	SNOWFLAKE	Insert transformed data into the target database.
```

**Code:**

```python
# Example Airflow DAG for data movement between Snowflake and EMR
from datetime import timedelta

from airflow import DAG
from airflow.providers.amazon.aws.operators.emr import EmrAddStepsOperator
from airflow.providers.snowflake.operators.snowflake import SnowflakeOperator
from airflow.utils.dates import days_ago

default_args = {
    'owner': 'data_engineer',
    'depends_on_past': False,
    'start_date': days_ago(2),
}

dag = DAG(
    dag_id='snowflake_emr_integration_dag',
    default_args=default_args,
    schedule_interval=timedelta(days=1),
)

extract_data = SnowflakeOperator(
    task_id='extract_data_from_snowflake',
    sql="""
        SELECT * FROM my_database.my_table
    """,
    parameters={'table_name': 'my_table'},
    conn_id='snowflake_conn_id',
    dag=dag,
)

transform_data = EmrAddStepsOperator(
    task_id='transform_data_on_emr',
    job_flow_id="{{ task_instance.xcom_pull(task_ids='create_emr_cluster', key='return_value') }}",
    aws_conn_id='aws_default',
    steps=[
        {
            'Name': 'Transform Data',
            'ActionOnFailure': 'CONTINUE',
            'HadoopJarStep': {
                'Jar': 'command-runner.jar',
                'Args': [
                    '/usr/bin/spark-submit', '--class', 'com.example.MyTransformer',
                    's3://my-bucket/my-transformer-jar.jar', --table my_table
                ],
            },
        }
    ],
)

load_data = SnowflakeOperator(
    task_id='load_transformed_data_into_snowflake',
    sql="""
        INSERT INTO target_database.target_table (columns)
        VALUES (%s)
    """,
    parameters={'data': 'transformed_data'},
    conn_id='snowflake_conn_id',
    dag=dag,
)

extract_data >> transform_data >> load_data
```

**Example input / output:**

```text
Input:
- Data extracted from Snowflake table "my_table"

Output:
- Transformed data loaded into Snowflake table "target_table"
```

**Data:**

```json
{
  "emr_cluster": {
    "id": "j-1234567890ABCDEF",
    "status": "RUNNING"
  },
  "snowflake_connection": {
    "account": "my_account",
    "user": "my_user",
    "password": "my_password"
  }
}
```

**Flow:**

1. Yes
2. No
3. Extract Data
4. Is Transformation Needed?
5. Transform Data with Spark
6. Load Data to Snowflake

