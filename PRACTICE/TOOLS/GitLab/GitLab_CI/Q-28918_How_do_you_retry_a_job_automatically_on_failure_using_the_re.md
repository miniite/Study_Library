---
type: "practice"
difficulty: "Intermediate"
---

# How do you retry a job automatically on failure using the 'retry' keyword?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Debugging Runner Failures](<../../../../TOOLS/GitLab/GitLab_CI/Debugging_Runner_Failures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a GitLab CI job is retried when it fails, we can use the retry keyword within the .gitlab-ci.yml configuration file. This approach automates retries based on specific failure criteria, making our pipeline more robust and resilient to transient errors.

#### Triaging the Question

The core of this question revolves around understanding how GitLab CI handles job failures and the mechanism for automatically retrying those jobs. The interviewer is probing your knowledge of GitLab CI configuration syntax and your ability to handle job dependencies and retries effectively. We will assume that you have a basic understanding of GitLab CI pipelines.

Step-by-Step Solution
Understanding Retries in GitLab CI: In GitLab CI, the retry keyword allows us to specify how many times a job should be retried if it fails.
Configuring the retry Keyword:
The basic syntax for specifying retries is by adding the when: on_failure directive within the script or command section of your GitLab CI job configuration.
You can also specify a maximum number of retries using the max_attempts parameter.

#### Example
.gitlab-ci.yml
YAML
stages:
  - build

build_job:
  stage: build
  script:
    - echo "Running the build script..."
    - if [ "$CI_COMMIT_REF_NAME" = "master" ]; then exit 1; fi # Introduce a failure on master branch for testing
  retry:
    max_attempts: 3
    when: always  # Always retry, not just on failure

#### Closing Summary
+ Common Mistakes/Follow-Ups

In the example above, we have configured a job named build_job to run as part of the build stage. The script contains an intentional failure condition for testing purposes. We use the retry keyword within the script section to specify that this job should be retried up to three times. By setting when: always, we ensure the job is retried regardless of its previous run status.

#### Common Mistakes
Failing to Specify Retries: Not using the retry keyword at all, leading to manual intervention when a job fails.
Misconfiguring Retries: Forgetting to set the correct number of attempts or not specifying when to retry (e.g., only on failure).

#### Follow-Ups

The interviewer might ask about handling more complex scenarios, such as retrying based on specific error messages or integrating with custom scripts to determine whether a job should be retried. They could also inquire about best practices for managing retries in CI/CD pipelines.

