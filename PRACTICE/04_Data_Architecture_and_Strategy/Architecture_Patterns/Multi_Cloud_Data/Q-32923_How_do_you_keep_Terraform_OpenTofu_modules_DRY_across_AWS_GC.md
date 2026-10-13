---
type: "practice"
difficulty: "Deep"
---

# How do you keep Terraform/OpenTofu modules DRY across AWS/GCP/Azure when their primitives genuinely differ — where do you accept per-cloud modules versus forcing a leaky abstraction?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Multi-cloud Portability](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Multi_Cloud_Data/Multi-cloud_Portability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure Data Engineering practices remain DRY (Don't Repeat Yourself) across multi-cloud environments like AWS, GCP, and Azure using Terraform/OpenTofu, we need to find a balance between leveraging common abstractions while respecting the unique features each cloud provider offers. The key challenge lies in deciding when to create per-cloud modules versus forcing a single abstraction that might introduce unnecessary complexity or inefficiencies.

The fundamental question is: How do you manage the trade-offs between maintaining DRY principles and accommodating the specific nuances of different cloud providers? This involves understanding the capabilities, limitations, and best practices for each environment while ensuring your infrastructure code remains maintainable and scalable.

**Comparison:**

```
Cloud Provider	Configuration
AWS	Private ACL, Versioning enabled
GCP	Force Destroy Enabled, EU Region
Azure	Soft Delete Enabled, Retention Days 30
```

**Code:**

```python
# Example of a generic resource module that attempts to abstract differences
resource "generic_s3_bucket" {
  bucket = var.bucket_name

  # AWS-specific configuration (DRY principle violation)
  if aws_env == "aws" {
    acl         = "private"
    versioning { enabled = true }
  }

  # GCP-specific configuration
  if gcp_env == "gcp" {
    force_destroy = true
    location      = "EU"
  }

  # Azure-specific configuration
  if azure_env == "azure" {
    soft_delete_enabled = true
    retention_days      = 30
  }
}
```

**Example input / output:**

```

Given the input data, this generic module would not be DRY across clouds due to conditionals that force cloud-specific configurations. A better approach involves creating per-cloud modules and using a central configuration layer.

PYTHON
Copy
# Centralized Configuration Layer
module "s3_bucket_aws" {
  source = "./modules/s3/terraform-aws"
}

module "s3_bucket_gcp" {
  source = "./modules/s3/terraform-gcp"
}

module "s3_bucket_azure" {
  source = "./modules/s3/terraform-azure"
}
```

**Data:**

```json
{
  "aws_env": "aws",
  "gcp_env": "gcp",
  "azure_env": "azure",
  "bucket_name": "example-bucket"
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Centralized ConfigurationLayer
6. Is it AWS?
7. Invoke AWS-Specific Module
8. Is it GCP?
9. Invoke GCP-Specific Module
10. Invoke Azure-Specific Module

