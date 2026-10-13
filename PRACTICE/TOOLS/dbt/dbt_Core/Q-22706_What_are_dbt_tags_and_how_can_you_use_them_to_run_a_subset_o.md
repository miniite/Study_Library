---
type: "practice"
difficulty: "Intermediate"
---

# What are dbt tags and how can you use them to run a subset of models?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Node Selection](<../../../../TOOLS/dbt/dbt_Core/Node_Selection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

DBT tags allow you to categorize your models based on various criteria, such as priority or domain. This feature is incredibly useful when you want to run a specific set of models without affecting the rest of your project. In this answer, I will explain what dbt tags are, how they work, and provide examples of their usage in different scenarios.

#### Triage
The question asks for an explanation of dbt tags and their application in running subsets of models. It is probing your understanding of dbt's tag system, its implementation, and practical use cases. Key assumptions include familiarity with dbt as a data transformation tool and the ability to manage model execution flow effectively.

Step-by-step solution from first principles:

Understanding Tags: In dbt, tags are custom labels that you can apply to your models or tests. These tags can be used to categorize your models based on various criteria, such as "high priority," "finance," or "daily refresh."
Tagging Models: To use tags, you first tag the models you want to manage. For instance, if you have a model named customers, you might tag it with @high_priority and @finance.
Running Tagged Models: Once your models are tagged, you can run only those tagged models using dbt commands. This is particularly useful for incremental development or when you want to test specific changes without running the entire project.

For example, if you have a model named customers with tags @high_priority and @finance, you can run these specifically by executing:

BASH
dbt run --select @high_priority @finance

#### Closing summary
DBT tags provide a flexible way to categorize your models based on various criteria. This system allows for targeted execution of specific models, making it easier to manage projects with many models and to prioritize certain workloads.

#### Common mistakes/follow-ups

Misunderstanding Tag Application: Always ensure you correctly apply tags to the appropriate models.
Overlooking Tag Dependencies: Ensure that any dependencies between tagged models are considered when running them.
Using Tags for Naming Conventions: While it is possible, using tags as a naming convention can lead to confusion and make maintenance harder. Tags should be used more for categorization rather than as part of the model name.

**Code:**

```python
# Example dbt model with tags
{{ config(materialized='table', tags=['high_priority', 'finance']) }}

select *
from customers
```

**Example input / output:**

```text
# Running specific tags
dbt run --select @high_priority @finance

# Expected output: Only models with the specified tags are executed.
```

