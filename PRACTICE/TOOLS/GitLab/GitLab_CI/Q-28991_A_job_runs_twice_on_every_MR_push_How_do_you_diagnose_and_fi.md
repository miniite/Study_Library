---
type: "practice"
difficulty: "Intermediate"
---

# A job runs twice on every MR push. How do you diagnose and fix the duplicate pipeline?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Pipeline Rules](<../../../../TOOLS/GitLab/GitLab_CI/Pipeline_Rules.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a GitLab CI issue where jobs run twice on each merge request (MR) push, it's crucial to systematically diagnose and resolve the problem. This scenario can be caused by multiple factors related to how your GitLab Runner is configured or how you've set up your pipeline. Let's walk through this from first principles.

First, we need to understand what triggers the pipelines in GitLab CI. Typically, each push to a merge request (MR) will trigger a new pipeline. If a job runs twice, it could be due to one of two main reasons: either the runner is configured incorrectly or there's an issue with how the stages are defined in your pipeline.

To diagnose this, follow these steps:

Check the Pipeline Stages: Ensure that each stage is correctly defined and not inadvertently running multiple times.
Review Runner Configuration: Verify that a single runner is not being used by both pipelines or incorrectly configured to run jobs more than once.
Pipeline YAML File: Inspect your .gitlab-ci.yml file for any potential misconfigurations.

Now, let's go through each step in detail:

Step-by-Step Diagnosis and Fix

Review Your Pipeline YAML File:

Open your .gitlab-ci.yml file.
Look at the stages to ensure they are not duplicating jobs. For example, if you have two before_script sections or multiple script lines with identical steps, this could cause duplication.

Check for Runner Multiplexing:

If your runners are multiplexed (i.e., shared across projects), it's possible that the same runner is being used by both pipelines, leading to duplicate job runs.
Run the command gitlab-runner list in the runner environment to check if the same runner is indeed being reused.

Pipeline Stages:

Ensure each stage and job has a unique purpose. For example, you might have two separate stages for tests (e.g., unit tests and integration tests) or for different types of deployments.
Avoid redundant jobs by ensuring that your pipeline definition is as minimal as possible while still covering all necessary steps.

Run Specific Commands to Debug:

Use gitlab-runner stop followed by gitlab-runner start to restart the runner and see if the issue persists.
Check GitLab's logs using tail -f /var/log/gitlab/gitlab-rails/production.log for any errors or warnings related to runners.

#### Example
of a Simple .gitlab-ci.yml File

Here’s an example of a simple CI file that ensures no duplicate jobs:

YAML
stages:
  - build
  - test

build_job:
  stage: build
  script:
    - echo "Building the project"

test_job:
  stage: test
  script:
    - echo "Running tests"

#### Common Mistakes
and Follow-Ups
Mistake: Failing to ensure that runner names or tags are correctly configured.

#### Follow-Up
Investigating if there's a race condition in your pipeline where jobs might be triggered multiple times unintentionally.

By methodically going through these steps, you should be able to identify the root cause and fix the issue. If the problem persists after following these diagnostics, consider reaching out for more detailed logs or consulting with other team members who may have insights into specific configurations or workarounds.

**Code:**

```yaml
stages:
  - build
  - test

build_job:
  stage: build
  script:
    - echo "Building the project"

test_job:
  stage: test
  script:
    - echo "Running tests"
```

