---
type: "practice"
difficulty: "Deep"
---

# How would you split your DAG into hourly and daily job cadences using tags, and how do you keep dependencies consistent so an hourly model never reads a stale daily parent?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ dbt Models and Runs](<../../../../TOOLS/dbt/dbt_Core/dbt_Models_and_Runs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the problem of splitting a dbt DAG into hourly and daily job cadences while ensuring that hourly models read only up-to-date daily parent data, we need to carefully design our dbt project structure using tags and dependencies. The core idea is to leverage dbt's powerful tagging system to manage different schedules efficiently. Here’s how you can approach this:

#### Overview
We will split the DAG into two main categories: hourly models that require fresh daily data as their parent, and daily models that serve as parents for hourly models.

#### Triage
The question is essentially about managing dependencies in a dbt project to ensure that hourly jobs only depend on latest daily data. This involves using tags effectively and setting up appropriate model relationships.

Let's break this down step-by-step:

Step 1: Define Tags

We will create two tags:

hourly: For models that run every hour.
daily_parent: For parent models that have a full daily cycle.
Step 2: Organize Models by Cadence

Create separate directories for hourly and daily models to keep the structure clear. For example:

CODE
models/
  -- hourly/
      - model_name_hourly.sql
  -- daily/
      - model_name_daily.sql
Step 3: Define Tags in dbt_project.yml

Update your dbt_project.yml file with the tags:

YAML
model:
  tags:
    - hourly
    - daily_parent
Step 4: Tagging Models

For models that run hourly, tag them as hourly. For parent models that run daily and are dependencies for hourly models, tag them as daily_parent.

#### Example
of a Daily Parent Model (model_name_daily.sql):
SQL
-- model_name_daily.sql
{{ config(tag="daily_parent") }}

with base_table as (
  select * from {{ ref('raw_data') }}
)
select * from base_table

#### Example
of an Hourly Model (model_name_hourly.sql):
SQL
-- model_name_hourly.sql
{{ config(tag="hourly", materialized='table') }}

with daily_parent_data as (
  -- Use the result from daily parent model
  select * from {{ ref('model_name_daily') }}
)
select * from daily_parent_data
Step 5: Ensure Dependencies Are Correct

Use ref and depends_on_nodes to ensure that hourly models depend on latest daily data. For instance, if you have a model named hourly_model, it should depend on the most recent daily_model.

SQL
-- hourly_model.sql
{{ config(tag="hourly", depends_on=['daily_model_latest']) }}

with daily_data as (
  -- Use the result from daily parent model
  select * from {{ ref('daily_model_latest') }}
)
select * from daily_data
Step 6: Managing Latest Daily Data

To manage fetching the latest daily data, you can use dbt’s ref with a unique identifier. For example:

SQL
-- hourly_model.sql
{{ config(tag="hourly", depends_on=['daily_model_latest']) }}

with latest_daily as (
  select * from {{ ref('daily_model', 'latest') }}
)
select * from latest_daily

#### Closing Summary
and Common Mistakes/Follow-Ups:

By leveraging tags and appropriate dependency management, we ensure that hourly models always read the most recent daily data. Some common pitfalls include not properly tagging or configuring dependencies, leading to stale data being used in hourly jobs.

#### Follow-up
questions might involve discussing how to handle failed tasks, managing state in complex pipelines, or integrating with other tools like Airflow for more robust job management.

**Comparison:**

```
Daily Model
Latest Daily Data
Hourly Model
```

**Code:**

```yaml
# Example dbt_project.yml
model:
  tags:
    - hourly
    - daily_parent

# model_name_daily.sql
{{ config(tag="daily_parent") }}

with base_table as (
  select * from {{ ref('raw_data') }}
)
select * from base_table

# model_name_hourly.sql
{{ config(tag="hourly", depends_on=['daily_model_latest']) }}

with latest_daily as (
  select * from {{ ref('daily_model', 'latest') }}
)
select * from latest_daily
```

**Example input / output:**

```text
- **Input**:
  - `model_name_daily.sql`: Raw data input.

- **Output**:
  - `model_name_daily.sql` output: Processed daily data.

- **Hourly Input**:
  - Latest daily model output (e.g., `daily_model_latest`).

- **Hourly Output**:
  - Processed hourly data, using the latest daily data as input.

### Example Workflow
1. Run `dbt run --select daily_model`.
2. Run `dbt run --select hourly_model`.

Ensure that hourly runs only depend on the latest daily output.
```

**Data:**

```json
{
  "raw_data": {
    "id": ["1", "2", "3"],
    "value": [10, 20, 30]
  }
}
```

**Flow:**

1. digraph DependencyFlow {
2. rankdir=LR;
3. node [shape=box, style=filled, color="#f0e68c"];
4. A[Daily Model] -> B[Latest Daily Data] -> C[Hourly Model]
5. }

