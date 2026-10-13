---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd write an OPA/Conftest policy in a Terraform pipeline that flags expensive resource types or missing tags. Where do you make it a hard fail versus a warning?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cost-aware Defaults](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Cost-aware_Defaults.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure effective cloud cost governance and manage tagging practices, we'll write an OPA (Open Policy Agent) policy with Conftest integrated into our Terraform pipeline. This approach helps us identify and prevent costly resource types or missing tags before deployment. We will make certain checks a hard fail to enforce strict compliance policies, while others might be warnings to allow for corrective action.

**Comparison:**

```
Rule	Description	Severity
expensive_resource	Flagges costly resources	Hard Fail
missing_tags	Alerts for missing tags	Warning
```

**Code:**

```hcl
# Example OPA policy in Terraform template usingrego syntax

package cloud_cost_governance

import data "github.com/open-policy-agent/opa/plugins/data/tfstate"

rule expensive_resource {
    input = get_input()
    output = input.resource == "aws_instance" && input.cost > 100
}

rule missing_tags {
    input = get_input()
    output = !input.tags["Environment"] || !input.tags["Project"]
}
```

**Example input / output:**

```

Input:

JSON
Copy
{
  "resource": "aws_instance",
  "cost": 200,
  "tags": {
    "Environment": "staging",
    "Project": ""
  }
}

Output (based on the policy):

expensive_resource rule: True
missing_tags rule: False
```

**Data:**

```json
{
  "get_input": {
    "resource": "aws_instance",
    "cost": 200,
    "tags": {
      "Environment": "staging",
      "Project": ""
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. No
4. Yes
5. Resource Creation Request
6. Is Cost > Threshold?
7. Raise Hard Fail Alert
8. Are Tags Present?
9. Issue Warning, AllowDeployment
10. Approval Required forDeployment

