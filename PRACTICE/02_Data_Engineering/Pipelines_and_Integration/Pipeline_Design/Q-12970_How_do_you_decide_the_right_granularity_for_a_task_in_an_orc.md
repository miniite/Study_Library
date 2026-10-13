---
type: "practice"
difficulty: "Intermediate"
---

# How do you decide the right granularity for a task in an orchestrator — one big task vs many small tasks — and what are the failure-domain implications?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Bounding Memory for Skewed Partitions](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Bounding_Memory_for_Skewed_Partitions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To determine whether to use one big task or many small tasks in a data pipeline, we need to balance execution efficiency with fault isolation. The key factors include the task's runtime, resource requirements, and the impact of failures on downstream processes. Let’s break this down step by step.

First, consider the nature of the task itself. Small tasks are easier to manage and can provide faster feedback in case something goes wrong. For example, processing a single file or updating a specific table row might be well-suited for smaller tasks. Conversely, large tasks like full data warehouse refreshes could benefit from being broken down into smaller chunks if they consume significant resources.

Next, we need to think about the failure domain. If a small task fails, it should ideally not affect other parts of the pipeline, which is easier with fine-grained tasks. On the other hand, large tasks can be more resource-intensive and might cause cascading failures if they do go wrong. It's crucial to design your tasks so that partial successes are still useful.

Let’s illustrate this with a few practical examples:

File Processing: If we’re processing individual CSV files, each file could be considered a separate task. This way, if one file fails due to an unexpected format, it won’t impact the processing of other files.
Incremental Data Updates: For updating tables in real-time systems, tasks can be designed to update specific partitions or rows. If such a small task fails, only the affected part needs reprocessing rather than redoing the entire dataset.

In terms of failure domain implications:

Granularity and Fault Isolation: Smaller tasks offer better fault isolation because they operate on smaller subsets of data. This means if one task fails, it’s less likely to bring down other parts of the pipeline.
Resource Utilization: Larger tasks can be more resource-efficient when dealing with large datasets but come at a higher risk of failure affecting multiple processes.

To summarize: Smaller tasks are generally preferred for their better fault isolation and finer control over execution. However, there should be a balance where larger tasks are used judiciously to minimize resource overhead while maintaining reliability.

#### Common mistakes
include designing overly complex tasks that can’t fail without causing significant disruption or overly granular tasks that complicate management and monitoring.

**Comparison:**

```
Task Type	Granularity	Fault Isolation	Resource Utilization
Small	High	Good	Low
Large	Low	Poor	High
```

**Code:**

```python
# Example of defining a task in a data pipeline orchestrator (using Airflow)
from airflow.models import DAG
from airflow.operators.python_operator import PythonOperator

def process_file(file_path):
    # Function to process individual file
    pass

dag = DAG('data_pipeline', description='Data processing pipeline')

task1 = PythonOperator(
    task_id='process_single_file',
    python_callable=process_file,
    op_kwargs={'file_path': '/path/to/single/file.csv'},
    dag=dag)

# Another example of a more complex, but manageable task
def process_large_data_chunk(chunk):
    # Process large data chunks with appropriate resources
    pass

task2 = PythonOperator(
    task_id='process_large_chunk',
    python_callable=process_large_data_chunk,
    op_kwargs={'chunk': 'some_large_data_chunk'},
    dag=dag)
```

**Example input / output:**

```text
# Example input and output for a simple file processing task

Input:
- File: /path/to/single/file.csv

Output:
- Processed file: /path/to/processed/file.csv
```

**Data:**

```json
{
  "file_processing_tasks": [
    {
      "task_id": "process_file_1",
      "input_path": "/path/to/file_1.csv",
      "output_path": "/processed/path/file_1_processed.csv"
    },
    {
      "task_id": "process_file_2",
      "input_path": "/path/to/file_2.csv",
      "output_path": "/processed/path/file_2_processed.csv"
    }
  ],
  "large_data_processing_tasks": [
    {
      "task_id": "process_large_chunk_1",
      "input_chunk": "chunk_1",
      "output_path": "/processed/large/chunk_1"
    },
    {
      "task_id": "process_large_chunk_2",
      "input_chunk": "chunk_2",
      "output_path": "/processed/large/chunk_2"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Task Granularity
4. Is the task critical?
5. Small Tasks
6. Larger Tasks
7. Isolation: High, Utilization: Low
8. Isolation: Low, Utilization: High

