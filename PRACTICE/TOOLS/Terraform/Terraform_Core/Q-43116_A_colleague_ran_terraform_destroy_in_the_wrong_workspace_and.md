---
type: "practice"
difficulty: "Deep"
---

# A colleague ran terraform destroy in the wrong workspace and wiped prod. What guardrails (backend policy, CI gates, prevent_destroy) would have stopped it?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reviewing Plans for State Safety](<../../../../TOOLS/Terraform/Terraform_Core/Reviewing_Plans_for_State_Safety.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a DevOps engineer accidentally runs terraform destroy in the wrong workspace, particularly in production, it can lead to catastrophic data loss and downtime. To prevent such incidents, we need to implement multiple layers of guardrails including backend policy management, CI/CD pipeline gates, and the use of prevent_destroy. Let's dive into each of these strategies.

Firstly, the backend policy is crucial for managing state storage securely. This ensures that no one can accidentally or maliciously modify the state file without proper authorization. Secondly, implementing CI/CD gate mechanisms will ensure that any change to production infrastructure requires additional approvals and verification steps. Lastly, using prevent_destroy in Terraform modules provides a way to prevent accidental destruction of critical resources.

**Code:**

```hcl
# Example of backend policy in Terraform
terraform {
  backend "s3" {
    bucket = "my-terraform-state-bucket"
    key    = "production.tfstate"
    region = "us-west-2"
  }
}

# Backend Policy
aws_s3_bucket_policy my_state_bucket_policy {
  bucket = aws_s3_bucket.my_state.id

  policy = <<POLICY
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowReadFromIAMRole",
      "Effect": "Allow",
      "Principal": {
        "AWS": [
          "arn:aws:iam::123456789012:user/devops_user"
        ]
      },
      "Action": [
        "s3:GetObject"
      ],
      "Resource": [
        "arn:aws:s3:::my-terraform-state-bucket/production.tfstate"
      ]
    }
  ]
}
POLICY
}
```

**Flow:**

1. Yes
2. No
3. Workspace Selection
4. Is the Workspace Prod?
5. Run Terraform Destroy
6. Monitor for Destruction
7. Notify DevOps Team
8. Proceed with Deployment
9. Review and Approve in CI/CDPipeline
10. Execute Terraform Apply
11. Verify State Updates
12. Log Changes
13. Monitor for Issues

