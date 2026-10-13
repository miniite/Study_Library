---
type: "practice"
difficulty: "Intermediate"
---

# How do you write a singular test in dbt, and where does the SQL file live?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ dbt Tests](<../../../../TOOLS/dbt/dbt_Core/dbt_Tests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A singular test in dbt is used to validate that your model meets certain criteria or produces expected results. This is essential for ensuring data integrity and consistency across multiple environments. The SQL file containing a single test lives within the tests directory of your dbt project, alongside your models.

To write a simple test in dbt, you create an SQL file under the tests directory that follows specific naming conventions. Let's walk through writing a singular test from first principles.

