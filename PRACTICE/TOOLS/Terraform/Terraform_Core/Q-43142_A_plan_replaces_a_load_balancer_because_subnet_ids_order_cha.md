---
type: "practice"
difficulty: "Deep"
---

# A plan replaces a load balancer because subnet_ids order changed. Why is list order significant here, and how do you make the module order-stable?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Refactoring with State Moves](<../../../../TOOLS/Terraform/Terraform_Core/Refactoring_with_State_Moves.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a Terraform module’s subnet_ids parameter changes due to altered list order in a plan, it can lead to unexpected behavior or even resource deletion if the subnet IDs don’t align correctly with existing resources like the load balancer. The significance of list order lies in how Terraform evaluates and applies changes, making stability crucial for avoiding unintended side effects.

Let's break down the key aspects:

Understanding State Stability: Terraform uses state to track the current infrastructure. Changes to this state must be managed carefully to avoid disrupting existing resources.
Module Parameters and Subnet IDs: The subnet_ids parameter in a module defines which subnets are associated with network resources such as load balancers. List order is significant because it determines how Terraform interprets these subnet associations.

To ensure the module's state remains stable, we need to maintain consistent list ordering within our Terraform configuration.

#### Triaging the Question
What the Question Asks: Why does the order of subnet_ids matter, and how can we make this parameter order-stable in a module?
Interviewer's Probe: The interviewer is testing your understanding of state management, resource dependencies, and Terraform’s evaluation process. They are also assessing how you handle configuration stability.

#### First-Principles Solution
Understanding Resource Dependency: In AWS, subnets must be configured correctly for a load balancer to function properly. Changing the subnet_ids order can cause Terraform to misinterpret which subnets should be associated with the load balancer.
State Consistency and Order Stability:
Ensure that your subnet_ids list is consistent across all configurations where this parameter is used.
Use Terraform’s built-in functions or custom logic to maintain a stable order in sensitive parameters.

Here's an example approach:

TERRAFORM
module "load_balancer" {
  source = "./modules/load_balancer"

  # Ensuring subnet_ids are passed as a list with consistent order
  subnet_ids = [for s in module.subnet.outputs.public_subnets : s.id]
}

In this example, we dynamically generate the subnet_ids from another module or output to ensure that the order remains stable.

Using State Locks: When making significant changes to your infrastructure, you can use state locks to prevent accidental deletions or updates.
Terraform Workspaces and Version Control: Utilize workspaces in Terraform for different environments (dev, staging, prod) and version control systems like Git to manage changes systematically.

#### Closing Summary

By ensuring that sensitive parameters like subnet_ids are consistently ordered, you can avoid unintended behavior during Terraform plan execution. Use dynamic generation, state locks, and proper configuration management practices to maintain order-stable modules.

#### Common Mistakes
and Follow-Ups
Not Maintaining Consistent Order: Accidental reordering of subnets in the subnet_ids parameter can lead to resource misconfigurations.
Lack of State Management Practices: Not using state locks or version control systems can result in accidental changes.
Overlooked Dependent Resources: Failure to consider how subnet_ids interact with dependent resources (e.g., load balancers, network ACLs) can cause broader issues.

#### Follow-ups
might include discussing best practices for managing sensitive parameters, the role of Terraform workspaces, and strategies for testing configurations before applying them in production.

**Code:**

```
TERRAFORM
Copy
module "load_balancer" {
  source = "./modules/load_balancer"

  subnet_ids = [for s in module.subnet.outputs.public_subnets : s.id]
}
```

