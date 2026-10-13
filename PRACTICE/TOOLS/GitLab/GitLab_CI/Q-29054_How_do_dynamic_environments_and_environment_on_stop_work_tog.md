---
type: "practice"
difficulty: "Deep"
---

# How do dynamic environments and environment:on_stop work together to clean up review apps automatically?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Protected Environments](<../../../../TOOLS/GitLab/GitLab_CI/Protected_Environments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how dynamic environments and environment:on_stop in GitLab CI/CD work together, let’s first define the key concepts and then delve into a step-by-step explanation of their interaction.

TL;DR

Dynamic environments in GitLab CI allow you to create temporary applications (review apps) for specific branches or pull requests. The environment:on_stop directive is used to automatically clean up these environments once they are no longer needed, ensuring that resources are freed and costs minimized.

#### Triaging the Question

The core of this question revolves around understanding how GitLab CI manages lifecycle events of dynamic environments created during a pipeline run. Specifically, we need to explore:

How dynamic environments are defined and utilized in a GitLab project.
The purpose and mechanics of the environment:on_stop directive.
The interaction between these two concepts when managing review apps.

#### Assumptions
GitLab Project Setup: A GitLab project configured with CI/CD pipelines, including support for dynamic environments.
Dynamic Environment Configuration: Properly defined .gitlab-ci.yml configurations that create and manage review apps based on pull requests or branches.
Review App Requirement: The need to automatically clean up these temporary applications when they are no longer required.

#### First-Principles Solution

Creating a Dynamic Environment:

When a new branch or pull request is created, GitLab can trigger the creation of a dynamic environment (review app) using specific CI/CD stages in your .gitlab-ci.yml file.

Defining the Stages:

YAML
stages:
  - review
  - cleanup

review_app:
  stage: review
  script:
    - # Commands to setup and deploy the app
  environment:
    name: $CI_COMMIT_REF_SLUG-review
    url: http://$REVIEW_APP_URL/$CI_PROJECT_NAME
  allow_failure: false
  when: on_push

Environment Definition:

The environment key in the stage definition specifies the details of the environment, such as its name and URL.
$CI_COMMIT_REF_SLUG-review dynamically sets the environment name based on the branch or pull request.

Using environment:on_stop:

To ensure that the review app is automatically cleaned up once it’s no longer needed (e.g., when a PR is merged, closed, or the branch is deleted), you use the on_stop key.
YAML
cleanup_review_app:
  stage: cleanup
  script:
    - # Commands to clean up the environment
  environment:
    name: $CI_COMMIT_REF_SLUG-review
    action: terminate

Stages Interaction:

The review_app stage creates the review app.
Once a specific condition is met (e.g., PR merged or closed), GitLab triggers the cleanup_review_app stage, which contains the environment:on_stop directive to clean up the environment.

Automatic Cleanup:

When the cleanup stage runs, it sends a termination request to the cloud provider hosting your dynamic environment.
This process ensures that resources are freed and costs associated with the review app are minimized.

#### Closing Summary

By leveraging GitLab CI’s dynamic environments and environment:on_stop, you can automate the lifecycle management of temporary applications like review apps. This approach not only enhances efficiency but also helps in maintaining cost control by ensuring that unused environments are cleaned up promptly.

#### Common Mistakes/Follow-Ups
Misconfiguration: Forgetting to define the environment name or URL correctly.
Incorrect Termination Logic: Not properly setting the action key within environment:on_stop.
Dependency Issues: Failing to include necessary cleanup commands in the cleanup_review_app stage.
Timeouts and Retries: Ensuring that the termination process handles timeouts and retries appropriately.

