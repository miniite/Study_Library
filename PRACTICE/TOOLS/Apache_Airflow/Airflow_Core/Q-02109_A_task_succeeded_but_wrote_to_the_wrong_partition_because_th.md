---
type: "practice"
difficulty: "Intermediate"
---

# A task succeeded but wrote to the wrong partition because the operator built the path from the wrong template var — how do you make templated paths robust and catch this in review?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Scheduling Retries and Trigger Rules](<../../../../TOOLS/Apache_Airflow/Airflow_Core/Scheduling_Retries_and_Trigger_Rules.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To ensure robustness of templated paths in Apache Airflow, we need to design our task templates carefully by using Jinja2 templating effectively. We should also implement rigorous validation checks and a review process that catches any discrepancies early.

#### Triaging the Question

The question is about ensuring the reliability of templated file paths used by Airflow tasks. Specifically, the task generated a successful run but wrote to an incorrect partition because it incorrectly interpreted a template variable. The key aspects the interviewer might be probing include:

Template Variable Consistency: Ensuring that all parts of the system use consistent and correct variables.
Path Validation: Implementing checks within Airflow tasks or upstream processes to validate paths before execution.
Review Process: Establishing a robust review process to catch such errors early in development.

#### Assumptions
The task uses Jinja2 templating for constructing file paths.
There are predefined partitions, each with its own unique identifier.
The path is dynamically generated based on the template and variables provided at runtime.
We can integrate validation checks into our Airflow DAG definitions.

#### First-Principles Solution
Step 1: Define Jinja2 Templates

First, we define our templates in an args dictionary or directly within the task:

PYTHON
from datetime import datetime

default_args = {
    'owner': 'airflow',
    'start_date': datetime(2023, 6, 1),
}

dag = DAG(
    dag_id='example_dag',
    default_args=default_args,
)

# Define the template variable for partition
partition_var = Variable.get('partition')
Step 2: Construct Paths Using Jinja Templates

Next, we construct our file path using a Jinja2 template:

PYTHON
from jinja2 import Template

template_path = f'/data/{partition_var}/output/{{ execution_date.year }}-{{ execution_date.month }}.csv'
task = PythonOperator(
    task_id='write_data',
    python_callable=lambda: write_to_file(template_path),
    dag=dag,
)
Step 3: Implement Path Validation

We can add validation logic to ensure that the generated path is correct:

PYTHON
def validate_path(path, partition_var):
    expected_path = f'/data/{partition_var}/output/2023-06.csv'  # Example date
    return path == expected_path

task >> BranchPythonOperator(
    task_id='validate_path',
    python_callable=lambda: validate_path(template_path, partition_var),
)
Step 4: Robust Review Process

To catch such errors early in the development process, we can use a review tool or manual checks. For example:

Pre-commit Hooks: Use pre-commit hooks to run static analysis tools like flake8 and yamllint on DAG definitions.
Code Reviews: Regular code reviews by peers who understand the partitioning schema.
Automated Testing: Write unit tests for tasks that explicitly test template variables.

#### Closing Summary

By carefully designing our templates, implementing robust validation checks, and maintaining a strong review process, we can ensure that templated paths are consistent and correct. This approach minimizes the risk of errors like writing to the wrong partition.

#### Common Mistakes/Follow-Ups
Template Variables Not Consistent: Ensure all parts of the system use the same variable names.
Insufficient Validation: Paths should be validated both at runtime and during static checks.
Manual Errors in Validation Logic: Validate paths against expected values explicitly.

Do you want to dive deeper into any specific part or discuss other related topics?

**Comparison:**

```
Partition	Expected Path	Actual Path
202306	/data/202306/output/2023-06.csv	/data/202306/output/2023-06.csv
```

**Code:**

```python
from datetime import datetime

default_args = {
    'owner': 'airflow',
    'start_date': datetime(2023, 6, 1),
}

dag = DAG(
    dag_id='example_dag',
    default_args=default_args,
)

partition_var = Variable.get('partition')

template_path = f'/data/{partition_var}/output/{{ execution_date.year }}-{{ execution_date.month }}.csv'

def validate_path(path, partition_var):
    expected_path = f'/data/{partition_var}/output/2023-06.csv'  # Example date
    return path == expected_path

task >> BranchPythonOperator(
    task_id='validate_path',
    python_callable=lambda: validate_path(template_path, partition_var),
)
```

**Example input / output:**

```text
Input:
- Partition Variable Value: '202306'

Output:
- Expected Path: '/data/202306/output/2023-06.csv'
- Actual Path Written by Task: '/data/202306/output/2023-06.csv' (Success, Correct Path)
```

**Data:**

```yaml
# Example Airflow DAG Configuration
dag:
  id: example_dag
  default_args:
    owner: airflow
    start_date: '2023-06-01'
  tasks:
  - task_id: write_data
    python_callable: lambda: write_to_file('/data/{{ partition }}/output/{{ execution_date.year }}-{{ execution_date.month }}.csv')
  - task_id: validate_path
    python_callable: lambda: validate_path('{{ ti.xcom_pull(task_ids="write_data") }}', 'partition_var')
```

**Flow:**

1. Yes
2. Correct Path
3. Incorrect Path
4. Start
5. No
6. Generate Path Using JinjaTemplate
7. Validate Path Against ExpectedValue
8. Task Success
9. Path Validation Failure, NotifyDeveloper
10. Configure Pre-commit Hooksand Code Reviews

