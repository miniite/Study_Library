---
type: "practice"
difficulty: "Intermediate"
---

# How do you use vars and env_var together to parameterize a run for backfill date ranges without editing model code?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ dbt Fundamentals](<../../../../TOOLS/dbt/dbt_Core/dbt_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To parameterize a run in dbt for backfill date ranges using vars and env_vars, you can leverage these features to dynamically set your start and end dates, ensuring that you don't need to modify model code each time you want to perform a backfill. Here’s how you can do it:

First, understand the core idea: vars allow you to define variables within your project's configuration files (e.g., dbt_project.yml), while env_vars let you set environment-specific values via shell commands or an environment file.

#### Triaging the Question

The question is asking how to use vars and env_vars together in a dbt project to manage backfill date ranges without editing model code. The interviewer might be probing your understanding of:

How vars and env_vars work within dbt.
Best practices for maintaining flexibility and separation of concerns.
Potential pitfalls or common mistakes.

#### First-Principles Solution

Define Vars in dbt_project.yml:

YAML
vars:
  backfill_start_date: "2023-09-01"
  backfill_end_date: "2023-09-30"

Use the vars in your model code:

SQL
-- models/my_model.sql
{{ config(materialized='table') }}

select *
from source_table
where date >= '{{ var("backfill_start_date") }}'
  and date <= '{{ var("backfill_end_date") }}';

Set env_vars via CLI or Environment File: You can set these values either through the command line or in an environment file.

Using a shell:

SH
dbt run --vars "{'backfill_start_date': '2023-10-01', 'backfill_end_date': '2023-10-31'}"

Or in an environment file (.env):

CODE
BACKFILL_START_DATE=2023-10-01
BACKFILL_END_DATE=2023-10-31

Override with run-prompt: If you want to manually override the values, use:

SH
dbt run --vars "{'backfill_start_date': 'manual_start', 'backfill_end_date': 'manual_end'}"

#### Closing Summary
and Common Mistakes/Follow-Ups
Common Mistake: Forgetting to define vars in your project's configuration file.

#### Follow-Up
Discuss how this approach scales with multiple models and projects, and consider using a single env file for all projects.

By following these steps, you can dynamically control backfill ranges without modifying model code, making maintenance easier and reducing the risk of human error.

**Code:**

```
dbt_project.yml

vars: backfill_start_date: "2023-09-01" backfill_end_date: "2023-09-30"

CODE
Copy

```sql
-- models/my_model.sql
{{ config(materialized='table') }}

select *
from source_table
where date >= '{{ var("backfill_start_date") }}'
  and date <= '{{ var("backfill_end_date") }}';
```

**Example input / output:**

```text
# Example CLI command to run the model with specific backfill dates
dbt run --vars "{'backfill_start_date': '2023-10-01', 'backfill_end_date': '2023-10-31'}"
```

**Flow:**

1. Define vars in dbt_project.yml
2. Use in model code
3. Run with --vars or .env file
4. Automatically backfillsspecified date range

