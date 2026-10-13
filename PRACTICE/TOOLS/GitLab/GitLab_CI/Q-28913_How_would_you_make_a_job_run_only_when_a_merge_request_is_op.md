---
type: "practice"
difficulty: "Intermediate"
---

# How would you make a job run only when a merge request is opened?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Pipeline Rules](<../../../../TOOLS/GitLab/GitLab_CI/Pipeline_Rules.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When setting up GitLab CI to run jobs based on events, one common requirement is to trigger specific jobs only when certain actions are taken, such as opening a merge request (MR). To achieve this, we need to configure the .gitlab-ci.yml file correctly and use GitLab's event system effectively. This involves understanding how GitLab handles different CI events and leveraging appropriate conditions in our pipeline configuration.

#### Triage

The question asks us to set up a job that runs only when a merge request is opened. Here, we need to:

Understand the different types of events that can trigger jobs.
Configure the .gitlab-ci.yml file to recognize and respond to merge_request_event.
Ensure that other events (like push or tag) do not accidentally run our specific job.

#### First-Principles Solution

To make a job run only when a merge request is opened, we need to define the job in the .gitlab-ci.yml file with appropriate conditions. GitLab provides first-class support for this via event-based triggers. Let's break down the process step by step:

Identify the Job: Define the specific job that needs to be triggered only on a merge request.
Configure Conditions: Use only or when keywords to specify that the job should run only when a merge request is created.
Test Locally (Optional): Optionally, you can test your .gitlab-ci.yml configuration locally using GitLab's CI/CD tools.
Detailed Steps

Identify the Job: Let’s assume we have a job named test_merge_request.

Configure Conditions:

Use the only keyword to specify that this job should run only when a merge request is created.
YAML
test_merge_request:
  stage: test
  script:
    - echo "Running tests for merge request"
  only:
    - merge_requests

Test Locally: If you're using GitLab's CI/CD tools locally, ensure that the .gitlab-ci.yml configuration works as expected without accidental triggers from other events.

#### Closing Summary
and Common Mistakes/Follow-Ups

By configuring our pipeline correctly, we can ensure that the test_merge_request job runs only when a merge request is opened. Some common mistakes to avoid include:

Incorrectly setting up event conditions.
Overlooking local testing of CI/CD configurations.
Confusing other events (like push) with merge requests.

If there are follow-ups, such as handling specific branches or customizing the job further, we can extend our .gitlab-ci.yml file accordingly. For example:

YAML
test_merge_request:
  stage: test
  script:
    - echo "Running tests for merge request"
  only:
    - merge_requests
    - changes_in_branches: [feature/*]

This ensures that the job runs specifically when a merge request is opened from branches prefixed with feature/.

**Code:**

```yaml
test_merge_request:
  stage: test
  script:
    - echo "Running tests for merge request"
  only:
    - merge_requests
```

**Flow:**

1. Define Job
2. Set Conditions in .gitlab-ci.yml
3. Configure Only MergeRequests
4. Test Locally
5. Deploy and Monitor

