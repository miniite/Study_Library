---
type: "practice"
difficulty: "Intermediate"
---

# A dbt incremental model on Snowflake keeps doing full refreshes; how do you debug the is_incremental logic and merge behavior?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Snowflake Loading and Pipelines](<../../../../TOOLS/Snowflake/Snowflake_Platform/Snowflake_Loading_and_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a dbt incremental model that's performing full refreshes on Snowflake, it’s crucial to understand and debug the is_incremental logic and merge behavior. This involves checking how records are identified as new or updated and ensuring they're correctly merged into the target table. The key steps involve examining the model code, verifying data consistency, and potentially using Snowflake's features for debugging.

**Code:**

```sql
-- Example dbt incremental model with is_incremental logic
{% macro stage_data(source_model) %}
    {{ adapter.dispatch('stage_data', 'my_utils')()(source_model) }}
{% endmacro %}

{# Default implementation #}
{% macro default_stage_data(source_model) %}
    -- This function should handle the is_incremental logic and merge behavior.
    -- For simplicity, we'll assume it's a generic incremental model here
    select * from {{ source_model }}
{% endmacro %}

-- Custom implementation in a specific dbt model
{# Custom stage_data function for this model #}
{% macro custom_stage_data(source_model) %}
    -- Ensure the logic is correct to handle updates and inserts
    with staged_data as (
        select * from {{ source_model }}
    )
    , target as (
        select * from {{ this }}
    )
    , to_merge as (
        select
            id,
            case when t2.id is null then 1 else 0 end as is_new,
            coalesce(t1.a, t2.a) as a,
            coalesce(t1.b, t2.b) as b
        from staged_data t1
        full outer join target t2 on t1.id = t2.id
    )
    select * from to_merge
{% endmacro %}
```

**Example input / output:**

```text
| id | a           | b               |
|----|-------------|-----------------|
| 1  | old_value   | old_value       |
| 2  | new_value   | new_value       |

Merged Data:
| id | is_new | a           | b               |
|----|--------|-------------|-----------------|
| 1  | 0      | changed_value | new_value       |
| 2  | 1      | new_value    | new_value       |
```

**Data:**

```json
{
  "staged_data": [
    {"id": 1, "a": "old_value", "b": "old_value"},
    {"id": 2, "a": "new_value", "b": "new_value"}
  ],
  "target": [
    {"id": 1, "a": "changed_value", "b": "new_value"},
    {"id": 3, "a": "unchanged_value", "b": "unchanged_value"}
  ]
}
```

