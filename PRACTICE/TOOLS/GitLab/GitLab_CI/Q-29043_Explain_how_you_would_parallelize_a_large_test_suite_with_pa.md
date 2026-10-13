---
type: "practice"
difficulty: "Deep"
---

# Explain how you would parallelize a large test suite with parallel and split test files evenly across shards.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Pipeline Rules](<../../../../TOOLS/GitLab/GitLab_CI/Pipeline_Rules.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When parallelizing a large test suite in GitLab CI, the goal is to reduce the overall execution time by distributing tests across multiple runners. This involves splitting the test files into smaller chunks and running each chunk on different runner instances concurrently. The key challenge is ensuring that the tests are split evenly without causing race conditions or data conflicts.

To achieve this, we need to follow these steps:

Understand the Test Suite: Identify how many files contain tests, their dependencies, and any shared resources.
Determine Runner Capabilities: Know the number of available runners and their capabilities (e.g., CPU, memory).
Split Tests Evenly: Divide test files into a manageable number of shards based on runner availability.
Configure GitLab CI Pipeline: Use GitLab CI's parallel execution features to run tests in parallel.

**Code:**

```yaml
# .gitlab-ci.yml
stages:
  - test

test-job:
  stage: test
  script:
    - ./run_tests.sh $SHARD_ID $TOTAL_SHARDS
  parallel:
    matrix:
      - SHARD_ID: [0, 1, 2, 3] # Assuming we have 4 shards (can be configured based on total shard count)
```

**Example input / output:**

```text
# Example output of run_tests.sh script

## Shard 0:
Running tests in /path/to/tests/shard_0/

## Shard 1:
Running tests in /path/to/tests/shard_1/

## Shard 2:
Running tests in /path/to/tests/shard_2/

## Shard 3:
Running tests in /path/to/tests/shard_3/
```

**Flow:**

1. Start
2. Identify Tests
3. Determine Runners
4. Split Tests into Shards
5. Configure CI
6. Test Execution
7. End

