---
type: "practice"
difficulty: "Intermediate"
---

# What is Terraform and what problem does it solve?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Cloud Deployment and Service Models](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Cloud_Deployment_and_Service_Models.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: Terraform is an infrastructure as code (IaC) tool that allows you to define, manage, and version your cloud resources in a consistent and repeatable way. It solves the problem of manually managing cloud resources, which can be error-prone and difficult to scale.

Terraform is a powerful tool for provisioning and managing cloud resources across multiple providers like AWS, Azure, and Google Cloud. The question is asking us to explain what Terraform is and how it addresses common issues in infrastructure management, particularly focusing on the need for automation and consistency in provisioning cloud resources.

To tackle this, we'll start by understanding the context of IaC, then dive into the specific benefits that Terraform provides, followed by a detailed explanation of its capabilities and use cases. Finally, we'll discuss some common mistakes to avoid when using Terraform.

**Code:**

```hcl
# Example Terraform configuration for creating an S3 bucket
resource "aws_s3_bucket" "example" {
  bucket = "example-bucket"
}
```

