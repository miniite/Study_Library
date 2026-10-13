---
type: "practice"
difficulty: "Deep"
---

# What is 'interruptible: true' and how does it interact with newer commits?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Pipeline Rules](<../../../../TOOLS/GitLab/GitLab_CI/Pipeline_Rules.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In GitLab CI, interruptible: true is a feature that allows pipelines to be interrupted and resumed at any stage when new commits are pushed during their execution. This can significantly reduce pipeline downtime and improve efficiency, especially in large development teams where frequent commits might cause ongoing pipelines to block other work.

#### Triaging the Question

The question probes your understanding of GitLab CI's advanced features for managing continuous integration workflows. Specifically, it asks you to explain interruptible: true and its interaction with new commits. Key assumptions include familiarity with basic GitLab CI concepts such as pipeline stages and how they handle new changes.

#### First-Principles Solution
Understanding Interruptible Pipelines: When a pipeline is set with interruptible: true, it means that if the pipeline execution is interrupted (e.g., by new commits), GitLab will attempt to resume the pipeline from where it left off. This can be particularly useful in scenarios where multiple developers are committing changes frequently.
Interaction with New Commits: When a commit occurs while an ongoing pipeline stage is running, GitLab identifies this as a conflict and attempts to re-execute only the stages affected by the new commit. The goal is to ensure that newer code is tested without fully rerunning all previous stages, which can be time-consuming.
Stage-Level Interruptibility: interruptible: true applies on a stage-by-stage basis. If a pipeline has multiple stages and one of them encounters an interruption, only that specific stage will be re-run or paused depending on the situation.
Detailed Explanation

#### Cons
ider a GitLab CI configuration with two stages: lint-check and test. When the pipeline is running, if a new commit introduces changes in the test files while the lint-check stage is still executing, the lint-check will complete. However, once it encounters the test-related changes, the entire pipeline might need to be paused or re-run only from the point of the change.

YAML
stages:
  - lint-check
  - test

variables:
  interruptible: true

lint-check:
  script:
    - echo "Linting code..."
    - npm run lint

test:
  script:
    - echo "Running tests..."
    - npm run test

#### Closing Summary
and Common Mistakes/Follow-Ups

Understanding interruptible: true is crucial for managing efficient CI workflows in large projects. A common mistake might be assuming that all stages will rerun when a new commit happens, leading to unnecessary delays. Instead, GitLab intelligently handles interruptions by resuming from the affected stage.

#### Follow-ups
could include discussing other GitLab CI features like parallelization or caching strategies and how they interact with interruptible pipelines for optimal performance.

**Code:**

```yaml
stages:
  - lint-check
  - test

variables:
  interruptible: true

lint-check:
  script:
    - echo "Linting code..."
    - npm run lint

test:
  script:
    - echo "Running tests..."
    - npm run test
```

**Flow:**

1. Yes
2. No
3. Start
4. Lint Check Stage
5. New Commit?
6. Test Stage from Change Point
7. Complete Pipeline
8. Complete Test Stage and Finish

