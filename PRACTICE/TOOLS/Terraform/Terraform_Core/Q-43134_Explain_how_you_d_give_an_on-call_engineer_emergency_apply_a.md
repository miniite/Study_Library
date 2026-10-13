---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd give an on-call engineer emergency apply access without them being able to read every secret in state day-to-day.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reviewing Plans for State Safety](<../../../../TOOLS/Terraform/Terraform_Core/Reviewing_Plans_for_State_Safety.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary: In this scenario, we need to provide an on-call engineer with the ability to perform emergency Terraform apply operations while ensuring they cannot access sensitive secrets stored in the Terraform state during their normal day-to-day activities.

#### Triage
The core of the question revolves around managing permissions and secret handling in a multi-user environment. We are required to ensure that an on-call engineer can execute critical operations but not have unrestricted access to sensitive data. This involves understanding how Terraform's state management works, specifically with modules, and leveraging AWS infrastructure to control access.

#### First-Principles Solution

Terraform State Management: Terraform uses a backend to store its state files, which can be encrypted using various backends like S3 or Vault.
S3 Bucket for State Storage: We will use an S3 bucket as the state backend and configure it with encryption at rest and in transit.
IAM Policies & Roles: Create specific IAM policies and roles to control access to the S3 bucket and Terraform operations.
Terraform Modules: Use modules to modularize our infrastructure, ensuring that sensitive data is stored outside of the codebase.

Let's break it down step-by-step:

S3 Bucket Configuration:

Create an S3 bucket for state storage with encryption enabled.
Configure the S3 bucket policy and IAM policies to allow read-only access to the on-call engineer during non-emergency times.

Terraform Backend Configuration:

HCL
terraform {
  backend "s3" {
    bucket = "<your-s3-bucket-name>"
    key    = "<path-to-your-state-file>"
    region = "<region>"
  }
}

IAM Roles and Policies:

Create a role for the on-call engineer that allows them to perform terraform apply in emergency situations.
JSON
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "<arn-of-your-s3-bucket>/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "iam:PassRole"
      ],
      "Resource": "*"
    }
  ]
}

Encrypted Secrets Management:

Use AWS Secrets Manager or a similar service to store and manage secrets.
Create a Terraform variable that references the ARN of the secret stored in Secrets Manager.

Module Usage:

Use modules to encapsulate sensitive logic, ensuring that the on-call engineer only has access to the necessary resources during emergencies.
HCL
module "example" {
  source  = "./modules/example"
  version = "~> 1.0"

  secret_arn = var.secret_arn
}

Dry-Run and Validation:

Perform a dry-run using the on-call engineer's credentials to ensure they can only perform emergency apply operations.

#### Closing Summary
By leveraging Terraform modules, S3 bucket policies, and IAM roles, we can provide an on-call engineer with the ability to execute critical operations during emergencies while ensuring that their day-to-day activities are restricted. This approach ensures both security and operational efficiency.

#### Common Mistakes/Follow-Ups

Forgetting to encrypt sensitive data at rest and in transit.
Not properly configuring IAM policies, leading to overly permissive access.
Failing to use modules effectively, which could expose sensitive logic during non-emergency operations.

**Comparison:**

```
Resource	Action
S3 Bucket	GetObject, PutObject
IAM Role	PassRole
```

**Code:**

```
Example Terraform backend configuration for S3 state storage

terraform { backend "s3" { bucket = "<your-s3-bucket-name>" key = "<path-to-your-state-file>" region = "<region>" } }

CODE
Copy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "<arn-of-your-s3-bucket>/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "iam:PassRole"
      ],
      "Resource": "*"
    }
  ]
}
HCL
Copy
module "example" {
  source  = "./modules/example"
  version = "~> 1.0"

  secret_arn = var.secret_arn
}
```

**Example input / output:**

```text
- On-call engineer credentials and IAM role setup.
- Terraform apply command during an emergency situation.
- Validation steps to ensure the on-call engineer can only perform emergency operations.
```

**Data:**

```text
- S3 bucket ARN: `<arn-of-your-s3-bucket>`
- IAM policy ARN: `<arn-of-your-iam-policy>`
- Terraform backend configuration for S3 state storage.
- Example of a module usage in the main Terraform file.
```

**Flow:**

1. On-Call Engineer
2. Assumes IAM Role withRestricted Permissions
3. S3 Read/Write Access for StateStorage
4. Terraform Apply EmergencyOperations
5. Encrypted Secrets in AWSSecrets Manager
6. Module Encapsulation ofSensitive Logic

