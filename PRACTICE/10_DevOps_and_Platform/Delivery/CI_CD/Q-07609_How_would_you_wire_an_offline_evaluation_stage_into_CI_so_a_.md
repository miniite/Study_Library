---
type: "practice"
difficulty: "Intermediate"
---

# How would you wire an offline evaluation stage into CI so a PR that changes feature code automatically reports metric deltas against a fixed validation set?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Pull Request Pipelines](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Pull_Request_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To integrate an offline evaluation stage into Continuous Integration (CI) for a pull request (PR) that modifies feature code, we need to ensure that the PR's changes are automatically tested using a pre-defined validation dataset. This involves setting up a CI pipeline that runs this offline evaluation upon every push or PR creation. The core idea is to isolate the evaluation process from the main build and test phases to prevent any disruption in the regular development flow.

In essence, the question is asking how to set up an automated job that:

Triggers on the creation of a new PR.
Runs an offline evaluation script using a fixed validation dataset.
Reports the performance metrics before and after the changes introduced by the PR.
Ensures the evaluation process completes without interfering with other CI jobs.

#### Assumptions
include having a well-defined validation set, the presence of a suitable online/offline evaluation framework, and the ability to configure automated tests in your CI system (e.g., Jenkins, GitHub Actions, GitLab CI).

We will break down this problem from first principles by setting up a basic workflow:

Triggering the Evaluation: The PR creation should trigger an event that initiates the offline evaluation.
Setting Up the Offline Evaluation Script: This script will run against the validation dataset to measure performance metrics.
Storing and Retrieving the Validation Data: Ensure the validation data is accessible within the CI environment.
Reporting the Metrics: Post-evaluation, report the metric deltas to a tracking system or UI.

Let’s outline this step by step:

Event Trigger: The PR creation event should be detected by a webhook or similar mechanism in your CI system. For example, GitHub Actions can listen for push and pull_request events.

Offline Evaluation Script: This script would fetch the latest commit from the branch being reviewed (e.g., via Git), apply any necessary changes to the feature code, run the offline evaluation against the validation dataset, and output the performance metrics.

Validation Data Storage: The validation data should be stored in a reliable format that can be easily accessed by the evaluation script. This could be a database, a JSON file, or even a separate Git repository.

Reporting Metrics: After evaluating, the results should be posted to a tracking system such as Prometheus, Grafana, or a custom dashboard. Alternatively, you might push these metrics to an internal Slack channel for quick feedback.

Here’s a basic example using GitHub Actions:

YAML
# .github/workflows/offline-eval.yml
name: Offline Evaluation

on:
  pull_request:
    branches:
      - main

jobs:
  offline-evaluation:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout Repository
      uses: actions/checkout@v2
    - name: Set up Python environment (if necessary)
      run: |
        python3 --version
    - name: Install dependencies
      run: |
        pip install -r requirements.txt
    - name: Run Offline Evaluation Script
      env:
        TOKEN: ${{ secrets.TOKEN }}
        BASE_URL: https://api.github.com/graphql
      run: |
        python3 evaluation.py --validation-data validation.json

This workflow triggers on pull requests to main, checks out the repository, sets up any necessary environment (e.g., Python), installs dependencies, and runs a script to perform offline evaluation.

**Code:**

```yaml
# .github/workflows/offline-eval.yml
name: Offline Evaluation

on:
  pull_request:
    branches:
      - main

jobs:
  offline-evaluation:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout Repository
      uses: actions/checkout@v2
    - name: Set up Python environment (if necessary)
      run: |
        python3 --version
    - name: Install dependencies
      run: |
        pip install -r requirements.txt
    - name: Run Offline Evaluation Script
      env:
        TOKEN: ${{ secrets.TOKEN }}
        BASE_URL: https://api.github.com/graphql
      run: |
        python3 evaluation.py --validation-data validation.json
```

**Example input / output:**

```text
Input:
- A pull request with changes to feature code.
Output:
- Performance metrics (e.g., accuracy) before and after the PR.
Example Dry Run:
1. Pull Request created for branch "feature-x".
2. CI triggers offline evaluation using validation.json.
3. Evaluation script runs, comparing new vs old features.
4. Metrics: Feature A improved from 0.92 to 0.95; Feature B stayed at 0.85.
```

**Data:**

```json
{
  "validation_data": {
    "features": [
      {"id": 1, "name": "Feature A", "accuracy": 0.92},
      {"id": 2, "name": "Feature B", "accuracy": 0.85}
    ]
  }
}
```

