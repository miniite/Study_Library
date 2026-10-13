---
type: "practice"
difficulty: "Intermediate"
---

# How do you tag resources created by Terraform so cost allocation actually works? Walk me through default_tags versus per-resource tags and what happens to resources Terraform doesn't manage.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Carbon and Cost Trade-offs](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Carbon_and_Cost_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When discussing FinOps and cloud cost governance, proper tagging of resources is crucial for accurate cost allocation. In this context, we need to understand the difference between default_tags in Terraform configurations and per-resource tags, and how these affect tagged resources that are managed or not by Terraform.

#### Triage
Understanding the Question

The question revolves around ensuring that resource tagging works effectively within a FinOps framework, particularly when using Terraform for provisioning infrastructure. The interviewer is probing your knowledge of Terraform's tagging mechanisms and their implications on cost allocation, especially in scenarios where resources are managed or unmanaged by Terraform.

#### First-Principles Solution

Terraform offers two primary ways to apply tags: default_tags and per-resource tags. Default_tags are used to set default values for all resources that do not specify individual tags. Per-resource tags allow you to define specific tags for each resource, overriding the defaults if necessary.

Default Tags: These are global settings applied to all resources by default unless overridden at a more granular level.
Per-Resource Tags: These are explicitly defined on an individual basis and can override default_tags for specific resources.
Step-by-Step Solution

Configure Default Tags:

HCL
terraform {
  required_version = ">= 0.14"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.26"
    }
  }

  # Set default tags for all resources
  default_tags {
    department = "engineering"
    environment = "dev"
  }
}

Apply Per-Resource Tags:

HCL
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  # Override default tags with per-resource tags
  tags = {
    Name        = "dev-web-server"
    Owner       = "john.doe@example.com"
    Environment = "development"
  }
}

Handling Unmanaged Resources:

Managed Resources: Terraform-managed resources will always respect the default_tags and per-resource tags as configured.
Unmanaged Resources: These are resources that Terraform does not manage or track, such as manually created S3 buckets or EBS volumes. In such cases, any cost allocation policies based on tagging may not work unless these unmanaged resources are explicitly tagged.

#### Closing Summary

Proper resource tagging is essential for effective FinOps and cloud cost governance. By using both default_tags and per-resource tags in Terraform configurations, you can ensure that all managed resources are correctly labeled according to your organizational structure and cost allocation policies. However, it’s important to remember that unmanaged resources may require additional manual steps to be properly tagged.

#### Common Mistakes/Follow-Ups
Forgetting to Apply Per-Resource Tags: While default_tags provide a base level of tagging, not all resources might use these defaults.
Misunderstanding Tag Inheritance: Resources can inherit tags from parent resources or other configurations; this needs to be considered when setting up your tagging strategy.
Lack of Monitoring and Validation: Regularly checking the tag compliance across both managed and unmanaged resources ensures that cost allocation policies are effective.

**Comparison:**

```
Default Tags	Per-Resource Tags
department = "engineering"	Name = "dev-web-server"
environment = "dev"	Owner = "john.doe@example.com"
	Environment = "development"
```

**Code:**

```hcl
# Example Terraform configuration with default_tags and per-resource tags

terraform {
  required_version = ">= 0.14"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.26"
    }
  }

  # Set default tags for all resources
  default_tags {
    department = "engineering"
    environment = "dev"
  }
}

resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  # Override default tags with per-resource tags
  tags = {
    Name        = "dev-web-server"
    Owner       = "john.doe@example.com"
    Environment = "development"
  }
}
```

**Example input / output:**

```text
Input:
terraform apply

Output:
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Resources:

- aws_instance.example: created

Outputs:

- default_tags.department: engineering
- default_tags.environment: dev
- per_resource_tags.Name: dev-web-server
- per_resource_tags.Owner: john.doe@example.com
- per_resource_tags.Environment: development
```

**Data:**

```json
{
  "default_tags": {
    "department": "engineering",
    "environment": "dev"
  },
  "per_resource_tags": {
    "Name": "dev-web-server",
    "Owner": "john.doe@example.com",
    "Environment": "development"
  }
}
```

**Flow:**

1. Yes
2. No
3. Define Default Tags
4. Are Resources Managed byTerraform?
5. Apply Per-Resource Tags
6. Tagging Complete
7. Manually Tag UnmanagedResources

