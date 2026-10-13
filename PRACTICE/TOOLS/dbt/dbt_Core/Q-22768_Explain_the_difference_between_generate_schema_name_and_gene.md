---
type: "practice"
difficulty: "Deep"
---

# Explain the difference between generate_schema_name and generate_database_name overrides, and why teams override the default schema-prefixing behavior.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Node Selection](<../../../../TOOLS/dbt/dbt_Core/Node_Selection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To explain the differences between generate_schema_name and generate_database_name overrides in dbt, we'll start by understanding their core functionalities and then dive into the nuances of why teams might choose to override these default behaviors.

In dbt, both generate_schema_name and generate_database_name are powerful features that allow you to customize the naming conventions for your models. The primary difference lies in their scope: generate_schema_name is used to control the schema name within a given database, while generate_database_name influences how databases themselves are named.

#### Triage

The question asks us to explain these differences and why teams might override them. We need to consider:

What each function does.
How they interact with dbt's default behavior.
The implications of overriding these functions in terms of data management, security, and team practices.

#### First Principles Solution
generate_schema_name

This function allows you to control the schema name for your models within a database. By default, dbt uses the target profile’s schema setting as the schema name. However, there are scenarios where teams might want more flexibility or consistency in their schema naming conventions.

#### Example
override:

PYTHON
def generate_schema_name(self):
    return "custom_schema_{{ this.name }}"
generate_database_name

This function controls how databases themselves are named. By default, dbt uses the target profile’s database setting as the database name. Overriding this function can be useful for customizing naming conventions or ensuring that databases follow specific naming patterns.

#### Example
override:

PYTHON
def generate_database_name(self):
    return "custom_db_{{ project_name }}"
Full Solution and Closing Summary

Teams might override these functions to achieve several goals:

#### Cons
istency: Ensure a consistent schema naming convention across projects.
Security: Prevent sensitive data from being exposed by using more descriptive or obfuscated schema names.
Team Practices: Align the schema and database naming conventions with existing team standards.

For instance, consider an e-commerce company that wants to name its schemas based on product categories:

PYTHON
def generate_schema_name(self):
    return "product_{{ this.name }}"

This ensures that all models related to a specific product category are grouped under a relevant schema, enhancing data organization and security.

#### Common Mistakes/Follow-Ups
Misunderstanding Scope: Forgetting the difference between generate_schema_name and generate_database_name, leading to incorrect implementations.
Overcomplicating Naming Conventions: Creating overly complex or nested naming patterns that can be harder to manage and query.
Neglecting Performance Considerations: Improperly overriding these functions might affect performance, especially if the overrides involve heavy computations.

For example, an overly complex generate_schema_name function:

PYTHON
def generate_schema_name(self):
    return "complex_{{ this.name | replace(' ', '_') | lower }}"

While this may seem fine at first glance, it could potentially impact execution time and readability of the generated code.

**Code:**

```python
# Example override for generate_schema_name
def generate_schema_name(self):
    return "custom_schema_{{ this.name }}"

# Example override for generate_database_name
def generate_database_name(self):
    return "custom_db_{{ project_name }}"
```

