---
type: "practice"
difficulty: "Intermediate"
---

# What is a provider package in Airflow 2 and why were providers split out of the core?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Airflow Architecture](<../../../../TOOLS/Apache_Airflow/Airflow_Core/Airflow_Architecture.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A provider package in Apache Airflow 2 refers to an optional third-party library that extends Airflow's functionality by adding new operators, sensors, hooks, and other components. These packages are separated from the core Airflow codebase to maintain modularity and facilitate easier updates and maintenance. Providers split out of the core allow for a cleaner architecture and promote community-driven contributions without affecting the stability of the main Airflow project.

The question essentially asks why providers were decoupled in Airflow 2, which involves understanding the design philosophy behind this change and its benefits. We'll break it down from first principles to understand the reasoning and full solution.

**Code:**

```python
# Example code snippet for a simple operator within an Airflow provider package
from airflow.providers.postgres.hooks.postgres import PostgresHook

class CustomPostgresOperator:
    def execute(self, context):
        hook = PostgresHook(postgres_conn_id='my_connection')
        # Execute some SQL query here
```

