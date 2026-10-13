---
type: "practice"
difficulty: "Basic"
---

# What is the difference between a scheduled trigger and a push trigger for a pipeline?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Pipeline Triggers and Path Filters](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Pipeline_Triggers_and_Path_Filters.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In DevOps, Continuous Integration (CI) and Continuous Deployment (CD) pipelines are crucial for ensuring that code changes are automatically tested and deployed in a reliable manner. The difference between scheduled triggers and push triggers lies in how they initiate the pipeline execution.

#### Triage
Understanding Triggers
Scheduled Trigger: This type of trigger initiates pipeline runs at predefined intervals, such as daily or weekly.
Push Trigger: This trigger starts the pipeline upon receiving changes to a source control repository.

#### First-Principles Solution

Scheduled Triggers:

These are set up to run at specific times and on specific days.
Useful for non-critical tasks that need periodic execution, like running scheduled reports or cleaning up old data.

#### Example
A pipeline configured to execute daily at midnight.

Push Triggers:

Triggered automatically when changes (commits) are pushed to a source control repository.
Ideal for integrating new code into the production environment quickly and efficiently.

#### Example
A pipeline that starts immediately after developers commit changes to their branch in Git.

#### Closing Summary
+ Common Mistakes/Follow-Ups:
Common Mistake: Using scheduled triggers instead of push triggers when you need immediate feedback on code changes. This can delay the release process.

#### Follow-Up
Consider using both types of triggers for a balanced pipeline: push triggers for fast, frequent deployments and scheduled triggers for tasks that should be done periodically.

#### Example Scenario

Suppose you are working on a web application where developers frequently push changes to fix bugs or add features. You might configure your CI/CD pipeline with a push trigger so it runs automatically whenever a developer pushes code. Additionally, you could set up a scheduled trigger to run nightly tests and clean up old logs.

**Comparison:**

```
Trigger Type	When It Runs	Use Case
Scheduled	At predefined intervals	Running periodic, non-critical tasks.
Push	On code push to repo	Immediate feedback on new changes.
```

**Code:**

```yaml
# Example YAML configuration for a CI/CD pipeline using Jenkins

pipeline:
  triggers:
    - schedule: 'H 0 * * *' # Daily at midnight
    - git: true # Trigger on every push to Git repository
  stages:
    - name: Build
      steps:
        - script: echo "Building the project"
    - name: Test
      steps:
        - script: echo "Running tests"
```

**Example input / output:**

```text
Input:
- New code pushed to the 'main' branch in Git.
Output:
Pipeline starts automatically and performs build, test, and deployment steps.

Input:
- Cron job triggers at midnight on a specific day.
Output:
Pipeline runs for nightly tasks such as report generation or log cleanup.
```

**Data:**

```json
{
  "triggers": [
    {
      "type": "schedule",
      "cronExpression": "H 0 * * *"
    },
    {
      "type": "git",
      "branchFilter": "main"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Code Pushed
4. Is it a scheduled time?
5. Run Pipeline (Build, Test,Deploy)
6. Scheduled Time Reached

