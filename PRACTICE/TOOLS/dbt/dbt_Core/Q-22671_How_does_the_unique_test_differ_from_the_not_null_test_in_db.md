---
type: "practice"
difficulty: "Basic"
---

# How does the unique test differ from the not_null test in dbt?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ dbt Tests](<../../../../TOOLS/dbt/dbt_Core/dbt_Tests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In dbt, a data transformation and analytics framework, unique and not_null are two types of model tests used to ensure data integrity during your ETL (Extract, Transform, Load) processes. A unique test verifies that a column contains only unique values, while a not_null test ensures that a column does not contain any null values.

To understand the difference between these two tests and how they can be applied in dbt, let's break down their functionality from first principles.

#### Triage
What is being Probed?

The question is probing your understanding of two important model validation methods in dbt. These tests are critical for ensuring data quality and consistency before further analysis or reporting. The key aspects the interviewer might be interested in include:

The definitions and purposes of unique and not_null tests.
How these tests differ from each other.
Situations where you would use one over the other.
Step-by-Step Solution

Understanding the unique Test:

A unique test checks whether a column contains only unique values, meaning no two rows in that column should have the same value.
For example, an ID or SKU column might be expected to contain unique values because each record should have its own identifier.

Understanding the not_null Test:

A not_null test ensures that a column contains non-null values. This means every row in the table must have a value for this column.
For example, if you have a customer ID column, it’s essential to ensure every customer has an ID assigned.

Difference Between unique and not_null:

The primary difference lies in their focus: unique is about the distinctness of values within a column, while not_null is about ensuring each row has a value for that column.
A column can fail both tests. For example, if you have duplicate IDs and some IDs are null, your model will report failures on both unique and not_null checks.

#### Example Scenario

Let's consider an example of a sales dataset:

Order ID: This should be not_null because every order must have an identifier.
Customer Name: While this can be not_null, it doesn’t need to be unique. A customer might place multiple orders, so duplicate names are acceptable.

#### Closing Summary
and Common Mistakes
Common Mistake 1: Overlooking both tests in your model validation process can lead to data quality issues that go unnoticed.
Common Mistake 2: Failing to understand the specific needs of each column. For instance, while Order ID must be unique (to avoid duplicate orders), a simple name column should allow duplicates.

By understanding these differences and applying appropriate tests, you ensure your database is both consistent and accurate for further analysis.

Common Follow-Ups
The interviewer might ask about other types of model validation in dbt or how to handle exceptions when these tests fail.
They might also inquire about integrating these tests into a larger data quality strategy.

**Code:**

```sql
-- Example of using unique and not_null tests in dbt

{% test unique.test_unique(model, column_name) %}
    select
        {column_name}
    from {{ model }}
    group by 1
    having count(*) > 1
{% endtest %}

{% test not_null.test_not_null(model, column_name) %}
    select
        {{ adapter.quote(column_name) }} as value
    from {{ model }}
    where {{ adapter.quote(column_name) }} is null
{% endtest %}
```

