---
type: "practice"
difficulty: "Deep"
---

# What is a reusable workflow (or shared pipeline template), and why would a platform team maintain one centrally?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Reusable Workflows](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reusable_Workflows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

A reusable workflow or shared pipeline template in Git and GitHub allows teams to define, version control, and share CI/CD configurations that can be reused across multiple projects. This approach promotes consistency, accelerates development cycles, and reduces the risk of errors by leveraging predefined best practices.

#### Triaging the Question

The question is essentially asking about the concept and importance of reusable workflows or shared pipeline templates in a DevOps context. Specifically, it probes why maintaining such templates centrally makes sense for a platform team. We need to understand that centralizing these templates ensures uniformity across projects, simplifies maintenance, and enhances collaboration.

#### Assumptions
The platform team is responsible for managing CI/CD pipelines.
Reusable workflows are defined using YAML files (GitHub Actions, Jenkinsfile, etc.).
These templates will be shared among multiple projects or teams within the organization.
Step-by-Step Solution:

Defining a Workflow Template:

A reusable workflow template is essentially a pre-defined set of steps that define how code changes should be tested and deployed.
For example, it might include steps like linting, unit testing, integration testing, building the application, deploying to staging, and finally to production.

Centralizing Templates:

Centralizing these templates means storing them in a shared repository or directory where multiple teams can access and use them.
This could be done via GitHub repositories, internal GitLab instances, or any other version control system that supports shared workflows.

Benefits of Centralization:

#### Cons
istency: Ensures that all projects follow the same best practices for testing, deployment, security configurations, etc.
Maintenance: Easier to update and maintain configurations in one place rather than scattered across multiple project repositories.
Collaboration: Teams can contribute to and improve shared templates over time, fostering a collaborative environment.
Scalability: As the organization grows, centrally managed templates are easier to scale without introducing inconsistencies.

#### Implementation
Example:

A platform team might maintain a shared-pipelines repository in GitHub where various reusable workflow files reside (e.g., lint.yml, test.yml, deploy.yml).
These files can be included or referenced directly from project repositories, ensuring that the latest best practices are always used.

#### Example
Workflow Template:

YAML
name: Lint and Test

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.8'
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
      - name: Lint code
        run: |
          flake8 .

  test:
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v2
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
      - name: Run tests
        run: |
          pytest .

#### Closing Summary
and Common Mistakes/Follow-Ups:

#### Closing Summary
Centralizing reusable workflows in Git and GitHub ensures consistency, simplifies maintenance, and promotes collaboration within a platform team. This approach is crucial for organizations that want to standardize their CI/CD processes.

#### Common Mistakes
Not versioning templates properly, leading to unexpected changes during updates.
Failing to document the templates well, making it difficult for new team members to understand and use them correctly.
Overcomplicating the templates by including too many unnecessary steps or features.

#### Follow-Ups
How does one handle conflicts when multiple teams need to modify a shared template?
What are some best practices for testing and validating reusable workflow templates before deploying them across projects?

**Code:**

```yaml
name: Lint and Test

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.8'
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
      - name: Lint code
        run: |
          flake8 .

  test:
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v2
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
      - name: Run tests
        run: |
          pytest .
```

