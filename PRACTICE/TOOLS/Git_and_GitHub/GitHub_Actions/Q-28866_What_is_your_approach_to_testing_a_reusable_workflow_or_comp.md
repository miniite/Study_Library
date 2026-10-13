---
type: "practice"
difficulty: "Deep"
---

# What is your approach to testing a reusable workflow or composite action itself before shipping it to 40 consuming repos?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Pinning Third-party Actions](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Pinning_Third-party_Actions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with GitHub Actions, especially in scenarios where you have reusable workflows or composite actions that will be consumed by multiple repositories, it's crucial to ensure robust testing before deploying them widely. The approach I take involves several steps to cover various aspects of the workflow’s functionality and reliability.

Firstly, let me quickly summarize my strategy: we set up a dedicated test repository for each reusable action or workflow. This allows us to simulate real-world usage scenarios without affecting production environments. We then implement a series of checks and tests that cover both functional correctness and error handling.

**Comparison:**

```
Argument	Expected Value
arg1	value1
arg2	value2
Result	expected_value
```

**Code:**

```bash
# Example setup in the .github/workflows/test.yml file for a composite action

name: Test Composite Action

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '16'

      - name: Install dependencies
        run: npm install

      - name: Test the composite action
        id: test_action
        uses: ./path-to-action/composite-action
        with:
          arg1: value1
          arg2: value2

      - name: Assert output
        if: steps.test_action.outputs.result != 'expected_value'
        run: echo "Test failed, expected result was not met."
```

**Example input / output:**

```text
Given an input where `arg1` is set to `value1` and `arg2` is set to `value2`, the test workflow runs the composite action, expecting a specific output. If the actual result does not match `expected_value`, the step fails.
```

**Data:**

```yaml
# Example input for the test workflow
arguments:
  arg1: value1
  arg2: value2

expected_output:
  result: expected_value
```

**Flow:**

1. Yes
2. No
3. Checkout code
4. Set up Node.js
5. Install dependencies
6. Test the composite action
7. Result == 'expected_value'
8. PASS
9. FAIL, output error message

