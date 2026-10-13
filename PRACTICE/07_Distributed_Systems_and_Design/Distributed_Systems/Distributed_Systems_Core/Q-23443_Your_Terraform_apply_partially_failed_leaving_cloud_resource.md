---
type: "practice"
difficulty: "Intermediate"
---

# Your Terraform apply partially failed leaving cloud resources in a state that doesn't match the state file. Walk me through reconciling drift safely without recreating live resources and why import beats blind re-apply here.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Verifying Live Migrations](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Verifying_Live_Migrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you encounter a situation where Terraform applies partially fail, leaving cloud resources in an inconsistent state with the state file, it’s crucial to address this drift carefully without recreating live resources unnecessarily. This scenario requires understanding and leveraging Terraform's import feature to reconcile the state. Let's break down how to safely resolve this issue.

Firstly, let's define what we're dealing with: a partially applied Terraform plan that has left resources in a state where they exist but are not managed by Terraform (unmanaged resources) or are in a state that doesn't match the desired configuration defined in the Terraform code. The goal is to bring these resources back into alignment with your Terraform configuration without causing downtime.

#### Triage
and Assumptions
Partial Apply Failure: Resources were created but Terraform stopped before completion, leaving some resources unmanaged.
Live Resources: These are currently operational but may be misconfigured or not in the desired state.
State File: Contains the definitive list of managed resources as defined by your infrastructure-as-code (IaC) configuration.

#### First Principles Solution
Identify Unmanaged Resources: Use terraform state commands to identify which resources are unmanaged and which ones have drifted from their intended state.
Import Resources into Terraform Management:
Import each unmanaged resource back into Terraform using the terraform import command.
Ensure you understand the resource type, ID, and any dependencies before importing.
Reconcile Drifted Resources: Once all resources are managed by Terraform, use a plan to identify changes needed without actually applying them.
Apply Changes Gracefully:
Apply the changes using terraform apply.
Monitor for any issues during or after the apply.
Step-by-Step Solution

List Current State vs Desired State:

SH
terraform state list

This will show all resources managed by Terraform.

Identify Drifted Resources: Compare this with your desired state or use:

SH
terraform plan -out=tfplan && diff <(terraform show tfplan | jq .resource_changes[].actions) <(terraform show)

This will highlight which resources have drifted.

Import Unmanaged Resources into Terraform: For each unmanaged resource, use:

SH
terraform import <resource_type>.<resource_id> <external_id>

Replace <resource_type> with the type of resource (e.g., aws_instance), and <resource_id> and <external_id> as appropriate.

Reconcile Changes: After importing all resources, run a plan to see what changes will be applied:

SH
terraform plan -out=tfplan

Apply the Plan Safely: Finally, apply the changes while monitoring closely:

SH
terraform apply tfplan

#### Common Mistakes
and Follow-Ups
Mistake: Blindly reapplying Terraform without importing unmanaged resources can lead to resource duplication or loss.

#### Follow-Up
Ensure you have a backup plan in case the drift is too severe for terraform import alone. Use automation scripts to manage complex states.

#### Closing Summary

By leveraging Terraform's import feature, we safely reconcile drifted resources without recreating live ones. This approach minimizes disruption and ensures that your infrastructure aligns with your IaC definitions.

**Code:**

```
SH
Copy
# Example commands
terraform state list
terraform import aws_instance.example i-0123456789abcdef0
terraform plan -out=tfplan
terraform apply tfplan
```

**Flow:**

1. Identify Drift
2. List Resources Managed byTerraform
3. terraform state list
4. Identify Unmanaged andDrifted Resources
5. Import Unmanaged Resources
6. terraform import<resource_type>.<resource_id> <external_id>
7. Reconcile Changes Using Plan
8. terraform plan -out=tfplan
9. Apply Changes Safely
10. terraform apply tfplan

