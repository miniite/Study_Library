---
type: "practice"
difficulty: "Intermediate"
---

# When would you reach for dbt-expectations or a custom generic test over the four built-ins, and give an example of a business rule you encoded as a test.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ dbt Tests](<../../../../TOOLS/dbt/dbt_Core/dbt_Tests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to decide when to use dbt-expectations or a custom generic test compared to the four built-in tests provided by dbt. The key factors to consider are the complexity of the business rule, the data distribution, and whether domain-specific knowledge is required.

#### Triage

The question asks for a scenario where we should prefer either dbt-expectations or a custom generic test over the built-in tests in dbt. We need to understand that:

Built-in Tests: These are simple and cover common use cases, like checking the number of rows, basic data type checks, and value comparisons.
dbt-expectations: This tool allows for more complex business logic validation using a declarative approach.
Custom Generic Test: A custom solution that can be tailored to specific needs but may require more development effort.

#### First Principles Solution

Scenario Identification:

When Business Logic is Complex: If the business rule involves multiple conditions, joins, or complex transformations, a custom generic test might be necessary.
When Domain Knowledge is Required: If there are specific domain requirements that aren't covered by built-in tests, dbt-expectations can help formalize these rules.

#### Example Scenario
Suppose we have an e-commerce platform and need to ensure that the total revenue in a given period matches the sum of individual product sales revenues. This rule is complex because it involves joins between multiple tables (orders, products) and aggregation over time.

Detailed Explanation

Let's walk through the steps involved:

Identify Business Rule:

The business rule states that the total revenue for a given period should equal the sum of individual product sales revenues.

Modeling the Rule with dbt-expectations:

SQL
{% test check_total_revenue_matches_individual_sales(revenue_model, orders_model, products_model) %}
  SELECT
    SUM(order_subtotal)
  FROM
    {{ ref('orders_model') }}
  WHERE
    order_date >= '2023-01-01'
=>
  SELECT
    SUM(revenue)
  FROM
    {{ ref('revenue_model') }}
  WHERE
    date >= '2023-01-01' AND product_id IN (
      SELECT
        product_id
      FROM
        {{ ref('orders_model') }}
    )
{% endtest %}
This test compares the total revenue calculated from revenue_model with the sum of individual sales revenues from orders_model and products_model.

Custom Generic Test: If the business rule requires specific domain logic, we might write a custom generic test using dbt's macro capabilities:

SQL
{% macro check_total_revenue_matches_individual_sales(revenue_model, orders_model, products_model) %}
  {{ adapter.dispatch('check_total_revenue_matches_individual_sales', 'custom_tests')()(revenue_model, orders_model, products_model) }}
{% endmacro %}

{# Define the logic for this custom test #}
{% macro check_total_revenue_matches_individual_sales(revenue_model, orders_model, products_model) %}
  -- Custom implementation
  SELECT *
  FROM (
    SELECT
      SUM(order_subtotal)
    FROM
      {{ ref('orders_model') }}
    WHERE
      order_date >= '2023-01-01'
  ) AS total_revenue
  JOIN (
    SELECT
      SUM(revenue)
    FROM
      {{ ref('revenue_model') }}
    WHERE
      date >= '2023-01-01' AND product_id IN (
        SELECT
          product_id
        FROM
          {{ ref('orders_model') }}
      )
  ) AS individual_sales_revenue
WHERE
  total_revenue <> individual_sales_revenue
{% endmacro %}

#### Closing Summary
and Common Mistakes/Follow-Ups
Mistake: Overusing dbt-expectations for simple business rules, leading to unnecessary complexity.

#### Follow-Up
Discuss how to prioritize between built-in tests, dbt-expectations, and custom generic tests based on the specific requirements of your project.
Code Block
SQL
-- Example dbt-expectations test
{% test check_total_revenue_matches_individual_sales(revenue_model, orders_model, products_model) %}
  SELECT
    SUM(order_subtotal)
  FROM
    {{ ref('orders_model') }}
  WHERE
    order_date >= '2023-01-01'
=>
  SELECT
    SUM(revenue)
  FROM
    {{ ref('revenue_model') }}
  WHERE
    date >= '2023-01-01' AND product_id IN (
      SELECT
        product_id
      FROM
        {{ ref('orders_model') }}
    )
{% endtest %}

-- Custom generic test implementation
{% macro check_total_revenue_matches_individual_sales(revenue_model, orders_model, products_model) %}
  -- Custom logic here
  SELECT *
  FROM (
    SELECT
      SUM(order_subtotal)
    FROM
      {{ ref('orders_model') }}
    WHERE
      order_date >= '2023-01-01'
  ) AS total_revenue
  JOIN (
    SELECT
      SUM(revenue)
    FROM
      {{ ref('revenue_model') }}
    WHERE
      date >= '2023-01-01' AND product_id IN (
        SELECT
          product_id
        FROM
          {{ ref('orders_model') }}
      )
  ) AS individual_sales_revenue
WHERE
  total_revenue <> individual_sales_revenue
{% endmacro %}
Self-check

Correctness + completeness confirmed.

**Code:**

```sql
-- Example dbt-expectations test
{% test check_total_revenue_matches_individual_sales(revenue_model, orders_model, products_model) %}
  SELECT
    SUM(order_subtotal)
  FROM
    {{ ref('orders_model') }}
  WHERE
    order_date >= '2023-01-01'
=>
  SELECT
    SUM(revenue)
  FROM
    {{ ref('revenue_model') }}
  WHERE
    date >= '2023-01-01' AND product_id IN (
      SELECT
        product_id
      FROM
        {{ ref('orders_model') }}
    )
{% endtest %}

-- Custom generic test implementation
{% macro check_total_revenue_matches_individual_sales(revenue_model, orders_model, products_model) %}
  -- Custom logic here
  SELECT *
  FROM (
    SELECT
      SUM(order_subtotal)
    FROM
      {{ ref('orders_model') }}
    WHERE
      order_date >= '2023-01-01'
  ) AS total_revenue
  JOIN (
    SELECT
      SUM(revenue)
    FROM
      {{ ref('revenue_model') }}
    WHERE
      date >= '2023-01-01' AND product_id IN (
        SELECT
          product_id
        FROM
          {{ ref('orders_model') }}
      )
  ) AS individual_sales_revenue
WHERE
  total_revenue <> individual_sales_revenue
{% endmacro %}
```

