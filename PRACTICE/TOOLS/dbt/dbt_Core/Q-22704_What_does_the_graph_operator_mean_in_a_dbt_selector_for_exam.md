---
type: "practice"
difficulty: "Deep"
---

# What does the graph operator + mean in a dbt selector, for example my_model+ versus +my_model?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Node Selection](<../../../../TOOLS/dbt/dbt_Core/Node_Selection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In dbt selectors, the + symbol is used to include models or seeds that are referenced by other models in your project. The key difference between my_model+ and +my_model lies in how they are interpreted and their scope.

Firstly, let's break down what these selectors mean:

+my_model: This selector includes the model named my_model. It is a straightforward way to reference a specific model.

my_model+: This selector includes all models that directly or indirectly depend on my_model, along with my_model itself.

#### Triage

The question is asking about the nuances of dbt selectors, specifically focusing on the use of + in selectors. The key points to understand are:

What does + represent?
How does it affect model selection?

We will explore this step-by-step from first principles.

Step-by-Step Solution

Understanding dbt Selectors:

dbt uses a powerful and flexible selector system that allows you to target specific models, seeds, or tests in your project.
These selectors can be used in your dbt_project.yml file within the include directive.

Basic Selector Usage:

A simple model name like my_model will only include that exact model.

The + Operator:

The + operator is used to include models that depend on or are depended upon by a specified model.
For example, if you have a model named user_activity, and it depends on another model called session_data, using user_activity+ will also include session_data.
Detailed Example

Let's consider the following simple dbt project structure:

PLAINTEXT
models/
  - marts/
    - user_activity.sql
    - session_data.sql
    - daily_summary.sql

daily_summary.sql depends on both `user_activity` and `session_data`.
Using + in Selectors:

Selector: user_activity+:

YAML
include:
  - user_activity+

This will include user_activity, as well as any models that depend on it, such as daily_summary.

Selector: +user_activity:

YAML
include:
  +user_activity

This will also include user_activity and all models that are depended upon by user_activity, which is the same behavior as using user_activity+.

#### Closing Summary

In summary, the + operator in dbt selectors serves to include both a model and its dependent or dependency chain. Whether you use my_model+ or +my_model, they essentially do the same thing: they expand to include all models that have a direct or indirect relationship with my_model.

#### Common Mistakes/Follow-Ups
Mistake: Using + without understanding its scope can lead to unintended model inclusions, potentially bloating your build.

#### Follow-Up
Question: How does the + operator interact with other selectors like -, and what are some best practices for using it?

**Code:**

```yaml
# Example dbt_project.yml inclusion
include:
  - user_activity+
```

