---
type: "practice"
difficulty: "Intermediate"
---

# How do you enforce tagging discipline so Cost Management reports are actually usable, and what do you do about untagged legacy resources?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Azure DevOps and Artifact Governance](<../../../../TOOLS/Azure/Azure_Platform/Azure_DevOps_and_Artifact_Governance/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that Azure resources are properly tagged for effective cost management reporting, we need to establish a consistent tagging policy and enforce it across all environments. This involves setting up governance rules, educating stakeholders, and addressing legacy resource tagging issues. Here's a step-by-step solution from first principles.

First, let's define the key concepts and assumptions:

We are working with an intermediate DevOps environment where some existing resources may be untagged.
The goal is to make cost management reports usable by ensuring all new and existing resources follow a consistent tagging policy.
We will leverage Azure Policy for enforcing tags, but also consider supplementary measures like automated tagging tools.
Step-by-Step Solution

Define the Tagging Requirements:

Establish a clear set of required tags based on your organization's needs (e.g., department, environment, cost center).
For example, you might require Department, Environment (Dev, Test, Prod), and Cost Center.

Implement Azure Policy for Enforcement:

Create an Azure Policy that checks if resources have the required tags.
Use a if-not-exists rule to ensure new resources comply with tagging requirements.

Educate Stakeholders:

Train all resource owners and administrators on the importance of proper tagging.
Provide documentation, workshops, or webinars to ensure everyone understands their role in maintaining compliance.

Audit Existing Resources:

Use Azure Policy to identify untagged resources from your legacy environment.
Create a comprehensive list of these resources for follow-up actions.

Automate Tagging Where Possible:

For new resources, use automation tools like Azure Resource Manager templates or Terraform scripts that enforce tagging policies.

#### Cons
ider using Azure DevOps pipelines to automatically apply tags during deployment processes.

Review and Update Policy as Needed:

Regularly review the tagging policy to ensure it aligns with business needs.
Adjust rules based on changes in organizational structure or new financial requirements.

#### Closing Summary
and Common Mistakes

In summary, ensuring proper tagging discipline requires a combination of governance tools like Azure Policy, continuous education for stakeholders, and a phased approach to address legacy resources. Key areas to focus on include:

Overlooking Existing Resources: Failing to identify and tag existing untagged resources can lead to incomplete cost management reports.
Misunderstanding Policy Scope: Ensuring that the policy is applied consistently across all relevant Azure services and resource types.
Insufficient Automation: Relying solely on manual processes for tagging new resources can be error-prone and time-consuming.

By implementing these steps, we can ensure that Cost Management reports are useful and provide actionable insights into our organization's spending patterns.

**Comparison:**

```
Resource Type	Required Tags
Virtual Machines	Department, Environment, CostCenter
Network Interfaces	Department, Environment, CostCenter
```

**Code:**

```powershell
# Example PowerShell script to check compliance with Azure Policy
$subscriptionId = "<your-subscription-id>"
Connect-AzAccount -Subscription $subscriptionId

$policyDefinitionName = "Tag Compliance"
$policyAssignments = Get-AzPolicyAssignment | Where-Object { $_.DisplayName -eq $policyDefinitionName }

if ($null -eq $policyAssignments) {
    Write-Output "No policy assignment found with name: $policyDefinitionName"
} else {
    Write-Output "Policy compliance checked successfully."
}
```

**Example input / output:**

```text
Input:
- Existing resources with no tags

Output:
- List of untagged resources identified and marked for tagging
- New resources automatically tagged during deployment
```

**Data:**

```json
{
  "tagName": [
    { "name": "Department", "value": "IT" },
    { "name": "Environment", "value": "Prod" },
    { "name": "CostCenter", "value": "0123456789" }
  ]
}
```

**Flow:**

1. Define Tagging Requirements
2. B
3. Implement Azure Policy
4. Educate Stakeholders
5. Audit Existing Resources
6. Automate Tagging
7. Review and Update Policy

