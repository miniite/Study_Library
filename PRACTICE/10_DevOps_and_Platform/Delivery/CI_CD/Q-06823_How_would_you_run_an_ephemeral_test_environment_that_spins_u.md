---
type: "practice"
difficulty: "Deep"
---

# How would you run an ephemeral test environment that spins up on PR open and tears down on PR close?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Ephemeral Environments](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Ephemeral_Environments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the need of running an ephemeral test environment that spins up when a pull request (PR) is opened and tears down upon its closure, we must design a CI/CD pipeline that integrates seamlessly with GitHub's webhook system. This involves setting up a continuous integration (CI) process to automatically trigger on PR events and a deployment strategy that ensures the test environment is promptly dismantled once the PR merge or close operation is completed.

#### Triaging the Question

The core of this question revolves around automating the lifecycle management of environments in response to GitHub's webhook notifications. We need to handle two main operations:

Environment Provisioning: Automatically start a temporary test environment when a PR is opened.
Environment Teardown: Shut down the environment once the PR is closed or merged.

#### First-Principles Solution
Step 1: Trigger on GitHub Webhook Events

We leverage GitHub's webhook system to detect when a PR opens and closes. The webhooks will notify our CI/CD server via HTTP requests, which we can then process using cloud functions or scripts to trigger the necessary actions.

Step 2: Environment Provisioning

For provisioning an environment, we use infrastructure as code (IaC) tools like Terraform, CloudFormation, or Kubernetes manifests. These IaC templates are parameterized with unique identifiers for each PR, ensuring that each PR gets a distinct and isolated test setup.

Step 3: Environment Teardown

Upon receiving notifications about PR closures, we use the same IaC tool to destroy the infrastructure created during provisioning. This ensures that resources are cleaned up promptly, reducing costs and preventing accidental leaks of sensitive information or services.

Detailed Implementation Steps

Set Up Webhooks in GitHub

Configure your repository's webhook settings to send push events only when a PR is opened or closed.
The payload will contain details about the PR, such as its ID, status, and actions taken (open/closed).

Create CI/CD Pipeline Triggers

Use GitHub Actions or GitLab CI to set up workflows that listen for webhook triggers.
Define specific jobs within these workflows to handle environment provisioning and teardown.

Provisioning Script with IaC Tools

Write a script (e.g., using Bash, Python) that interacts with your chosen IaC tool (Terraform, CloudFormation).
This script will read the PR information from the webhook payload and use it as input to create or update an environment.

Teardown Script with IaC Tools

Similarly, write a teardown script that destroys the environment using the same IaC tool.
Ensure this script is idempotent and can handle cases where no active PR exists for the environment.
Sample Workflow
PR Opened
Trigger CI/CD Pipeline
Provisioning Script with IaC(e.g., Terraform)
Environment Ready
Wait for PR Closure
Trigger CI/CD Pipeline
Teardown Script with IaC(Terraform)
Environment Teardown

#### Closing Summary
and Common Mistakes

To successfully implement this solution, ensure you:

Correctly configure webhooks in GitHub to capture PR lifecycle events.
Design your CI/CD pipeline to handle both provisioning and teardown workflows efficiently.
Use IaC tools effectively to manage stateful resources like networks and instances.

Common pitfalls include:

Overlooking the need for robust error handling in both provisioning and teardown scripts.
Not securing sensitive information such as API keys or environment variables used by your CI/CD infrastructure.
Failing to consider resource cleanup policies that could lead to billing issues if environments are not properly torn down.

**Code:**

```
Example provisioning script (Terraform)

terraform { required_providers { terraform = { source = "hashicorp/aws" } } }

resource "aws_ec2_instance" "test_instance" { ami = "ami-0c55b159cbfafe1f0" instance_type = "t2.micro"

tags = { Name = "pr-${var.pr_id}" } }

CODE
Copy

```bash
# Example teardown script (Terraform)
terraform {
  required_providers {
    terraform = { source = "hashicorp/aws" }
  }
}

resource "aws_ec2_instance" "test_instance" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name = "pr-${var.pr_id}"
  }

  lifecycle {
    ignore_changes = [tags]
  }
}
```

**Example input / output:**

```text
Input:
- GitHub Webhook payload indicating PR state change.

Output:
- CI/CD pipeline triggered to either provision or teardown an environment based on the PR action.
```

**Data:**

```json
{
  "webhook_payload": {
    "action": "opened",
    "number": 12345,
    "pull_request": {
      "title": "Implement feature XYZ",
      "head": { "repo": "username/repo" },
      "base": { "repo": "main-repo" }
    }
  }
}
```

**Flow:**

1. PR Opened
2. Trigger CI/CD Pipeline
3. Provisioning Script with IaC(e.g., Terraform)
4. Environment Ready
5. Wait for PR Closure
6. Trigger CI/CD Pipeline
7. Teardown Script with IaC(Terraform)
8. Environment Teardown

