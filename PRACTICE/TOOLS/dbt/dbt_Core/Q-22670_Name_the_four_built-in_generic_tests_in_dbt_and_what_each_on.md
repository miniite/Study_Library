---
type: "practice"
difficulty: "Basic"
---

# Name the four built-in generic tests in dbt and what each one checks.

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ dbt Tests](<../../../../TOOLS/dbt/dbt_Core/dbt_Tests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In dbt, there are four built-in generic tests that help ensure data quality and consistency across your models. These tests are fundamental tools for anyone working with dbt to maintain reliable data pipelines. Let's dive into each one:

unique: This test ensures that a column or set of columns contains no duplicate values.
not_null: It checks that at least one non-null value exists in the specified columns.
accepted_values: This test verifies that a column’s values are within a list of allowed values, ensuring data integrity.
boolean: It ensures that all values in a given column are boolean (true or false), maintaining consistency for boolean fields.

Each of these tests serves a specific purpose and helps catch common data quality issues early on in the development process. For example, unique can help prevent duplicate records, while not_null ensures that critical columns aren't empty.

**Comparison:**

```
Test	Description
unique	Ensures no duplicate values in the specified column(s)
not_null	Ensures at least one non-null value exists in the specified columns
accepted_values	Verifies that a column's values are within a list of allowed values
boolean	Ensures all values in a given column are true or false
```

**Example input / output:**

```

There are no specific input-output examples to provide for these built-in tests as they are defined in the dbt configuration files rather than run with explicit inputs. However, you can test them in your models like this:

SQL
Copy
-- Example usage of unique and not_null

{% test unique_test(model='my_table', column_name='customer_id') %}
    SELECT
        COUNT(*) - COUNT(DISTINCT customer_id) AS num_duplicates
    FROM
        {{ ref('my_table') }}
{% endtest %}

{% test not_null_test(model='my_table', column_name='name') %}
    SELECT
        SUM(CASE WHEN name IS NULL THEN 1 ELSE 0 END) AS null_count
    FROM
        {{ ref('my_table') }}
{% endtest %}
```

**Flow:**

1. Yes
2. No
3. No
4. Yes
5. No
6. Yes
7. No
8. Yes
9. unique
10. Is there duplicate data?
11. Report duplicates
12. Pass test
13. not_null
14. Are all values non-null?
15. Report nulls
16. Pass test
17. accepted_values
18. Is the value in accepted set?
19. Report failure
20. Pass test
21. boolean
22. Are all values true or false?
23. Report invalid values
24. Pass test

