---
type: "practice"
difficulty: "Intermediate"
---

# Why would you commit a schema.yml describing the expected columns and types of a source table so a pull request can enforce that the contract didn't change?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Conventional Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Conventional_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

Committing a schema.yml file in Git & GitHub is crucial to maintain consistency across development environments and catch potential schema changes early, ensuring data integrity and reducing integration issues during code reviews.

#### Triaging the Question

The core of this question revolves around why committing a schema.yml file that describes the expected columns and types of a source table is beneficial. Specifically, the interviewer is probing your understanding of version control best practices, the importance of schema contracts in data pipelines, and how these commits facilitate effective pull requests (PRs).

#### First-Principles Solution

Maintain Consistency Across Environments:

A schema.yml file ensures that all developers are working with a consistent understanding of the source table's structure.
This helps avoid bugs caused by discrepancies in how different teams interpret the schema.

Enforce Schema Contracts:

By committing this file, you establish a formal contract for the expected data format.
During code reviews and merges via pull requests (PRs), GitHub can automatically enforce that changes do not alter the existing schema without approval.

Facilitate Automated Testing:

The presence of a schema.yml allows integration with tools like DataPipeline or Meltano to run automated tests.
These tests ensure that the actual schema matches the expected schema, catching issues early in the development cycle.

Ease of Onboarding and Collaboration:

A well-defined schema reduces the learning curve for new team members.
It also makes collaboration smoother as everyone is aligned on what to expect from the source tables.

Traceability and Auditing:

Committing the schema.yml file provides a historical record of the schema changes, making it easier to audit and track modifications over time.

#### Closing Summary

Committing a schema.yml file in Git & GitHub is essential for maintaining data integrity, ensuring consistency across development environments, and facilitating automated testing. By doing so, you can enforce strict adherence to schema contracts through pull requests, reducing the risk of integration issues and improving overall team productivity.

#### Common Mistakes/Follow-Ups
Ignoring Schema Changes: Developers might overlook changes to the schema, leading to data inconsistencies.
Manual Verification: Overreliance on manual verification instead of automated checks can result in delays and human errors.
Overcomplication: Adding unnecessary complexity to the schema may lead to over-engineering and increased maintenance overhead.

During a follow-up discussion, you might be asked about specific tools or workflows that support these practices (e.g., using dbt for schema management) or how to handle complex schemas with multiple dependencies.

**Code:**

```yaml
# Example of a simple schema.yml file

tables:
  - name: customers
    columns:
      id: integer
      name: string
      email: string
```

**Flow:**

1. Schema Commit
2. PR Opened
3. Code Review
4. Approval / Revisions
5. Merge & Run Tests
6. Data Pipeline Runs
7. Onboarding Documentation
8. Audit History

