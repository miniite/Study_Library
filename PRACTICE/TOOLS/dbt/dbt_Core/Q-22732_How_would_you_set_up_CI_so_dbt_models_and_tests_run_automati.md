---
type: "practice"
difficulty: "Deep"
---

# How would you set up CI so dbt models and tests run automatically on a pull request?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ dbt Models and Runs](<../../../../TOOLS/dbt/dbt_Core/dbt_Models_and_Runs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

CI/CD is crucial for ensuring data quality, consistency, and traceability in any ETL pipeline, especially when using tools like dbt. To set up CI so that dbt models and tests run automatically on a pull request (PR), we need to follow several steps from first principles.

Firstly, the question asks us to ensure that dbt models and tests are automatically executed during PRs. This means setting up an automated pipeline that triggers when a PR is created or updated, runs the relevant dbt commands, and reports back on any failures.

#### Triage

The key aspects we need to consider are:

Triggering Mechanism: When should CI run? On every PR event?
Execution Environment: Where will the execution happen? In a local development environment or in a remote CI/CD platform like GitHub Actions, GitLab CI, or Jenkins.
Dbt Commands: Which dbt commands are needed? run, test, and possibly seed for data loading.
Reporting Mechanism: How will we know if the PR failed? Through notifications, logs, or status updates.
Step-by-Step Solution
Choose a CI/CD Platform: For this example, let's use GitHub Actions due to its extensive support and ease of integration with Git repositories.
Define the Workflow File: This file will contain the instructions for running dbt commands in response to PR events.
Install Dependencies: Ensure that all necessary dependencies are installed on the CI environment.
Run Dbt Commands: Execute dbt run and dbt test to ensure models are correctly generated and tests pass.
Configure Notifications: Set up notifications (e.g., Slack, email) for failed builds.

Here’s an example of a GitHub Actions workflow file:

