---
type: "practice"
difficulty: "Intermediate"
---

# What breaks when you run dbt models directly from a BashOperator versus using Cosmos or the dbt-Airflow integration and how does observability differ?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Orchestration vs Processing](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Orchestration_vs_Processing/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When running dbt models directly from a BashOperator in Apache Airflow, there are several potential issues compared to using Cosmos or the dbt-Airflow integration, primarily related to observability, error handling, and maintainability. In this response, I will outline the key differences and how these impact observability.

#### Triage
Restating the Question

The question is essentially asking about the trade-offs between running dbt models directly via a BashOperator versus using the more specialized tools like Cosmos or the built-in dbt-Airflow integration. The core of the question revolves around understanding what can go wrong and how different approaches affect observability.

Triaging Assumptions
BashOperator: A generic task in Airflow that allows you to run shell commands.
Cosmos: A custom operator provided by Airflow specifically for dbt models, offering enhanced functionality.
dbt-Airflow Integration: A set of plugins and integrations that allow seamless interaction between dbt and Airflow.

#### First Principles Solution

Error Handling:

BashOperator: Error handling is more manual and requires custom logging and error catching.
Cosmos or dbt-Airflow Integration: These tools provide built-in mechanisms for error handling, including retries and more granular logging.

Observability:

BashOperator: Observability can be challenging due to the lack of built-in support for tracking model execution status, lineage, and metrics.
Cosmos or dbt-Airflow Integration: These tools provide better observability through predefined hooks, sensors, and UI integrations that help in monitoring, debugging, and managing dbt models.

Maintainability:

BashOperator: Maintenance can be more cumbersome due to the need for shell scripting expertise.
Cosmos or dbt-Airflow Integration: These tools are designed with maintainability in mind, offering cleaner code and easier updates.

Dependencies and Environment Management:

BashOperator: Managing dependencies and environments can be tricky and inconsistent across runs.
Cosmos or dbt-Airflow Integration: These tools handle dependency management more effectively, ensuring consistent environment setup.

#### Closing Summary

In summary, running dbt models directly from a BashOperator in Airflow can lead to issues related to error handling, observability, maintainability, and environment consistency. Using Cosmos or the dbt-Airflow integration provides better support for these aspects, making it a more robust solution. However, direct use of BashOperator might be suitable in scenarios where custom logic is required that isn't supported by the built-in tools.

#### Common Mistakes/Follow-Ups
Mistake: Ignoring error handling and logging when using BashOperator.

#### Follow-Up
How do you ensure data lineage and model tracking with both methods?
Run dbt Models viaBashOperator
Error Handling
Manual Logging, ErrorCatching
Observability
Custom Monitoring
Use Cosmos or dbt-AirflowIntegration
Error Handling
Built-in Mechanisms, Retries
Observability
Predefined Hooks, UIIntegrations
Maintainability
Manual Shell Scripting
Maintainability
Cleaner Code, Easier Updates
Environment Management
Inconsistent Setup
Environment Management

#### Cons
istent Environment Setup

**Comparison:**

```
Operator	Error Handling	Observability	Maintainability	Environment Management
BashOperator	Manual	Limited	Custom Scripting	Inconsistent
Cosmos	Built-in	Enhanced	Cleaner Code	Consistent
dbt-Airflow	Built-in	Enhanced	Cleaner Code	Consistent
```

**Code:**

```python
# Example BashOperator
def run_dbt_bash():
    return """
        dbt run --project-dir /path/to/project
    """

bash_task = BashOperator(
    task_id='run_dbt_models',
    bash_command=run_dbt_bash(),
    dag=dag
)
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Run dbt `run` command in BashOperator | Task logs and error messages captured in Airflow UI |

**Data:**

```json
{
  "dbt_project_dir": "/path/to/project",
  "models": ["model_a", "model_b"],
  "targets": ["prod"]
}
```

**Flow:**

1. Direct BashOperator
2. Error Handling
3. Manual Logging, ErrorCatching
4. Observability
5. Custom Monitoring
6. IntegrationCosmos/dbt-Airflow
7. Error Handling
8. Built-in Mechanisms, Retries
9. Observability
10. Predefined Hooks, UIIntegrations
11. Maintainability
12. Manual Shell Scripting
13. Maintainability
14. Cleaner Code, Easier Updates
15. Environment Management
16. Inconsistent Setup
17. Environment Management
18. Consistent Environment Setup

