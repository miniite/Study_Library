---
type: "practice"
difficulty: "Intermediate"
---

# What is a scheduled pipeline and how would you set one up to run nightly?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ GitLab CI Fundamentals](<../../../../TOOLS/GitLab/GitLab_CI/GitLab_CI_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR Summary

A scheduled pipeline in GitLab CI allows you to automate the execution of your build, test, and deployment processes at specific intervals. In this case, setting up a nightly pipeline ensures that your code is built and tested every day automatically. This setup helps in maintaining a consistent development workflow and identifying issues early.

#### Triaging the Question

The question is essentially asking how to configure GitLab CI to run a pipeline on a scheduled basis, specifically at night. The key points here are understanding what a scheduled pipeline is and knowing the steps required to set one up in GitLab CI.

Key Assumptions:
You have basic knowledge of GitLab.
Your project has a .gitlab-ci.yml file configured for existing pipelines.
You want to run these pipelines daily at a specific time, such as midnight.
Step-by-Step Solution from First Principles

To set up a scheduled pipeline in GitLab CI:

Understand the .gitlab-ci.yml File: This is where you define your jobs and how they should be triggered.
Add or Modify Job Definitions: You need to add a job that will run at a specific time.
Use Time-Based Triggers (Cron Jobs): GitLab CI supports cron-like expressions to schedule pipelines.
Step 1: Understand the .gitlab-ci.yml File

The .gitlab-ci.yml file contains jobs, each with a script, image, and other configurations. Here is an example of how it looks:

YAML
stages:
  - build
  - test

build-job:
  stage: build
  script:
    - echo "Building the project..."
Step 2: Add or Modify Job Definitions

You need to add a new job that will run at midnight. You can do this by using the schedule keyword in GitLab CI.

Here’s an example of how you might define a scheduled pipeline:

YAML
stages:
  - build
  - test

build-job:
  stage: build
  script:
    - echo "Building the project..."
  only:
    schedules:
      - cron: '0 0 * * *' # Runs at midnight UTC every day

test-job:
  stage: test
  script:
    - echo "Running tests..."
  only:
    schedules:
      - cron: '0 0 * * *'
Step 3: Use Time-Based Triggers (Cron Jobs)

The schedule keyword allows you to define a schedule using a cron-like expression. In the example above, '0 0 * * *' runs the job at midnight UTC every day.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, setting up a scheduled pipeline involves defining jobs in your .gitlab-ci.yml file and using the schedule keyword to specify when they should run. It’s crucial to consider time zones and ensure that the cron expression matches your desired schedule.

#### Common Mistakes

Forgetting to define the only: section for scheduling.
Using incorrect timezone expressions, leading to unexpected runs.
Not testing the scheduled job before going live.
Triaging Common Follow-Ups
Handling Different Time Zones: Ensure that the cron expression is adjusted according to your project’s time zone.
Error Handling and Notifications: Implement error handling mechanisms and notifications in case of failures.
Scalability and Performance Considerations: If multiple pipelines are running, consider the impact on resource usage.

