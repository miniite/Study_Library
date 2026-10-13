---
type: "practice"
difficulty: "Intermediate"
---

# If your streaming job is producing duplicate records downstream, how would you start investigating?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Bad Record Handling](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Bad_Record_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a situation where a streaming job produces duplicate records downstream, the first step is to understand that duplicates can arise from various sources and processing stages. To investigate this issue effectively, we need to identify whether it's an ingestion problem, a transformation issue, or something else in our pipeline.

Triaging the Problem

The core of the question revolves around identifying the source of duplicate records within your streaming job. You should ask yourself: Are duplicates introduced during data ingestion? During processing logic? Or do they emerge due to architectural flaws such as retries and idempotency issues?

#### Assumptions
The issue is confirmed, and not just a temporary fluke.
The environment includes common logging tools like Apache Flume or Logstash for tracing data paths.
Step-by-Step Investigation
1. Log Analysis:
Begin by checking the logs from the source systems where the data is being ingested. Look for any anomalies in how the records are being generated.
For example, if you're pulling data from a database, check for any issues like primary key violations or concurrent write operations.
2. Pipeline Profiling:
Use tools specific to your streaming framework (e.g., Apache Kafka’s Consumer Groups, Flink's Execution Graph) to trace the flow of records through each stage.
Identify which stages are potentially responsible for introducing duplicates. For instance, in a stream processing job using Flink, check if any stateful operations might be causing this issue.
3. State Management:
In a streaming context, understanding state management is crucial. Ensure that state updates and checkpoints are correctly handling the data.
For example, if you’re performing joins or aggregations, ensure that these operations are idempotent. If not, consider implementing deduplication logic at those points.
4. Duplicate Detection:
Implement a simple deduplication mechanism in your pipeline to see if removing duplicates mitigates the issue.
You can use a hash-based approach where you maintain a set of seen records and drop any duplicates.

#### Solution

Given these steps, here's an outline of how we might implement a basic solution:

PYTHON
# Example Python code snippet using Flink for deduplication
from pyflink.dataset import ExecutionEnvironment
from pyflink.table import TableConfig, DataTypes, StreamTableEnvironment
from pyflink.table.descriptors import Schema, OldCsv, FileSystem

env = ExecutionEnvironment.get_execution_environment()
t_config = TableConfig()
t_env = StreamTableEnvironment.create(env, t_config)

# Read the input data stream
t_env.connect(FileSystem().path('input_path'))
     .with_format(OldCsv()
                  .field('id', DataTypes.INT())
                  .field('value', DataTypes.STRING()))
     .with_schema(Schema()
                  .field('id', 'INT')
                  .field('value', 'STRING'))
     .register_table_source('source')

# Register the table
t_env.from_path('source').execute_insert(
    'target_table',
    Schema()
        .field('id', DataTypes.INT())
        .field('value', DataTypes.STRING()),
    'SELECT id, value FROM source WHERE NOT EXISTS (SELECT 1 FROM target_table WHERE id = source.id)'
)

#### Closing Summary

By following these steps and using the provided solution, you can systematically investigate and resolve issues with duplicate records in your streaming job. Key areas to focus on include log analysis, pipeline profiling, state management, and implementing deduplication mechanisms.

#### Common Mistakes/Follow-Ups

Mistake: Assuming the issue lies solely within a single component of the system.

#### Follow-Up
Ensure you comprehensively trace data flow across all components.

Mistake: Ignoring architectural constraints like idempotence and state management.

#### Follow-Up
Review and enforce these principles in your design.

**Comparison:**

```
ID	Value
1	A
2	B
3	C

Processed:

ID	Value
1	A
2	B
3	C
```

**Code:**

```python
from pyflink.dataset import ExecutionEnvironment
from pyflink.table import TableConfig, DataTypes, StreamTableEnvironment
from pyflink.table.descriptors import Schema, OldCsv, FileSystem

env = ExecutionEnvironment.get_execution_environment()
t_config = TableConfig()
t_env = StreamTableEnvironment.create(env, t_config)

# Read the input data stream
t_env.connect(FileSystem().path('input_path'))
     .with_format(OldCsv()
                  .field('id', DataTypes.INT())
                  .field('value', DataTypes.STRING()))
     .with_schema(Schema()
                  .field('id', 'INT')
                  .field('value', 'STRING'))
     .register_table_source('source')

# Register the table
t_env.from_path('source').execute_insert(
    'target_table',
    Schema()
        .field('id', DataTypes.INT())
        .field('value', DataTypes.STRING()),
    'SELECT id, value FROM source WHERE NOT EXISTS (SELECT 1 FROM target_table WHERE id = source.id)'
)
```

**Example input / output:**

```text
Input Path: /data/input
Output Table: output_table

Expected:
- Input records with duplicate IDs should be filtered out.
- Only unique records are written to the target table.
```

**Data:**

```json
{
  "input_path": "/data/input",
  "target_table": "output_table"
}
```

**Flow:**

1. Yes
2. No
3. Data Ingestion
4. Is Deduplication Needed?
5. Deduplicate Records
6. Forward Records
7. Write to Target Table
8. Write to Target Table

