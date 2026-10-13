---
type: "practice"
difficulty: "Basic"
---

# How does dbt build a dependency graph (DAG) between models?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ dbt Models and Runs](<../../../../TOOLS/dbt/dbt_Core/dbt_Models_and_Runs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this explanation, we'll delve into how dbt constructs a dependency graph (DAG) to manage the relationships between your database models. A DAG is essential because it helps dbt understand which models need recompilation when changes occur in underlying data or other models.

TL;DR: dbt uses a combination of explicit and implicit dependencies defined in your models to build a DAG, ensuring that only necessary transformations are run. The process involves parsing model definitions, identifying dependencies through depends_on, and resolving circular references if any exist.

#### Triage
The question asks about the mechanism dbt uses to create a dependency graph. This is important for understanding how dbt optimizes data pipelines by ensuring that models only re-run when necessary. Key points include explicit dependencies via the depends_on directive and implicit dependencies inferred from model relationships.

First Principles: Let's start with the basics. A DAG in dbt consists of nodes representing individual models or seeds, and edges indicating dependencies between them. When you run dbt, it first parses the DAG to determine which models need recompilation based on changes detected (e.g., new data in a source table).

Model Definitions: Each model in your dbt project is written as a SQL query or a Python script that references other models.
Explicit Dependencies via depends_on: You can explicitly define dependencies between models using the depends_on tag, which ensures that certain models are re-run only if their dependencies have been updated.
Implicit Dependencies: Implicit dependencies are inferred based on the logic within your model queries and how they reference other tables or views.

For example:

SQL
{{
  config(
    materialized='table',
    depends_on=[ref('source_table')]
  )
}}

with base as (
  select * from {{ ref('source_table') }}
)
select * from base

In this snippet, depends_on is used to specify that the model should be re-run if source_table has been updated.

Resolving Dependencies: The dbt engine parses these dependencies and builds a graph. If there are circular references or transitive dependencies, dbt will handle them gracefully by ensuring no cycles exist in the DAG. This is crucial to maintain data integrity and prevent infinite loops during recompilation.

#### Closing Summary
+ Common Mistakes/Follow-Ups: Understanding how dbt builds its dependency graph is critical for optimizing your ETL pipelines. Explicitly defining dependencies using depends_on can help speed up recompilation times by ensuring that only necessary models are run. However, it's essential to be mindful of implicit dependencies and circular references.

#### Common mistakes
include:

Overusing depends_on, which can lead to unnecessary recompilations.
Ignoring implicit dependencies, leading to incomplete model updates.

To further explore this topic, you might want to look into how dbt handles different types of sources (e.g., CSVs, APIs) and the impact of schema changes on model dependencies. Additionally, understanding how dbt integrates with version control systems can provide deeper insights into dependency management in data engineering projects.

**Comparison:**

```
Model	Depends On
model_a	source_table
model_b	model_a, source_table
```

**Code:**

```sql
-- Example of a model definition with explicit dependency
{{ config(materialized='table', depends_on=[ref('source_table')]) }}

with base as (
  select * from {{ ref('source_table') }}
)
select * from base
```

**Example input / output:**

```text
dbt run --select model_b

# Output: Dependencies identified, models recompiled as necessary.
```

**Data:**

```yaml
# Example dbt project schema.yml
version: 2

models:
  - name: model_a
    description: "Model that references a source table"
    columns:
      - name: id
        description: "Unique identifier for each record"

  - name: model_b
    description: "Model dependent on model_a and source_table"
    depends_on:
      - model_a
      - source_table

sources:
  - name: source_data
    tables:
      - name: source_table
```

**Flow:**

1. Source Table
2. Model A
3. Model B

