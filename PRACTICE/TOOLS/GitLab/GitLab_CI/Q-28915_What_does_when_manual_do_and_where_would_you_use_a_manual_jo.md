---
type: "practice"
difficulty: "Intermediate"
---

# What does 'when: manual' do and where would you use a manual job?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Job Fundamentals](<../../../../TOOLS/GitLab/GitLab_CI/Job_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In GitLab CI, when: manual is used to create jobs that can be triggered by the user manually from the UI or API. This gives developers more control over certain parts of their pipeline, allowing them to bypass automated steps and focus on specific tasks when needed. Here’s a breakdown of what it does and where you might use such a job:

#### Triaging the Question

The question is essentially asking us to explain how when: manual works in GitLab CI pipelines and provide scenarios where this feature would be beneficial.

#### First-Principles Solution

In GitLab CI, jobs can have different triggers based on their visibility. When you define a job with when: manual, it means the job will not run automatically as part of any pipeline stages or matrix configurations. Instead, these jobs need to be manually initiated either through the GitLab UI or by using specific API commands.

Detailed Explanation

When setting up your CI/CD pipeline in GitLab, you can define various stages and jobs within them. By default, jobs are triggered automatically based on certain events like pushes to branches, merges, etc. However, there might be times when you want a job to run only if explicitly requested by a developer or tester.

For example, consider a scenario where your pipeline has an automated test suite that runs every time code is pushed to a branch. There might also be a need for a manual quality assurance (QA) check before the final deployment. Here’s how when: manual can fit into this:

YAML
stages:
  - build
  - test
  - qa

build_job:
  stage: build
  script:
    - echo "Building project..."

test_job:
  stage: test
  when: always # Runs automatically based on push events
  script:
    - echo "Running tests..."

qa_check:
  stage: qa
  when: manual # Triggered manually by a user or through API
  script:
    - echo "Performing QA check..."

#### Common Mistakes
and Follow-Ups

A common mistake is to not properly understand the implications of when: manual. For example, if you mistakenly set up an important job as manual, it could lead to pipeline delays if those jobs are required for deployment or release. It’s crucial to use this feature judiciously.

Another follow-up might be asking about integrating such manual jobs with other CI/CD practices. For instance, how would you handle logging and notifications when a user triggers a manual job? Would you want to log the reason for triggering it, notify relevant team members, or both?

#### Closing Summary

In summary, when: manual in GitLab CI is used to create jobs that are not automatically triggered but require explicit user interaction. This feature is useful when certain steps need to be controlled by human operators rather than being automated. Proper use of this can enhance the flexibility and control over your pipeline.

**Code:**

```yaml
stages:
  - build
  - test
  - qa

build_job:
  stage: build
  script:
    - echo "Building project..."

test_job:
  stage: test
  when: always # Runs automatically based on push events
  script:
    - echo "Running tests..."

qa_check:
  stage: qa
  when: manual # Triggered manually by a user or through API
  script:
    - echo "Performing QA check..."
```

