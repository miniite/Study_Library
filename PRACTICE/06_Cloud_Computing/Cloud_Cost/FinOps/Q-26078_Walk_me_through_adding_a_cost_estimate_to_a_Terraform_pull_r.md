---
type: "practice"
difficulty: "Intermediate"
---

# Walk me through adding a cost estimate to a Terraform pull request with Infracost. What do you show the reviewer, and would you ever block a merge on it?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Serverless Cost Visibility](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're diving into integrating FinOps principles within our DevOps processes by adding cost estimates to Terraform pull requests using Infracost. The goal is to ensure financial governance and transparency in infrastructure changes. Let’s break down the process from first principles.

Firstly, Infracost allows us to automatically generate cost estimates for Terraform code by analyzing the planned AWS/GCP/Azure resources. This tool helps in understanding the costs associated with our proposed infrastructure changes before they are deployed.

The key steps involved include:

Running Infracost to generate a cost breakdown.
Reviewing and discussing the cost estimate with team members.
Making decisions on whether to merge or not based on financial implications.

**Comparison:**

| Resource          | Cost Estimate |
|-------------------|---------------|
| aws_vpc           | $50.82/month   |
| aws_s3_bucket     | $17.49/month   |

**Code:**

```python
# Example of running Infracost in a CI/CD pipeline

import subprocess

def run_infracost():
    result = subprocess.run(['infracost', 'analyze'], capture_output=True, text=True)
    return result.stdout

cost_estimate = run_infracost()
print(cost_estimate)
```

**Example input / output:**

```text
| Resource          | Cost Estimate |
|-------------------|---------------|
| aws_vpc           | $50.82/month   |
| aws_s3_bucket     | $17.49/month   |

**Input:** Running the `infracost analyze` command.

**Output:** A JSON or YAML file with detailed cost estimates for each resource.
```

**Data:**

```json
{
  "aws_vpc": {
    "name": "my-vpc",
    "cost": "$50.82/month"
  },
  "aws_s3_bucket": {
    "name": "my-bucket",
    "cost": "$17.49/month"
  }
}
```

**Flow:**

1. Run Infracost
2. Generate Cost Estimate
3. Publish to PR Comments
4. Review Cost Estimate
5. Approve/Merge if withinbudget
6. Reject/Refactor if over budget

