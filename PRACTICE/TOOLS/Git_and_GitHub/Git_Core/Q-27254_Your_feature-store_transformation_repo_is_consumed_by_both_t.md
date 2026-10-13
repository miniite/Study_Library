---
type: "practice"
difficulty: "Deep"
---

# Your feature-store transformation repo is consumed by both training and serving, and a change that's safe offline broke online low-latency serving. How would you gate feature PRs with a train/serve parity check and a p99 latency budget check before merge?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Isolating CI Environments](<../../../../TOOLS/Git_and_GitHub/Git_Core/Isolating_CI_Environments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that changes to our feature-store transformation repo don't break offline training or online low-latency serving, we need to implement a robust gate-check process. This involves verifying both train/serve parity and adherence to a p99 latency budget before merging any pull request (PR). Here’s how this can be achieved:

First, let's summarize the core idea: We will integrate automated checks within our CI/CD pipeline that run every time a PR is created. These checks will validate the feature transformations against both training and serving environments to ensure they are compatible.

Now, triaging the question, we need to consider:

Train/Serve Parity Check: Ensure that any changes in feature transformations do not alter the data schema or logic in such a way that it impacts model training.
Latency Budget Check: Confirm that serving these features will meet our required low-latency requirements.
Step-by-Step Solution

Setup CI/CD Pipeline:

Integrate GitHub Actions for automated testing and validation.
Define the workflow to run tests whenever a PR is opened, branched off, or pushed.

Train/Serve Parity Check:

Create a parity test suite that runs on both training and serving environments with the same feature transformations.
Use unit tests and integration tests to compare outputs from these two environments.

#### Example
Run feature transformations using a model trained on production data in both environments (training and serving) and check for identical results.

Latency Budget Check:

Implement latency profiling tools that can measure the performance of serving requests with transformed features.
Define a p99 latency threshold based on our service level agreement (SLA).
Use load testing to simulate high-traffic scenarios and ensure that the new feature transformations do not exceed this threshold.

Automated Execution:

Configure GitHub Actions to automatically run both types of tests whenever a PR is created.

#### Example
A GitHub Action workflow might look like this:
YAML
name: Feature Store Gate Check

on:
  pull_request:
    branches:
      - main
jobs:
  parity_check:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.9'
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
      - name: Run parity tests
        run: |
          pytest --features-parity

  latency_check:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.9'
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
      - name: Run latency tests
        run: |
          pytest --latency-check

Failure Handling:

If any test fails, the PR should be blocked from merging and a detailed report should be generated.
The report will include information on which checks failed and why.

Review Process:

Ensure that reviewers also manually verify the changes before final approval.
Provide clear instructions on how to handle failures and make necessary adjustments.

#### Closing Summary

By implementing these automated tests in our CI/CD pipeline, we can ensure that any changes to the feature-store transformation repo are thoroughly vetted for both training and serving compatibility. This process not only prevents regressions but also ensures high availability and performance of our online services.

#### Common Mistakes/Follow-Ups
Mistake: Failing to include both train/serve parity checks.

#### Follow-Up
Ensure all critical paths are tested against both environments.
Mistake: Not setting up proper latency monitoring or failing to define the budget threshold.

#### Follow-Up
Clearly document and set realistic p99 latency targets based on historical data.
Mistake: Neglecting manual review processes.

#### Follow-Up
Integrate a final manual approval step before merging.

**Comparison:**

```
Test Type	Description
Parity	Ensures identical outputs from training and serving environments.
Latency	Verifies p99 latency stays within defined threshold.
```

**Code:**

```yaml
# Example GitHub Actions workflow
name: Feature Store Gate Check

on:
  pull_request:
    branches:
      - main
jobs:
  parity_check:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.9'
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
      - name: Run parity tests
        run: |
          pytest --features-parity

  latency_check:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.9'
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
      - name: Run latency tests
        run: |
          pytest --latency-check
```

**Example input / output:**

```text
Input (example):
- PR with new feature transformation

Output:
- Parity check passes: True/False
- Latency check passes: True/False
- Detailed failure report if any checks fail
```

**Data:**

```json
{
  "example_test_data": {
    "training_environment_output": [1, 2, 3],
    "serving_environment_output": [1, 2, 3]
  },
  "p99_latency_threshold": 50
}
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. PR Created
6. Checkout Code
7. Set Up Python
8. Install Dependencies
9. Run Parity Tests
10. Parity Check Passes?
11. Generate Detailed FailureReport
12. Run Latency Tests
13. Latency Check Passes?
14. Generate Detailed FailureReport
15. Merge PR

