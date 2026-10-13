---
type: "practice"
difficulty: "Deep"
---

# How do you use rules:changes to run only the affected service pipeline in a monorepo and avoid stale skips?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Pipeline Rules](<../../../../TOOLS/GitLab/GitLab_CI/Pipeline_Rules.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with GitLab CI, particularly in large monorepos, it's essential to minimize unnecessary pipeline runs to save time and resources. The rules:changes mechanism allows you to define specific conditions under which pipelines should run based on file changes. In this context, the goal is to ensure that only the affected service or component of a monorepo runs its CI pipeline, preventing stale skips.

#### Triage

The question is probing how to leverage GitLab's rules:changes feature to create targeted and efficient CI pipelines within a monorepo setup. This involves understanding file structure, defining precise rules, and ensuring that only relevant services are triggered based on changes made to specific files or directories. Key assumptions include familiarity with GitLab's repository organization and the ability to map out pipeline dependencies.

#### First-Principles Solution
Understand Monorepo Structure: Identify how your monorepo is structured—whether it’s a monolithic setup or if services are split into separate projects.
Define Service Boundaries: Clearly delineate each service or component within the monorepo to understand which parts of the project can be isolated and run independently.
Configure rules:changes:
Use patterns in your .gitlab-ci.yml files to define which changes trigger specific jobs.
Ensure that only the relevant pipeline is triggered when a change occurs.
Step-by-Step Solution

Identify Service Boundaries: Assume you have services named service-a, service-b, and service-c. Each service has its own directory structure within the monorepo.

Configure .gitlab-ci.yml for Each Service:

YAML
# .gitlab-ci.yml (example)
stages:
  - build
  - test

service-a-build:
  stage: build
  script:
    - echo "Building service-a"
  rules:
    - if: $CI_COMMIT_PATH == "/service-a/"
      when: always

service-b-test:
  stage: test
  script:
    - echo "Running tests for service-b"
  rules:
    - if: $CI_COMMIT_PATH == "/service-b/"
      when: on_success
      artifacts:
        paths:
          - target/

Use variables to Simplify Configuration: Define variables in the project settings or .gitlab-ci.yml to simplify paths and conditions.

YAML
variables:
  SERVICE_A_PATH: /service-a/
  SERVICE_B_PATH: /service-b/

service-a-build:
  stage: build
  script:
    - echo "Building service-a"
  rules:
    - if: $CI_COMMIT_PATH == "$SERVICE_A_PATH"
      when: always

service-b-test:
  stage: test
  script:
    - echo "Running tests for service-b"
  rules:
    - if: $CI_COMMIT_PATH == "$SERVICE_B_PATH"
      when: on_success
      artifacts:
        paths:
          - target/

#### Closing Summary
and Common Mistakes/Follow-Ups

By carefully configuring rules:changes in GitLab CI, you can ensure that only the affected services run their pipelines based on file changes. This approach minimizes unnecessary pipeline runs, saves resources, and ensures a more responsive CI/CD process.

#### Common Mistakes

Failing to properly define service boundaries.
Overly complex rules leading to misconfigurations.
Not considering edge cases where multiple services might be affected by the same change.

#### Follow-Ups

How would you handle scenarios with shared dependencies between services?
What strategies can you use to ensure that stale skips are minimal and that all services remain up-to-date?

**Code:**

```yaml
# Example .gitlab-ci.yml snippet
stages:
  - build
  - test

service-a-build:
  stage: build
  script:
    - echo "Building service-a"
  rules:
    - if: $CI_COMMIT_PATH == "/service-a/"
      when: always

service-b-test:
  stage: test
  script:
    - echo "Running tests for service-b"
  rules:
    - if: $CI_COMMIT_PATH == "/service-b/"
      when: on_success
      artifacts:
        paths:
          - target/
```

