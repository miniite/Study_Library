---
type: "practice"
difficulty: "Expert"
---

# How do you decide whether to enforce a cost policy as a hard-deny at the IaC layer versus a detect-and-notify after deploy? Give me the risk framework for choosing prevention versus detection per policy class.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cost-aware Defaults](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Cost-aware_Defaults.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In deciding between enforcing a cost policy via a hard-deny at the Infrastructure as Code (IaC) layer or implementing a detect-and-notify post-deploy, you need to consider several factors including risk tolerance, policy impact on business operations, and potential costs. This decision involves a nuanced risk framework that balances prevention versus detection strategies based on specific policy classes.

**Comparison:**

```
Policy Type	Enforcement Layer	Impact on Operations	Risk Mitigation	Cost Implications
Hard-Deny	IaC	Immediate	High	Low
Detect & Notify	Post-deploy	Delayed	Medium	High
```

**Code:**

```python
# Example code for evaluating cost policies
def evaluate_cost_policy(policy_type, policy_details):
    if policy_type == "hard-deny":
        # Implement IaC layer enforcement logic here
        pass
    elif policy_type == "detect-and-notify":
        # Implement post-deploy detection and notification logic here
        pass
```

**Example input / output:**

```text
Input Example:
Evaluate cost policy for AWS EC2 instances with a daily budget of $100 using "hard-deny" enforcement.

Output Example:
Policy enforced at IaC layer. Any attempt to provision or modify EC2 instances above the threshold will fail.
```

**Data:**

```yaml
# Example cost policy definition schema
cost_policy:
  type: string # "hard-deny" or "detect-and-notify"
  threshold: float # e.g., $500
  resource_type: string # e.g., EC2, S3
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Policy Type Selection
6. Is Policy Critical?
7. Enforce at IaC Layer
8. Is Real-Time MonitoringRequired?
9. Implement Post-DeployDetection & Notify
10. No Action Required
11. Review Policy Impact onOperations
12. Finalize Implementation Plan
13. Adjust Notification Thresholds
14. Monitor and Optimize

