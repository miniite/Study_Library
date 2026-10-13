---
type: "practice"
difficulty: "Intermediate"
---

# How do you select which models to run using the --select flag in dbt?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Node Selection](<../../../../TOOLS/dbt/dbt_Core/Node_Selection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with dbt, selecting specific models to run can be a key part of your workflow, whether you're a fresher looking to understand basic commands or an architect focusing on optimizing the build process. The --select flag is used to specify which models in your dbt project should be executed during the build. This command helps streamline development and deployment by allowing you to target specific parts of your data pipeline.

To start, let's clarify what the question is probing: the --select flag allows us to run only a subset of our dbt models based on a pattern or specific model name(s). The interviewer might be looking for an understanding of how dbt works under the hood and how you can leverage this tool effectively.

Assume we have a large dbt project with many models spread across multiple directories. Let's reason through selecting models from first principles:

Understand the Model Structure: Typically, dbt models are organized into different projects (or packages) within the models directory. For instance, you might have models/mart/sales/sales_model.sql.

Pattern Matching with --select:

You can use patterns to select models. This is useful if your model names or directories follow a specific naming convention.

#### Example
dbt run --select mart.sales* would run all models under the mart/sales/ directory.

Specifying Individual Models:

If you want to target individual models, simply provide their full path.

#### Example
dbt run --select models/mart/sales/sales_model.sql

Using Yaml Configuration:

Alternatively, dbt allows specifying which models to select using the project’s YAML configuration file (e.g., models.yml).
You can configure multiple models or patterns in this file and then use them via the --select flag.

Practical Example: Suppose we have a scenario where we need to update only specific sales-related models:

SH
dbt run --select mart.sales*

Complex Scenarios:

If you're building an ETL pipeline that involves multiple stages, you might want to selectively run certain models.
For example, if you have mart/sales, mart/transformations, and mart/reports directories, running only the transformations for a quick check could be done as:
SH
dbt run --select mart.transformations/*

Combining Patterns:

You can also combine patterns to target multiple models or directories.

#### Example
dbt run --select mart.sales*,mart.promotions*

Output and Logs: When you use the --select flag, dbt will only log and execute the selected models, which can significantly reduce the amount of output in your logs.

**Code:**

```
SH
Copy
# Example command to run specific models using --select
dbt run --select mart.sales*
```

