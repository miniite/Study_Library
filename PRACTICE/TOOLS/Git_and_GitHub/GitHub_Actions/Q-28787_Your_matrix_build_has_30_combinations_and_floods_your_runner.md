---
type: "practice"
difficulty: "Intermediate"
---

# Your matrix build has 30 combinations and floods your runner concurrency. How do you use max-parallel and include/exclude to keep it sane?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Concurrency Control in Workflows](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Concurrency_Control_in_Workflows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To manage a matrix build with 30 combinations that are overwhelming your GitHub Actions runner concurrency, we can leverage max-parallel along with include and exclude parameters in our workflow configuration. This approach ensures that the builds run efficiently without exceeding the available resources.

Triaging the Problem

The question is essentially about optimizing resource usage by controlling the parallelism of jobs in a GitHub Actions matrix build. We need to ensure that the number of concurrent jobs does not exceed the runner's capacity, while still covering all necessary combinations.

#### Assumptions
You are using a GitHub Actions matrix strategy.
The runners have limited concurrency.
You want to maintain control over which combinations run and when.

#### Solution

To solve this problem, we can use the following strategies:

Limit Parallelism with max-parallel: This parameter controls how many jobs are allowed to be executed in parallel. By setting an appropriate value, you can prevent excessive resource usage.
Filter Combinations with include and exclude: These parameters allow you to specify which matrix combinations should run and which should not.
Step-by-Step Solution

Determine Concurrency Requirements:

First, assess the current runner concurrency limit (e.g., 5 or 10).
Decide on a reasonable number of concurrent jobs that will still ensure timely execution but won't overwhelm the runners.

Configure max-parallel:

YAML
jobs:
  build_matrix:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        combinations: [1, 2, ..., 30] # Adjust as needed based on your actual combinations
      max-parallel: <N> # Replace <N> with the determined number of concurrent jobs
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Run build step
        run: make build

Filter Combinations with include and exclude:

YAML
strategy:
  matrix:
    combinations: [1, 2, ..., 30] # Adjust as needed based on your actual combinations
    include:
      - {paramA: value1, paramB: value2}
      - {paramA: value3, paramB: value4} # Include specific combinations
  exclude:
    - {paramC: 'badValue'} # Exclude specific combinations

#### Example
Configuration

Let's assume we have 50 jobs in total but need to limit the concurrency to 10. Additionally, some combinations may be known to fail or are not necessary.

YAML
jobs:
  build_matrix:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        combinations: [1, 2, ..., 30] # Adjust as needed based on your actual combinations
      max-parallel: 10 # Limit concurrent jobs to 10
      include:
        - {paramA: value1, paramB: value2}
        - {paramA: value3, paramB: value4} # Include specific combinations
      exclude:
        - {paramC: 'badValue'} # Exclude specific combinations
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Run build step
        run: make build

#### Closing Summary
and Common Mistakes/Follow-Ups

By using max-parallel, you can control the number of concurrent jobs, ensuring that your runners are not overwhelmed. Additionally, leveraging include and exclude helps in precisely selecting which matrix combinations to include or exclude based on specific criteria.

#### Common Mistakes

Setting max-parallel too high, leading to resource overutilization.
Not properly filtering out unnecessary combinations with exclude.
Ignoring the runner's concurrency limits.

If you have more complex scenarios or need further optimization, consider additional strategies like job grouping, conditional steps, and using separate workflows for different parts of your build process.

**Code:**

```yaml
jobs:
  build_matrix:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        combinations: [1, 2, ..., 30]
      max-parallel: 10
      include:
        - {paramA: value1, paramB: value2}
        - {paramA: value3, paramB: value4}
      exclude:
        - {paramC: 'badValue'}
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Run build step
        run: make build
```

