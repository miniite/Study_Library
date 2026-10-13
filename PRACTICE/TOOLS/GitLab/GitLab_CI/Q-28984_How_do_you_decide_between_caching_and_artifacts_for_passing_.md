---
type: "practice"
difficulty: "Deep"
---

# How do you decide between caching and artifacts for passing a dependency directory between jobs and what breaks if you pick wrong?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Cache vs Artifacts](<../../../../TOOLS/GitLab/GitLab_CI/Cache_vs_Artifacts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding whether to use caching or artifacts in GitLab CI, it's crucial to understand their respective strengths and weaknesses, as well as the specific requirements of your pipeline. Caching is typically used to speed up the build process by storing intermediate files that don't change frequently, whereas artifacts are used to pass data between jobs and persist the final results.

To decide which to use for passing a dependency directory between jobs, consider these steps:

Understand Your Workflow: Determine if your pipeline has multiple stages or jobs that need access to shared dependencies.
Evaluate Frequency of Changes: Assess how often the dependency directory changes. If it rarely changes, caching can significantly speed up subsequent builds by reusing cached data.

#### Cons
ider Data Size and Complexity: Evaluate whether passing a large or complex directory would be cumbersome or slow when using artifacts.

If you pick wrong, here’s what could break:

Caching Mistakes: Using caching for a dependency that frequently changes will result in slower builds as the cache is invalidated more often.
Artifact Mistakes: Passing a dependency through artifacts when it rarely changes can lead to unnecessary data transfers and slow down your pipeline.

Let's dive deeper into each scenario.

#### Triage
What the Question Is Really Asking

The question is probing your understanding of GitLab CI best practices, particularly around caching and artifact usage. You need to demonstrate how to make informed decisions based on the context of the dependency directory in question.

Step-by-Step Solution From First Principles

Assess Dependency Directory Changes:

If the directory contains commonly used libraries or tools that rarely change (e.g., a node_modules folder), caching is highly beneficial.
If the directory frequently changes, caching might be less effective and could lead to slower builds due to more frequent cache invalidation.

Evaluate Data Size and Complexity:

A large dependency directory can significantly slow down the build process if passed as an artifact. Consider whether the data size is manageable or if a smaller subset of dependencies would suffice.
If passing artifacts frequently, ensure that the pipeline can handle the additional load without performance degradation.

Use Caching for Stable Dependencies:

YAML
stages:
  - build

build-job:
  stage: build
  script:
    - npm install --cache-min 10080 # Example of caching node modules
  cache:
    key: $CI_COMMIT_REF_SLUG
    paths:
      - node_modules/

Use Artifacts for Unstable or Large Dependencies:

YAML
stages:
  - build
  - test

build-job:
  stage: build
  script:
    - npm install # No caching needed here if dependencies frequently change
  artifacts:
    paths:
      - node_modules/

test-job:
  stage: test
  script:
    - npm run test
  dependencies:
    artifacts: true

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, use caching for stable dependencies to speed up builds by reusing cached data. Use artifacts when you need to pass large or frequently changing directories between jobs, but be mindful of the performance impact.

#### Common Mistakes

Using caching on a dependency that changes too often.
Overusing artifacts and transferring unnecessary data.

#### Follow-Ups

Discuss potential optimizations like using more granular cache keys or artifact filtering.
Explore other advanced GitLab CI features like cache strategies, to further enhance performance.

