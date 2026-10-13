---
type: "practice"
difficulty: "Basic"
---

# What does the command dbt run actually do?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ dbt Fundamentals](<../../../../TOOLS/dbt/dbt_Core/dbt_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: The dbt run command in dbt (data build tool) compiles and executes your models, generating the final transformed data that aligns with the business logic defined in your schema.yml files.

Question Breakdown: The question is essentially asking about the purpose and outcome of using the dbt run command. The interviewer might be probing to understand the flow of data from raw sources through transformations to a structured format, as well as how dbt ensures that this process is repeatable and reliable.

Step-by-Step Solution:

Understanding dbt's Role: dbt is an open-source library for writing, testing, and deploying reusable transformation logic in SQL. It simplifies the process of managing data by allowing you to define transformations declaratively.

Command Breakdown:

dbt run specifically runs all models defined within a project.
Models are essentially SQL queries that take raw data from sources (like databases or files) and transform it into structured, consistent datasets.

Flow of Execution:

dbt first compiles the SQL code in your models according to the dependencies specified in your schema.yml files.
It then executes these compiled SQL statements against the database.
The results are stored in a new or existing table, typically in a data warehouse, and are ready for downstream analysis.

Key Concepts:

Models: These are essentially named groups of SQL code that define your transformations.
Sources: These are raw data sources from which you start your transformation process.
Dependents: Models can depend on other models or sources, ensuring the execution order is respected.

#### Example
Workflow:

Raw Data Source
Model 1
Model 2
Terminate Table

Outcome: The dbt run command ensures that all models are executed in the correct order, producing a final dataset that can be used for reporting and analysis.

#### Closing Summary
In summary, dbt run is a powerful tool that streamlines data transformation processes by compiling and executing SQL transformations in an ordered manner. It allows data engineers to maintain clear, reusable code while ensuring that their transformations are applied consistently across runs.

#### Common Mistakes/Follow-Ups

For fresher-level candidates, common mistakes include not understanding the dependencies between models or assuming all transformations happen in a single step.
For mid-level candidates, it's important to discuss how dbt handles versioning and testing of models.
For senior candidates, consider discussing scale considerations, performance optimization techniques, and best practices for managing large-scale data pipelines.

**Code:**

```yaml
# Example schema.yml file
models:
  my_project:
    my_model_1:
      name: "my_model_1"
      description: "First transformation of raw data"
      depends_on:
        - source.my_source.table_a

    my_model_2:
      name: "my_model_2"
      description: "Second transformation based on model 1 results"
      depends_on:
        - my_model_1
```

**Data:**

```yaml
# Example source.yml file
sources:
  - name: my_source
    tables:
      - name: table_a
        description: Raw customer data
```

