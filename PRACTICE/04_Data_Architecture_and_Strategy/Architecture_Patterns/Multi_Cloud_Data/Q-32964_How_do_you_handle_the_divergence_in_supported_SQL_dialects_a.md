---
type: "practice"
difficulty: "Deep"
---

# How do you handle the divergence in supported SQL dialects and functions when the same transformation must run in BigQuery and Snowflake — do you abstract with dbt or accept forks?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cross-cloud Data Contracts](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Cross-cloud_Data_Contracts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a multi-cloud data architecture, ensuring that transformations can run consistently across different cloud platforms like BigQuery and Snowflake is crucial. The key challenge lies in their differing SQL dialects and functions. This problem can be approached using tools like dbt (data build tool) for abstraction or by accepting the necessity of forks. In this response, I will outline the considerations for both approaches, explain the first-principles reasoning behind each, and ultimately provide a recommendation.

Firstly, let's clarify what the question is really asking: How do you ensure consistency in SQL transformations across BigQuery and Snowflake, given their different dialects and functions? The interviewer is probing your understanding of multi-cloud strategies, data pipeline management, and abstraction techniques. We will assume that both platforms have specific features or limitations not supported by the other.

To tackle this problem from first principles:

Understand the Divergences: BigQuery uses its own SQL dialect with functions like DATE_TRUNC, while Snowflake supports a broader range of standard SQL functions but also has proprietary ones.
Define Transformation Requirements: Determine what transformations are necessary and where they differ between platforms.
Choose an Approach: Evaluate dbt or creating separate forks as solutions.

Let's walk through each approach step by step.

Using dbt for Abstraction

#### Triage

dbt is a powerful tool for managing data transformation in the cloud, enabling consistent SQL execution across different environments using its flexible templating and macro capabilities.
It supports big query and Snowflake out-of-the-box with minimal configuration changes.

#### First-principles Solution

Template Macros: Use dbt's macro feature to abstract platform-specific functions into reusable templates.

SQL
-- Example of a template for BigQuery-specific function
{% macro date_trunc(date_column, interval) %}
  DATE_TRUNC('{{ interval }}', {{ date_column }})
{% endmacro %}

-- Example of a similar macro for Snowflake
{% macro date_trunc(date_column, interval) %}
  TO_DATE_TRUNC('{{ interval }}', {{ date_column }})
{% endmacro %}

Profile-Specific Macros: Define profile-specific macros to handle platform differences.

YAML
# dbt_project.yml
profiles:
  default:
    outputs:
      bigquery:
        type: bigquery
        threads: 16
        schema: my_schema
        macro_path: "/path/to/macros"

      snowflake:
        type: snowflake
        threads: 16
        schema: my_schema
        macro_path: "/path/to/macros"

Run Tests: Utilize dbt's testing framework to ensure transformations work correctly across both environments.

Documentation and Maintenance: Maintain a clear documentation of these macros and the rationale behind them.

Accepting Forks

#### Triage

This approach involves maintaining separate codebases for BigQuery and Snowflake, which increases development and maintenance complexity but ensures full control over transformations.

#### First-principles Solution

Separate Code Repositories: Create two distinct repositories or branches in the same project.
GIT
# Repo structure example
/data-engineering/
  - bigquery/
    - macros/
      - date_trunc.sql
    - models/
      - my_model.sql
  - snowflake/
    - macros/
      - date_trunc.sql
    - models/
      - my_model.sql
Configuration Files: Use configuration files to toggle between environments.
CI/CD Pipelines: Set up separate CI/CD pipelines for each environment to test and deploy independently.

#### Closing Summary
and Common Mistakes

Using dbt offers a more maintainable solution by abstracting platform-specific differences into reusable templates, making the code cleaner and easier to manage. However, it requires a thorough understanding of both platforms' dialects and ongoing maintenance of these macros.

Accepting forks can lead to duplicated effort but ensures that the transformations are fully optimized for each environment. This approach is less scalable if the number of environments increases significantly.

#### Common mistakes
include over-engineering the abstraction layer in dbt or neglecting proper documentation, leading to maintenance issues down the line. In the case of forks, developers might not keep both branches up-to-date with changes.

#### Conclusion

In conclusion, using dbt for abstracting platform-specific differences is generally more efficient and maintainable than maintaining separate forks, especially when dealing with a growing number of environments. However, the choice depends on the specific project requirements, team expertise, and organizational policies.

**Code:**

```sql
-- Example of a template macro in dbt
{% macro date_trunc(date_column, interval) %}
  CASE
    WHEN target.name = 'bigquery' THEN DATE_TRUNC('{{ interval }}', {{ date_column }})
    WHEN target.name = 'snowflake' THEN TO_DATE_TRUNC('{{ interval }}', {{ date_column }})
  END
{% endmacro %}
```

