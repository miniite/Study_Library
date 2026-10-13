---
type: "practice"
difficulty: "Intermediate"
---

# Your ETL PR bundled a connector refactor, a new dbt model, and a partition-key fix in one commit, and reverting the bad partition change would also drop the model. Explain concretely why this hurts revertability during a data incident and how you'd coach the restructure.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Atomic Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Atomic_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In an ETL pipeline where we've bundled multiple significant changes into one commit—such as a connector refactor, a new dbt model, and a partition-key fix—we risk making the codebase less manageable and harder to revert if something goes wrong. This can have serious implications during a data incident. Let's break down why this is problematic and how you could coach someone to improve their approach.

First, let’s triage the issue. The core problem here is that when we bundle multiple changes into one commit, it makes it difficult to isolate which specific change caused an issue. This can lead to a time-consuming and error-prone process of debugging and reverting if a problem arises. Additionally, having a single commit that introduces significant changes can complicate the review and merge process.

Assuming you are working with Git and GitHub, we need to ensure each change is atomic, meaning it encapsulates only one aspect of functionality. This improves readability, maintainability, and makes it easier to revert specific changes if necessary.

Step-by-Step Solution

Identify Individual Changes: Break down the bundled commit into smaller, more focused commits.

For example:
Commit 1: Refactor the connector logic.
Commit 2: Introduce a new dbt model.
Commit 3: Fix the partition key.

Commit Messages: Ensure each commit has a clear and descriptive message that explains what was changed. This helps other team members understand the change without diving into the code.

Code Reviews: Use GitHub’s pull request (PR) system to review each change independently.

This allows team members to provide feedback on each individual part of the pipeline, ensuring all changes are thoroughly vetted.

Testing: Integrate automated testing and continuous integration (CI) pipelines to catch issues early.

Running tests for each commit can help identify any regressions or new bugs introduced by the change.

Documentation: Update relevant documentation if necessary, ensuring that other team members know about the changes and their impact on the pipeline.

#### Example
Workflow

Let's assume we are working with a simple ETL pipeline using dbt for modeling. The initial commit (bundle) is as follows:

BASH
git commit -m "Refactor connector, add new model, fix partition key"

To break this down, you would create three separate commits:

Connector Refactor:

DIFF
# Before
def process_data(connector):
    return connector.read()

# After
def process_data_v2(connector):
    transformed_data = transform_data(connector.read())
    return filter_and_clean(transformed_data)

git commit -m "Refactor data processing logic"

New dbt Model:

SQL
{% macro create_model() %}
  with source as (
    select * from {{ ref('raw_source') }}
  )
  , transformed as (
    select * from source
    -- Add transformations here
  )
  , final as (
    select * from transformed
  )
  select * from final
{% endmacro %}

git commit -m "Add new dbt model for enhanced data transformation"

Partition Key Fix:

SQL
with source as (select * from {{ ref('raw_source') }} limit 1000)
  , transformed as (
    select * from source
    -- Add transformations here
  )
  , final as (
    select * from transformed
  )
  , partitioned_data as (
    select * from final
    where date >= '2023-01-01'
  )
select * from partitioned_data

git commit -m "Fix partition key in dbt model"

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, bundling multiple significant changes into one commit can make the codebase harder to manage and debug. By breaking down large changes into smaller, more focused commits, we improve revertability and maintainability. Additionally, using Git effectively for version control, integrating automated testing, and maintaining clear documentation are crucial practices.

#### Common mistakes
include:

Failing to review individual commits before merging.
Not documenting the changes adequately.
Ignoring the importance of code readability and maintainability.

#### Follow-up
questions could explore more about the CI/CD pipeline setup, the specifics of dbt models, or how to handle complex refactorings in a controlled manner.

