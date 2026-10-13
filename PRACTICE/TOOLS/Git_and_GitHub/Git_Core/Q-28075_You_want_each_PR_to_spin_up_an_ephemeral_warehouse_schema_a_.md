---
type: "practice"
difficulty: "Deep"
---

# You want each PR to spin up an ephemeral warehouse schema (a dbt clone-based dev schema) and tear it down on close, but leaked schemas are piling up and costing storage. How would you tie schema lifecycle to Git PR events reliably so nothing leaks?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Isolating CI Environments](<../../../../TOOLS/Git_and_GitHub/Git_Core/Isolating_CI_Environments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that each pull request (PR) in a Git repository reliably spins up an ephemeral warehouse schema using dbt clone and tears it down upon completion, we need to tie the schema lifecycle to the PR events. This approach not only ensures that no schemas leak but also optimizes storage usage by automatically cleaning up unused resources.

The key idea is to integrate a CI/CD pipeline with Git operations through event triggers, specifically leveraging GitHub webhooks for triggering schema creation and deletion scripts during PR creation and closure. Here’s how we can achieve this:

Understand the Question: The core of the question revolves around automating the lifecycle management of database schemas using Git-based pull requests. We need to ensure that each PR spins up a new, isolated environment (schema) for development purposes and then destroys it once the PR is closed or merged.

#### Assumptions

You have a GitHub repository with dbt models.
You use CI/CD tools like Jenkins, GitLab CI, or GitHub Actions to manage your workflows.
The team has basic familiarity with dbt and schema management in your database environment.

#### First-Principles Solution
To tie the schema lifecycle to PR events reliably:

GitHub Webhook Integration: Set up a webhook for push and pull_request events on your GitHub repository.
CI/CD Pipeline Triggers: Configure your CI/CD pipeline (e.g., GitHub Actions) to trigger schema creation when a PR is opened and destruction when the PR is closed or merged.
Schema Management Scripting: Create shell scripts or Python functions using dbt commands to handle schema creation and deletion.
Step-by-Step Solution

GitHub Webhook Setup: Configure GitHub to send webhooks for specific events, such as push and pull_request. This can be done in the repository settings under "Webhooks".

Event Triggers:

When a PR is opened (pull_request.opened event), trigger a script or workflow to spin up a new dbt clone schema.
When a PR is closed, merged, or abandoned (pull_request.closed, push if the branch associated with the PR is deleted/merged), trigger a script or workflow to delete the schema.

Schema Creation and Destruction Script:

BASH
# Example shell script for creating and destroying schemas

# Create Schema
dbt clone <source-model-path> --target dev
dbt run -s <model-names>

# Destroy Schema
dbt deps
dbt clean

GitHub Actions Workflow:

YAML
name: DBT CI

on:
  push:
    branches:
      - main  # or any other relevant branch
  pull_request:
    actions: [opened, closed]

jobs:
  create-schema:
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request' && github.event.action == 'opened'
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Setup Python
        uses: actions/setup-python@v2
        with:
          python-version: 3.8
      - name: Install dependencies
        run: |
          pip install dbt-core
      - name: Create DBT clone schema
        run: |
          dbt clone <source-model-path> --target dev
          dbt run -s <model-names>

  destroy-schema:
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request' && (github.event.action == 'closed' || github.event.action == 'merged')
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Setup Python
        uses: actions/setup-python@v2
        with:
          python-version: 3.8
      - name: Install dependencies
        run: |
          pip install dbt-core
      - name: Destroy DBT clone schema
        run: |
          dbt deps
          dbt clean

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Closing Summary
By integrating GitHub webhooks with a CI/CD pipeline, we can reliably manage the lifecycle of ephemeral schemas for each PR, ensuring no schemas leak and storage costs are optimized. This approach leverages automation to streamline database management practices.

#### Common Mistakes

Failing to properly configure the webhook events.
Not setting up event triggers within your CI/CD pipeline.
Misconfiguring the dbt commands for schema creation or deletion.

#### Follow-Ups
How do you handle edge cases where a PR is abandoned but never closed?
What additional checks could be implemented to ensure data consistency during schema teardown?

