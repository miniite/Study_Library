---
type: "practice"
difficulty: "Expert"
---

# Explain the gotcha where rules:changes behaves differently on merge request pipelines versus branch pipelines.

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Pipeline Rules](<../../../../TOOLS/GitLab/GitLab_CI/Pipeline_Rules.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In GitLab CI, the rules:changes directive is a powerful feature that allows you to define conditions based on changes made in your repository. However, there's an important gotcha when using this directive in merge request (MR) pipelines versus branch pipelines. This difference can lead to unexpected behavior and confusion if not properly understood.

#### Triage

The question asks for the specifics of how rules:changes behaves differently between MR pipelines and branch pipelines in GitLab CI. The core idea is that certain conditions might evaluate incorrectly or unexpectedly depending on the context (whether it's a regular commit or an MR).

#### First-Principles Solution

To understand this gotcha, we need to look at how rules:changes works under different scenarios:

Branch Pipelines: These are triggered when changes are pushed directly to a branch.
Merge Request Pipelines: These are triggered when changes are merged into the target branch through an MR.
Detailed Explanation
Branch Pipelines

In branch pipelines, rules:changes is straightforward:

If you push changes to a specific branch, GitLab CI checks if those changes match the defined rules:changes.
For example, if your rule specifies changes: "package.json", and you change package.json, the job will run.
Merge Request Pipelines

Merge request pipelines behave differently:

When a merge request is created, only the diff (the set of changes) between the source branch and the target branch is sent to GitLab CI.
This means that rules:changes in MR pipelines will not see the full state of the repository as it would in branch pipelines. Instead, they see the differences introduced by the merge request.
The Gotcha

The key difference here is that the changes seen by rules:changes are relative to the target branch:

Scenario 1: If you push a change directly to the target branch and then create an MR from another branch, GitLab CI will still run jobs based on the full state of the target branch.
Scenario 2: If you have an open MR that introduces changes but hasn't been merged yet, rules:changes in MR pipelines will only see those introduced changes. Therefore, your rule might not trigger as expected if it relies on a specific file or configuration existing in both branches.

#### Example

#### Cons
ider the following GitLab CI configuration:

YAML
stages:
  - build

job1:
  stage: build
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
      when: always
      changes: "README.md"

In this example, if you push a change to README.md in an MR that targets the main branch, GitLab CI will not run job1. This is because rules:changes only sees the difference between the source and target branches of the MR.

#### Closing Summary

This gotcha is critical for understanding how rules:changes works in different contexts. In branch pipelines, it checks against the full state of the repository at the time of the push. However, in MR pipelines, it only sees the changes introduced by the merge request relative to the target branch. This can lead to unexpected behavior if your rules rely on a specific file existing in both branches.

#### Common Mistakes/Follow-Ups
Misunderstanding Context: Assuming rules:changes works identically in MR and branch pipelines.
Overly Complex Rules: Creating overly complex rules that depend on the full state of the repository rather than just the changes introduced by an MR.

Understanding this gotcha can help you design your GitLab CI configuration more effectively, ensuring it behaves as expected across different scenarios.

**Code:**

```yaml
# Example GitLab CI configuration

stages:
  - build

job1:
  stage: build
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
      when: always
      changes: "README.md"
```

