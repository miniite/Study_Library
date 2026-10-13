---
type: "practice"
difficulty: "Deep"
---

# Design the architecture for a 'preview environment per pull request' that spins up frontend, backend, and a seeded database in under two minutes for 200 engineers. How do you keep costs sane and prevent these ephemeral envs from becoming a shadow production?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Frontend Graceful Degradation](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Frontend_Graceful_Degradation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing an architecture for a 'preview environment per pull request' (PEPR) that can be spun up in under two minutes, our primary goal is to balance resource efficiency with reliability and security. This architecture must support 200 engineers while keeping costs manageable and ensuring that these environments don't become a shadow of the production system. Let's break down this problem step by step.

#### Triaging the Question

The core of the question is about designing an efficient, cost-effective, and scalable environment for developers to test their pull requests before merging them into the main branch. We need to ensure that each engineer can quickly access a sandboxed environment without incurring excessive costs or creating a risk of shadow production systems.

Key Assumptions
Resource Utilization: Each PEPR must have dedicated resources (compute, storage) but should not over-provision.
Security and Isolation: The environments need to be isolated from the main production system while still providing necessary security measures.
Cost Management: Resources used by each PEPR are billed separately and efficiently managed.
Step-by-Step Solution
1. Environment Composition

Each PEPR will consist of:

Frontend (React or similar)
Backend (Node.js, Python, etc.)
Database (PostgreSQL, MySQL, etc.)

These components need to be spun up quickly, which means using infrastructure as code tools like Terraform, CloudFormation, or ARM templates.

2. Infrastructure Selection

To keep costs sane:

Cloud Provider: Use cloud providers that support rapid deployment and have built-in cost management tools (e.g., AWS, GCP).
Compute Resources: Opt for reserved instances where possible to save costs.
Networking: Utilize VPCs (Virtual Private Clouds) with private subnets to isolate environments. This also helps in managing security groups.
3. Provisioning Tool

Use a CI/CD pipeline to automate the provisioning of these environments:

Trigger: Integration between Git provider and CI/CD tool (GitHub, Bitbucket).
Provisioning Script: A script using Terraform or similar tools that spins up resources.
Lifecycle Management: Ensure resources are destroyed once the pull request is merged.
4. Database Initialization

For seeding databases:

Use a snapshot-based approach for creating initial data.
Implement an incremental backup/restore strategy to keep the database size manageable and reduce costs.
Detailed Steps

Environment Request:

A developer triggers the PEPR creation via their Git provider’s webhooks or through a custom UI integrated with the CI/CD pipeline.

Provisioning Script:

BASH
# Example Terraform script snippet for provisioning an environment
resource "aws_instance" "pepr" {
  ami           = var.pepr_ami_id
  instance_type = var.instance_type
  subnet_id     = var.subnet_id
  security_groups = [var.security_group]

  tags = {
    Name    = "${var.environment}-${var.pull_request_number}"
    Project = "PEPR"
  }
}

Database Initialization:

Use a predefined set of data seeds for the database.
Implement a backup/restore strategy using AWS RDS Snapshots.

#### Closing Summary

To summarize, we can achieve this by leveraging cloud provider services to spin up environments quickly and efficiently. By automating the provisioning process with CI/CD pipelines and ensuring proper lifecycle management, we can keep costs under control while maintaining security and isolation from production systems. This approach ensures that each pull request is validated in a sandboxed environment without impacting the overall budget.

#### Common Mistakes
and Follow-Ups
Over-provisioning: Ensuring that resources are not overallocated.
Security Risks: Failing to properly isolate environments or neglecting security best practices.
Cost Overruns: Not implementing efficient resource management strategies like reserved instances or proper lifecycle management of environments.

**Comparison:**

```
Resource	Cost Management Strategy
EC2	Reserved instances
RDS	Snapshots and backups
VPC	Private subnets
```

**Code:**

```bash
# Example Terraform script snippet for provisioning an environment
resource "aws_instance" "pepr" {
  ami           = var.pepr_ami_id
  instance_type = var.instance_type
  subnet_id     = var.subnet_id
  security_groups = [var.security_group]

  tags = {
    Name    = "${var.environment}-${var.pull_request_number}"
    Project = "PEPR"
  }
}
```

**Example input / output:**

```text
Input:
- Triggered via GitHub webhook for a pull request

Output:
- Frontend, backend, and database provisioned in under two minutes
- Environment named with the pull request number
- Resources tagged appropriately
- Database seeded from a snapshot
```

**Data:**

```yaml
# Example Terraform variables file for provisioning an environment
pepr_ami_id: ami-0c55b159cbEXAMPLE
instance_type: t2.micro
subnet_id: subnet-0d4e76fEXAMPLE
security_group: sg-0d1730ddEXAMPLE
```

**Flow:**

1. Pull Request Triggered
2. Trigger CI/CD
3. Terraform Provisioning Script
4. Resource Allocation & Tagging
5. Database Initialization
6. Environment Ready
7. Snapshot Backup
8. Cleanup on Merge
9. Cost Optimization

