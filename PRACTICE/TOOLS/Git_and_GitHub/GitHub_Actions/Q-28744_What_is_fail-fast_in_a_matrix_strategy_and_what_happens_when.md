---
type: "practice"
difficulty: "Deep"
---

# What is fail-fast in a matrix strategy and what happens when you set it to false?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Concurrency Control in Workflows](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Concurrency_Control_in_Workflows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Fail-fast in GitHub Actions' matrix strategy determines how tasks are processed based on conditions. When fail-fast is set to true (the default), as soon as any job fails, the entire matrix run is halted and no further jobs in that particular combination will be executed. Setting it to false means all jobs in a given combination will run even if some of them fail. This question delves into understanding how this setting affects your workflow and when you might want to change the default behavior.

**Code:**

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [12.x, 14.x]
        fail-fast: false # Set this to true for the default behavior
```

**Example input / output:**

```text
With `fail-fast` set to `false`, all combinations of node versions will be executed. For example:
- With `node-version: 12.x` and `fail-fast: false`, both jobs for 12.x will run even if one fails.
- Similarly, with `node-version: 14.x` and `fail-fast: false`, the corresponding jobs will also run regardless of previous failures.

This approach ensures that you get test results for all combinations before deciding on any action.
```

