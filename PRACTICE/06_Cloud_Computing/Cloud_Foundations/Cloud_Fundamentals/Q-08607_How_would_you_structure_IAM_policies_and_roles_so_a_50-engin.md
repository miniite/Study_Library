---
type: "practice"
difficulty: "Deep"
---

# How would you structure IAM policies and roles so a 50-engineer org has least-privilege access without drowning in per-person policy churn?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Identity and Access Foundations](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Identity_and_Access_Foundations/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

When structuring IAM policies and roles for a 50-engineer organization, our primary goal is to minimize privilege sprawl while ensuring compliance with least-privilege principles. We achieve this by leveraging role-based access control (RBAC) and a clear hierarchy of roles that align with team responsibilities. This approach ensures that each engineer has the necessary permissions without unnecessary exposure.

#### Triaging the Question

The question asks how to structure IAM policies and roles in an organization with 50 engineers, aiming for least-privilege access while avoiding excessive policy churn. The interviewer is probing understanding of RBAC principles, role hierarchies, permission management, and best practices for scaling IAM policies.

#### Assumptions
and Key Concepts
Organizational Hierarchy: Teams are well-defined with clear roles.
Least Privilege Principle: Engineers should only have the permissions required to perform their job functions.
RBAC Framework: Role-based access control is used to manage privileges.
Scalability: The solution needs to be scalable for 50 engineers without manual intervention.
Step-by-Step Solution
Step 1: Define Roles Based on Teams and Responsibilities

First, identify the main teams in the organization and define roles based on their responsibilities:

DevOps Team: Engineers responsible for infrastructure setup and maintenance.
Developers (Frontend/Backend): Developers building and maintaining applications.
Product Managers: Overseeing product development and release cycles.
Quality Assurance: Testing and validating application functionality.

For each team, create a role with appropriate permissions:

DevOps
Read Infrastructure
Manage Infrastructure
Developers (Frontend)
Developers (Backend)
Quality Assurance
Step 2: Create Permissions for Each Role

Define the permissions required by each role:

DevOps: read_infrastructure, manage_infrastructure
Developers (Frontend): write_frontend_code, access_frontend_resources
Developers (Backend): write_backend_code, access_backend_resources
Quality Assurance: test_applications, report_issues
Step 3: Assign Roles to Engineers

Assign roles based on engineers' responsibilities:

Engineer1
Developer (Frontend)
DevOps
Engineer2
Developer (Backend)
Quality Assurance
Step 4: Implement Conditional Permissions

Implement conditional permissions to further refine access based on specific use cases:

Access Control Lists (ACLs) for sensitive resources.
Service Accounts with limited permissions that can be shared among team members.
Step 5: Automate Policy Management

To avoid policy churn, automate the management of IAM policies using tools like AWS CloudFormation or Terraform. This ensures consistent and scalable deployment:

HCL
resource "aws_iam_role" "devops" {
  name = "DevOpsRole"

  assume_role_policy_json = jsonencode({
    Version = "2012-10-17",
    Statement = [
      {
        Action = "sts:AssumeRole",
        Effect = "Allow",
        Principal = {
          Service = ["ec2.amazonaws.com"]
        }
      }
    ]
  })

  policy {
    path         = "/"
    description  = "Allows reading and managing infrastructure"
    policy_name  = "DevOpsPolicy"
    document     = jsonencode({
      Version = "2012-10-17",
      Statement = [
        {
          Action = ["ec2:Describe*", "s3:Get*", "iam:Get*"],
          Effect = "Allow",
          Resource = "*"
        },
        {
          Action = ["ec2:Create*", "ec2:Delete*", "ec2:Modify*"],
          Effect = "Allow",
          Resource = "*"
        }
      ]
    })
  }
}
Step 6: Monitor and Audit

Implement continuous monitoring and auditing to ensure compliance with least-privilege policies:

JSON
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AuditIAMPolicy",
      "Effect": "Allow",
      "Action": ["iam:GetUser*", "iam:List*"],
      "Resource": "*"
    }
  ]
}

#### Closing Summary

By structuring IAM policies and roles based on clear team responsibilities, implementing conditional permissions, automating policy management, and continuously monitoring, we can achieve least-privilege access for a 50-engineer organization. This approach minimizes manual intervention while ensuring compliance.

#### Common Mistakes/Follow-Ups
Overlapping Roles: Engineers should not have overlapping permissions unless necessary.
Manual Policy Management: Automating policy management to avoid human error and churn.
Excessive Permissions: Ensuring that permissions are granular and least-privilege.
Regular Audits: Regularly reviewing and updating policies to maintain security.

**Comparison:**

```
Role	Permissions
DevOps	Read & Manage Infrastructure
Frontend	Write & Access Frontend Code
Backend	Write & Access Backend Code
QA	Test Applications, Report Issues
```

**Code:**

```hcl
resource "aws_iam_role" "devops" {
  name = "DevOpsRole"

  assume_role_policy_json = jsonencode({
    Version = "2012-10-17",
    Statement = [
      {
        Action = "sts:AssumeRole",
        Effect = "Allow",
        Principal = {
          Service = ["ec2.amazonaws.com"]
        }
      }
    ]
  })

  policy {
    path         = "/"
    description  = "Allows reading and managing infrastructure"
    policy_name  = "DevOpsPolicy"
    document     = jsonencode({
      Version = "2012-10-17",
      Statement = [
        {
          Action = ["ec2:Describe*", "s3:Get*", "iam:Get*"],
          Effect = "Allow",
          Resource = "*"
        },
        {
          Action = ["ec2:Create*", "ec2:Delete*", "ec2:Modify*"],
          Effect = "Allow",
          Resource = "*"
        }
      ]
    })
  }
}
```

**Example input / output:**

```text
Input:
- Engineers assigned to DevOps, Developers (Frontend/Backend), and Quality Assurance teams.
- Conditional permissions for sensitive resources.

Output:
- Automated role assignments using AWS CloudFormation or Terraform.
- Continuous monitoring and auditing of IAM policies.
```

**Data:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AuditIAMPolicy",
      "Effect": "Allow",
      "Action": ["iam:GetUser*", "iam:List*"],
      "Resource": "*"
    }
  ]
}
```

**Flow:**

1. Define Teams
2. Create Roles
3. Assign Permissions to Roles
4. Automate Policy Management
5. Monitor and Audit Policies

