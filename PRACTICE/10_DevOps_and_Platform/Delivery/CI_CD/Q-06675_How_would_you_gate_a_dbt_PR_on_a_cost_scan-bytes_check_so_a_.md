---
type: "practice"
difficulty: "Deep"
---

# How would you gate a dbt PR on a cost/scan-bytes check so a model that would scan terabytes in BigQuery can't merge without review?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Data and Analytics CI Gates](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Data_and_Analytics_CI_Gates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To gate a dbt PR on a cost/scan-bytes check, we need to ensure that large models, particularly those that could incur significant costs due to high scan bytes in BigQuery, undergo thorough review before merging into the main codebase. This involves several steps: setting up monitoring of resource usage, automating checks within CI/CD pipelines, and integrating these checks with the PR process.

First, we'll break down what's being asked:

Monitoring Resource Usage: We need to track how much data each model scans.
Automated Checks in CI/CD: Implement automated scans to trigger alerts or block merges if scan bytes exceed a threshold.
PR Process Integration: Ensure that the PR process is aware of these costs and triggers manual review when necessary.

#### Triaging the Question

The question asks for a mechanism to gate dbt model PRs based on cost/scan-bytes thresholds in BigQuery. The key elements being probed are:

How to monitor scan bytes.
How to automate checks during CI/CD.
How to integrate these checks with the PR process.

#### Assumptions
We have access to BigQuery and dbt.
We use a Git-based workflow for managing our data models.
The PR process involves automated CI/CD pipelines before manual review in Jenkins/GitLab/GitHub Actions, etc.
Cost thresholds are predefined based on organizational standards.

#### First-Principles Solution

Set Up Monitoring:

Use BigQuery's usage metrics to track the scan bytes for each model.
Define a custom metric or dashboard to monitor these metrics in real-time.

Integrate with CI/CD Pipelines:

Modify your dbt configuration to include a post-run script that collects and logs scan byte data.
Use a tool like bq command-line interface (CLI) to query BigQuery for usage statistics.

Automate Cost Checks:

Write a custom script or use an existing CI/CD platform’s plugin to run these checks as part of the build process.
Trigger alerts if scan bytes exceed predefined thresholds, and block the PR from merging until resolved.

PR Process Integration:

Modify your GitOps processes to include cost checks as a mandatory step in the review workflow.
Use Jenkins/GitLab/GitHub Actions to implement these checks, ensuring they are part of the CI/CD pipeline.
Step-by-Step Solution

Collect and Log Scan Bytes:

PYTHON
import subprocess

def get_scan_bytes(model_name):
    # Example command using bq CLI
    cmd = f'bq show --usage {model_name}'
    result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
    usage_info = result.stdout.split('\n')
    scan_bytes = 0
    for line in usage_info:
        if 'Total bytes processed' in line:
            scan_bytes = int(line.split()[4])
            break
    return scan_bytes

model_name = "my_large_model"
scan_bytes = get_scan_bytes(model_name)
print(f"Scan bytes for {model_name}: {scan_bytes}")

Set Up CI/CD Check:

Add a step to your dbt project's profiles.yml or dbt_project.yml to call this script.
Integrate with Jenkins/GitLab pipeline scripts.

Automate Cost Alerts and Block Merges:

YAML
stages:
  - scan-bytes-check

scan-bytes-check:
  stage: scan-bytes-check
  image: python:3.9
  script:
    - scan_bytes = $(get_scan_bytes my_large_model)
    - if [ $scan_bytes -gt 1000000000 ]; then
        echo "Scan bytes exceeded threshold"
        exit 1
      else
        echo "Scan bytes within acceptable range"
      fi

Integrate with PR Review Workflow:

Ensure that the CI/CD pipeline step is part of the PR review process.
Use GitOps tools to ensure these checks are part of the merge strategy.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, gating dbt PRs based on cost/scan-bytes involves setting up monitoring, automating checks in CI/CD pipelines, and integrating these with the PR review process. Common mistakes include not having a robust monitoring system, ignoring cost thresholds during development, or not effectively communicating these requirements to all team members.

#### Follow-ups
could include:

How to handle edge cases where models have unexpected high scan bytes.
Performance optimizations for large models that exceed threshold limits.
Cost savings and resource utilization strategies beyond gating PRs.

**Comparison:**

| Model Name | Scan Bytes |
|------------|-----------|
| my_large_model | 500,000,000 |
| another_model | 1,500,000,000 |
| yet_another_model | 700,000,000 |

**Code:**

```

import subprocess

def get_scan_bytes(model_name): cmd = f'bq show --usage {model_name}' result = subprocess.run(cmd, shell=True, capture_output=True, text=True) usage_info = result.stdout.split('\n') scan_bytes = 0 for line in usage_info: if 'Total bytes processed' in line: scan_bytes = int(line.split()[4]) break return scan_bytes

model_name = "my_large_model" scan_bytes = get_scan_bytes(model_name) print(f"Scan bytes for {model_name}: {scan_bytes}")

CODE
Copy

```yaml
stages:
  - scan-bytes-check

scan-bytes-check:
  stage: scan-bytes-check
  image: python:3.9
  script:
    - scan_bytes = $(get_scan_bytes my_large_model)
    - if [ $scan_bytes -gt 1000000000 ]; then
        echo "Scan bytes exceeded threshold"
        exit 1
      else
        echo "Scan bytes within acceptable range"
      fi
```

**Example input / output:**

```text
input:
  - model_name: "my_large_model"
output:
  scan_bytes: 500000000

input:
  - model_name: "another_model"
output:
  scan_bytes: 1500000000

input:
  - model_name: "yet_another_model"
output:
  scan_bytes: 700000000
```

**Data:**

```yaml
# Example dbt_project.yml for cost checks
metrics:
  - model: my_large_model
    scan_bytes_threshold: 1000000000

profiles.yml:
  default:
    project_dir: .
    target: dev
    outputs:
      dev:
        type: bigquery
        dataset: my_dataset
        schema: my_schema
```

**Flow:**

1. Initiate PR
2. Run dbt model
3. Collect scan bytes usingcustom script
4. Store in CI/CD pipeline
5. Automated checks (BigQueryusage metrics)
6. Alert if cost exceeds threshold
7. Block PR merge
8. Manual review required

