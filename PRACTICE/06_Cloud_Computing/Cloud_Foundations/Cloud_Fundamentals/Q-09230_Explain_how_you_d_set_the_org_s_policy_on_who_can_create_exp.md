---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd set the org's policy on who can create expensive cloud resources, so a junior engineer can move fast but cannot accidentally spin up a six-figure GPU cluster or a global database.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Identity and Access Foundations](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Identity_and_Access_Foundations/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that junior engineers can move quickly while avoiding accidental creation of high-cost cloud resources, we need to establish a robust yet flexible policy framework. The goal is to balance developer productivity with resource management. This involves setting up appropriate IAM roles, using cost-control features, and implementing governance tools.

The core idea is to define policies based on the principle of least privilege, ensuring that each user has only the permissions they need. We'll leverage AWS CloudFormation templates, AWS Policy Generator, and AWS Cost Explorer to enforce these policies effectively. Here’s a step-by-step approach:

Identify High-Cost Resources: List all cloud resources with high costs (e.g., GPU clusters, global databases). For this example, let's assume GPUs cost $50 per hour.
Define IAM Roles and Policies: Create fine-grained roles for junior engineers that allow them to perform basic tasks but restrict access to creating expensive resources.
Implement Cost Control Services: Use AWS Budgets and AWS Cost Explorer to monitor and alert on unexpected costs.
Use Service Catalog for Templates: Implement a Service Catalog with pre-approved templates, ensuring only necessary resources are created.

**Comparison:**

```
Role	Permissions	Cost Restrictions
Junior Dev	Basic EC2, S3	Deny GPU, Global DB
Senior Dev	All services	None
```

**Code:**

```python
# Example IAM Policy Definition (Python Boto3)
import boto3

iam = boto3.client('iam')

# Create a policy that denies creation of high-cost resources
policy_document = {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": [
                "ec2:RunInstances",
                "rds:CreateGlobalDatabase"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:RequestTag/COST_CONTROL_TAG": ["GPU", "GLOBAL_DATABASE"]
                }
            }
        }
    ]
}

iam.create_policy(
    PolicyName='DenyHighCostResources',
    PolicyDocument=json.dumps(policy_document)
)
```

**Example input / output:**

```
Input: A junior engineer tries to create a GPU instance without the required tags.
Output: The request is denied due to the enforced IAM policy.
PLAINTEXT
Copy
User: Attempted to run an instance with tags [COST_CONTROL_TAG: "GPU"]
Outcome: Request denied by IAM policy 'DenyHighCostResources'
```

**Data:**

```json
{
  "tags": {
    "COST_CONTROL_TAG": ["GPU", "GLOBAL_DATABASE"]
  },
  "services": [
    {"name": "ec2:RunInstances"},
    {"name": "rds:CreateGlobalDatabase"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. User Initiates Request
6. Is the request taggedcorrectly?
7. Resource Created
8. Does the user havepermission?
9. Request Denied

