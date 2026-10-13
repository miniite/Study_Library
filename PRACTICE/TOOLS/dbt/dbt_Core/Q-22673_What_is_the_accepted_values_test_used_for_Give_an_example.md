---
type: "practice"
difficulty: "Basic"
---

# What is the accepted_values test used for? Give an example.

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ dbt Tests](<../../../../TOOLS/dbt/dbt_Core/dbt_Tests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The accepted_values test in dbt (Data Build Tool) is a powerful validation mechanism that checks whether the values in a model conform to a predefined set of acceptable values. This ensures data integrity and consistency, which is crucial in maintaining high-quality datasets. Let's dive into how it works with an example.

**Code:**

```sql
-- Example table schema
CREATE TABLE users (
    id INT,
    status VARCHAR
);
```

**Example input / output:**

```

To illustrate the accepted_values test, let's define a model in dbt with this schema:

SQL
Copy
-- users_model.sql
{{ config(materialized='table') }}

SELECT * FROM source.users

-- Running the accepted_values test on status column
{% test accepted_values(model, column_name="status", values={"active", "inactive"}) %}
    SELECT *
    FROM {{ model }}
    WHERE {{ column_name }} NOT IN ({{ values }})
{% endtest %}

When you run this dbt model and the accepted_values test, it will check if any records have a status that is not either 'active' or 'inactive'. If there are such records, the test will fail and return an error.
```

**Data:**

```json
{
  "users": [
    {"id": 1, "status": "active"},
    {"id": 2, "status": "inactive"},
    {"id": 3, "status": "suspended"}
  ]
}
```

