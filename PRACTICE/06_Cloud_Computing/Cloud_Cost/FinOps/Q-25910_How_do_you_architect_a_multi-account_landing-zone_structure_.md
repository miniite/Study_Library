---
type: "practice"
difficulty: "Expert"
---

# How do you architect a multi-account/landing-zone structure (AWS Organizations, GCP folders) specifically to make cost governance, blast-radius isolation, and commitment sharing tractable? What are the anti-patterns you've seen?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Shadow IT and Budget Evasion](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Shadow_IT_and_Budget_Evasion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a robust multi-account/landing-zone structure in AWS Organizations or GCP folders for effective FinOps and cloud cost governance, we need to focus on several key aspects: blast-radius isolation, commitment sharing, and governance enforcement. The architecture should be designed with these goals in mind from the outset.

#### Triage

The primary objective of this architecture is to ensure that costs are managed effectively while maintaining a high level of security and compliance across multiple accounts or folders. AWS Organizations and GCP folders provide mechanisms for creating and managing separate organizational structures, which can help isolate teams and reduce blast radius in case of issues. However, the devil is often in the details—specifically, how policies and enforcement are managed.

#### First-Principles Solution
1. Multi-Account/Landing-Zone Structure
Organizations/Folders: Use AWS Organizations or GCP folders to create separate accounts for different teams or services.
Shared Services Accounts: Establish a few shared services (like logging, monitoring, security) that are used across multiple projects but isolated from business-critical applications.
2. Cost Governance
Budgets and Alerts: Implement AWS Budgets or GCP Cost Management to set up cost thresholds and receive alerts when costs exceed predefined limits.
Service Control Policies (SCPs): SCPs can be used to enforce access policies across multiple accounts, ensuring that certain resources are restricted based on roles.
3. Blast-Radius Isolation
VPC Peering: In AWS, VPC peering and shared VPC can help isolate environments while allowing necessary communications.
Network Policies: Use network policies to control traffic between VPCs or subnets effectively.
4. Commitment Sharing
Shared Tenancy Models: For AWS, this could mean using a single account for shared services with appropriate isolation mechanisms in place.
Cross-Account Roles and Policies: Establish trust between accounts via cross-account roles that allow controlled access to resources without full ownership.

#### Common Mistakes/Follow-Ups
Over-Complexity: Over-engineering the structure can lead to increased management overhead. It’s important to balance complexity with usability.
Misconfigured SCPs: Incorrectly configured SCPs can inadvertently lock out key services or personnel, leading to operational issues.
Lack of Granular Policies: Not implementing detailed policies at the resource level can result in overly broad and potentially dangerous permissions.

#### Closing Summary

By carefully designing a multi-account/landing-zone structure with blast-radius isolation, cost governance, and commitment sharing mechanisms, you can ensure that your cloud environment is both secure and efficient. The key is to leverage the native tools of AWS Organizations or GCP folders while implementing best practices for policy enforcement and resource management.

Anti-Patterns
Flat Organization Structure: Not leveraging the hierarchical nature of AWS Organizations/GCP folders can lead to a tangled mess where it’s difficult to manage policies effectively.
Overly Permissive Policies: Failing to enforce strict permissions across accounts can result in excessive costs and security risks.
Inadequate Monitoring and Alerts: Missing out on essential monitoring tools and alerts means you won’t be notified of issues until they become critical.

**Comparison:**

```
SCP	Description
Allow-*	Allow access to certain services or actions in the shared service account.
Deny-*	Deny access to specific resources or actions unless explicitly allowed by another policy.
```

**Code:**

```python
# Example of setting up AWS Budgets using Boto3
import boto3

client = boto3.client('budgets')

def create_budget(budget_name, threshold_amount):
    budget = {
        'Budget': {
            'BudgetName': budget_name,
            'TimeUnit': 'MONTHLY',
            'BudgetLimit': {'Amount': threshold_amount, 'Unit': 'USD'},
            'CostFilters': [],
            'PlannedSpending': None
        }
    }

    response = client.create_budget(**budget)
```

**Example input / output:**

```text
Input:
- Create a budget with threshold amount of $10,000 for the Dev account.
Output:
{
  "BudgetArn": "arn:aws:budgets:us-east-1:123456789012:budget/dev-budget"
}
```

**Data:**

```json
{
  "aws_organizations_accounts": [
    {
      "Name": "Dev",
      "Emails": ["dev@example.com"],
      "Roles": ["Editor", "Viewer"]
    },
    {
      "Name": "Prod",
      "Emails": ["prod@example.com"],
      "Roles": ["Administrator"]
    }
  ]
}
```

**Flow:**

1. Cost Budget Setup
2. Create AWS Budgets withThreshold Amount
3. SCP Configuration
4. Enforce Access Policies AcrossAccounts
5. VPC Peering
6. Isolate Environments WhileAllowing NecessaryCommunications
7. Network Policies
8. Control Traffic BetweenVPCs/Subnets

