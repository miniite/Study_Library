---
type: "practice"
difficulty: "Deep"
---

# How would you design an infrastructure-as-code module structure so 30 microservices share networking and IAM patterns without copy-paste drift?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cloud Network Topology](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Stateless_by_Default/Cloud_Network_Topology.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an infrastructure-as-code (IAC) module structure that enables 30 microservices to share networking and IAM patterns without copy-paste drift, we need to focus on modularity, abstraction, and centralized management. This approach ensures consistency across all services while minimizing the risk of drift and making maintenance more efficient.

#### Triaging the Question

The core question is about designing a scalable IAC structure that promotes reusability of networking and IAM configurations without relying on copy-pasting code. The interviewer is probing your ability to design modular, maintainable infrastructure components and understand best practices in cloud governance and automation.

#### Assumptions
We are working with AWS as the primary cloud provider.
Microservices are built using container orchestration like Kubernetes.
Networking needs include private and public subnets, VPC peering for inter-service communication, and network ACLs.
IAM policies and roles need to be managed centrally.

#### First-Principles Solution

Modularize Infrastructure Components:

Create separate modules for networking (e.g., networking.tf), IAM (e.e., iam_roles.tf, policies.tf), and deployment (e.g., kubernetes.yaml).

Centralize Configuration Management:

Use a centralized configuration management tool like Terraform to manage the infrastructure.
Define variables for common parameters such as subnet IDs, security group rules, role ARNs.

Leverage Modules and Outputs:

Create reusable modules that can be included in other project files.
Use outputs from these modules to pass necessary configurations to downstream resources.

Use Variables and Dynamic Configurations:

Define variables for environment-specific settings (e.g., environment, region).
Implement dynamic configuration using Terraform’s for_each or count features to scale the infrastructure as needed.

Implement Change Control and Testing:

Ensure that all changes are version-controlled.
Integrate continuous integration/continuous deployment (CI/CD) pipelines for testing and validation.

#### Example
Code

Here is a simplified example of how you might structure your IAC code:

HCL
# infrastructure/modules/networking/main.tf
module "networking" {
  source = "./modules/networking"

  vpc_id       = var.vpc_id
  public_subnets = var.public_subnets
  private_subnets = var.private_subnets
}

# infrastructure/modules/iam/main.tf
module "iam" {
  source = "./modules/iam"

  roles = ["service-role", "admin-role"]
  policies = ["data-access-policy", "write-access-policy"]
}

#### Closing Summary
and Common Mistakes/Follow-Ups

Summary: By modularizing infrastructure components, centralizing configurations, and leveraging dynamic configuration management tools like Terraform, we can ensure that 30 microservices share consistent networking and IAM patterns without copy-paste drift.

#### Common Mistakes

Failing to define variables for environment-specific settings.
Not implementing version control or CI/CD pipelines.
Overcomplicating the architecture with unnecessary abstractions.

#### Follow-Ups
The interviewer might ask about specific tools like Terraform, how you handle security groups and network ACLs, or more advanced topics like cross-account sharing of IAM roles.

**Code:**

```hcl
module "networking" {
  source = "./modules/networking"

  vpc_id       = var.vpc_id
  public_subnets = var.public_subnets
  private_subnets = var.private_subnets
}

module "iam" {
  source = "./modules/iam"

  roles = ["service-role", "admin-role"]
  policies = ["data-access-policy", "write-access-policy"]
}
```

**Flow:**

1. Define Networking Module
2. Create VPC and Subnets
3. Define IAM Module
4. Create Roles and Policies
5. Include Modules in MainProject
6. Call Networking and IAMModules
7. Use Outputs from Modules
8. Pass Configurations toMicroservices

