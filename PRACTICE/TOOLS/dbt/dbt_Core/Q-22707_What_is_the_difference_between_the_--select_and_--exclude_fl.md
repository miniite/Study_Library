---
type: "practice"
difficulty: "Basic"
---

# What is the difference between the --select and --exclude flags?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Node Selection](<../../../../TOOLS/dbt/dbt_Core/Node_Selection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: The --select and --exclude flags in dbt are used to control which models get compiled during a dbt run. While --select specifies the exact models you want to compile, --exclude helps avoid compiling models that match certain patterns or criteria.

To dive deeper into this question, let's first understand what these flags do and how they interact with each other in the context of dbt (data build tool).

#### Triage

The question is asking about two different ways to control which models get compiled when running a dbt project. The --select flag allows you to specify exactly which models should be included, whereas the --exclude flag lets you exclude certain models based on patterns or conditions.

#### First-Principles Solution

Understanding Models in dbt:

In dbt, each model is a piece of code that defines a specific data transformation.
By default, all models in your project will be compiled and deployed unless explicitly controlled by flags like --select or --exclude.

The --select Flag:

The --select flag allows you to specify which models should be included in the compilation process. You can use this flag with a specific model name, a pattern that matches multiple model names, or even a file path if your models are organized into files.
For example:
SHELL
dbt run --select my_model
This command will only compile and deploy the my_model specified.

The --exclude Flag:

The --exclude flag is used to exclude certain models from being compiled based on patterns or names.
For instance, if you have a large number of models but want to avoid compiling those related to test data:
SHELL
dbt run --exclude testing/*
This command will skip any model that matches the testing/* pattern.

Combining Flags:

You can use both flags together in one command to refine your compilation process. For example, you might want to compile only certain models and exclude others:
SHELL
dbt run --select my_model other_model --exclude testing/*

#### Closing Summary
+ Common Mistakes/Follow-Ups

Common Misunderstandings:

Sometimes beginners might confuse --select with specifying all the models they want to compile, when in fact it is used for explicitly choosing which ones.
Similarly, --exclude can be misunderstood as a blanket exclusion without understanding its powerful pattern-matching capabilities.

#### Follow-Ups

Ask about scenarios where these flags might be particularly useful, such as during feature toggling or testing different model configurations in development.
Explore how these flags interact with other dbt commands like seed, snapshot, and docs.

#### Example
Input->Output

Suppose you have the following models in your project:

my_model.sql
test_model.sql
customer_metrics.sql

Running:

SHELL
dbt run --select my_model --exclude testing/*

This will compile and deploy only my_model.sql, excluding any model that matches the testing/* pattern.

**Comparison:**

| Model           | Status         |
|-----------------|----------------|
| my_model.sql    | Compiled       |
| test_model.sql  | Not compiled   |
| customer_metrics.sql | Not compiled  |

**Code:**

```shell
# Example dbt command with --select and --exclude flags
dbt run --select my_model --exclude testing/*
```

**Example input / output:**

```text
# Command
dbt run --select my_model --exclude testing/*

# Output (assuming no errors)
Running dbt version 1.0.0
Compiling my_model.sql

+--my_model.sql
```

**Data:**

```json
{
  "models": [
    {"name": "my_model.sql"},
    {"name": "test_model.sql"},
    {"name": "customer_metrics.sql"}
  ]
}
```

**Flow:**

1. Run dbt with --select and--exclude flags
2. (dbt compiles models)
3. (dbt excludes models based onpatterns)
4. Compiles my_model.sql
5. Excludes test_model.sql,customer_metrics.sql

