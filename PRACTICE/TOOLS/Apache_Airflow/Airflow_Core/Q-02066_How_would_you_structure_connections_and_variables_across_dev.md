---
type: "practice"
difficulty: "Deep"
---

# How would you structure connections and variables across dev staging and prod so a DAG promotes cleanly through environments without code changes?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ DAG Design and Dependencies](<../../../../TOOLS/Apache_Airflow/Airflow_Core/DAG_Design_and_Dependencies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that Apache Airflow Directed Acyclic Graphs (DAGs) promote cleanly from development, staging, to production with minimal or no code changes, you need to carefully manage connections and variables across environments. This involves setting up a modular structure for DAGs and leveraging environment-specific configurations.

#### Triaging the Question

The question is essentially about how to maintain consistency and avoid errors when promoting Airflow DAGs from one environment to another. The interviewer is probing your understanding of environment management, Airflow’s architecture, and best practices for variable handling. We need to assume that the environments are distinct (e.g., dev, staging, prod) with varying configurations.

#### First-Principles Solution
Environment Configuration: Use environment-specific configurations via Variable API or Environment Variables.
DAG Directory Structure: Organize DAGs and their dependencies in a way that minimizes code changes during promotion.
Variable Management: Define variables at the provider level to ensure they are accessible across environments.
Step-by-Step Solution

Environment Configuration:

Use Airflow's Variable API to store environment-specific configurations such as database credentials, API keys, or paths to files.

#### Example
For a database connection in dev, staging, and prod environments, you might use variables like DEV_DB_CONN, STAGING_DB_CONN, and PROD_DB_CONN.

DAG Directory Structure:

Organize DAGs into directories based on their function or purpose.
Use environment-specific subdirectories to keep configuration files separate but logically linked.

Variable Management:

Define variables at the provider level (e.g., airflow.providers.sql) which can be overridden by environment-specific variables.

#### Example
Set a default variable in Airflow's settings file, e.g., DB_CONN, and override it with specific values for each environment.

#### Implementation

Defining Variables:

PYTHON
from airflow.models import Variable

# Define a base connection (e.g., SQL Server)
Variable.set('BASE_DB_CONN', 'sqlserver://username:password@localhost:1433/database_name')

# Override for dev environment
Variable.set('DEV_DB_CONN', 'sqlserver://dev_username:dev_password@localhost:1433/dev_database_name')

Using Variables in DAGs:

PYTHON
from airflow import DAG
from airflow.providers.sqlite.hooks.sqlite import SqliteHook
from airflow.models import Variable

def get_db_conn():
    return Variable.get('DB_CONN')

with DAG(dag_id='example_dag', schedule_interval=None, start_date=datetime(2021, 1, 1)) as dag:
    db_hook = SqliteHook(sql=get_db_conn())
    # Use the hook to run queries or extract data

Directory Structure:

CODE
dags/
├── dev/
│   └── example_dag.py
├── staging/
│   └── example_dag.py
└── prod/
    └── example_dag.py

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Common Mistakes

Failing to use the Variable API, leading to hardcoded values.
Not organizing DAGs in a modular structure, making it difficult to promote changes.
Overwriting base variables without proper overrides.

#### Follow-Up
Questions:

How would you handle secrets management?
What are some best practices for testing and deploying Airflow DAGs?
Triangulating the Solution

By using environment-specific configurations through Variable API, organizing DAGs logically, and leveraging provider-level defaults, we can ensure that promoting DAGs from one environment to another is seamless. This approach minimizes code changes and reduces the risk of errors.

**Code:**

```python
from airflow import DAG
from airflow.providers.sqlite.hooks.sqlite import SqliteHook
from airflow.models import Variable

def get_db_conn():
    return Variable.get('DB_CONN')

with DAG(dag_id='example_dag', schedule_interval=None, start_date=datetime(2021, 1, 1)) as dag:
    db_hook = SqliteHook(sql=get_db_conn())
```

**Flow:**

1. Development Environment
2. Define Base Variables
3. Staging Environment
4. Override Variables as Needed
5. Production Environment
6. DAG Directory Structure
7. Modular DAG Files
8. Environment-SpecificSubdirectories

