---
type: "practice"
difficulty: "Deep"
---

# What is the 'workflow' keyword and how does it control whether a pipeline is created at all?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Job Fundamentals](<../../../../TOOLS/GitLab/GitLab_CI/Job_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary TL;DR: The workflow keyword in GitLab CI is used to define branching strategies for your pipelines, ensuring that certain jobs or branches only run under specific conditions. This feature helps manage complex pipeline configurations and can be crucial in maintaining the integrity of your development processes.

#### Triage
The question revolves around understanding how the workflow keyword works within GitLab CI pipelines. Specifically, it probes how this keyword influences whether a pipeline is created at all based on certain logic. We need to assume that you are familiar with basic GitLab CI concepts and have some experience setting up and managing pipelines.

Step-by-Step Solution:

Understanding the workflow Keyword: The workflow keyword in GitLab CI allows you to define branching strategies for your pipelines, which dictate whether a pipeline should be executed based on specific conditions. This is particularly useful when you have complex workflows with multiple branches or stages that need conditional execution.

Basic Syntax:

YAML
workflow:
  rules:
    - if: '$CI_COMMIT_BRANCH == "master"'
      when: always
    - if: '$CI_PIPELINE_SOURCE == "push" && $CI_COMMIT_REF_NAME =~ /^feature-/'
      when: manual

Explanation of the Rules:

The rules clause under workflow allows you to specify conditions that determine whether a pipeline should run.
In the first rule, if the commit is made on the master branch ($CI_COMMIT_BRANCH == "master"), the pipeline will always run (when: always). This ensures that changes in the main development line are always tested and merged properly.
The second rule applies when a push event occurs on branches that match the pattern feature-*/. In this case, the pipeline runs only if someone manually triggers it (when: manual), which is ideal for feature branches where you want to avoid automatic execution.

Common Use Cases and Benefits:

Branch Filtering: Using patterns like ^feature-/*, you can control whether pipelines should run on specific branches, such as excluding temporary or auxiliary branches.
Manual Interventions: By setting certain jobs to require manual triggers, you ensure that critical changes are reviewed thoroughly before being merged into the main codebase.

#### Common Mistakes
and Follow-Ups:

Incorrect Patterns: Failing to use appropriate branch patterns can lead to unintended pipeline executions.
Overly Complex Rules: Overcomplicated rules might make it difficult for others to understand your pipeline configuration, so clarity is key.

#### Performance Considerations
While manual triggers are good for manual steps, overusing them can slow down the CI/CD process.

#### Closing Summary
The workflow keyword in GitLab CI provides a powerful way to control whether pipelines run based on specific conditions. By leveraging its rules and patterns, you can tailor your CI processes to meet complex requirements efficiently.

#### Common Mistakes/Follow-Ups

Ensure that branch patterns are correctly defined to avoid unintended pipeline executions.
Use clear and concise rules to make the configuration easy for others to understand.
Balance between manual interventions and automation to optimize your workflow.

