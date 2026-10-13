---
type: "practice"
difficulty: "Deep"
---

# You need deploys to stagger across AZs so a bad rollout can't hit all three failure domains at once; how would you make the CD system failure-domain-aware rather than relying on team discipline?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Standard Deployment Strategies](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Rollback_Strategy/Standard_Deployment_Strategies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that deployments in a CI/CD pipeline are staggered across availability zones (AZs), we need to build a robust, automated solution that leverages infrastructure as code and integrates with our existing CD system. The key idea is to decouple the deployment process from team discipline by automating it based on AZ awareness. This approach ensures reliability and minimizes the risk of a single point of failure impacting all availability zones simultaneously.

The core question here is how to make the CI/CD pipeline aware of the underlying infrastructure's AZs, ensuring that deployments are staggered across them. We'll reason from first principles by breaking down the problem into smaller steps: identifying the need for AZ-awareness, implementing a solution using infrastructure as code (IaC), and integrating this with our existing CD system.

First, let’s define the assumptions:

We have multiple environments, each replicated in different availability zones.
The deployment pipeline is already established but needs to be modified to include AZ awareness.
There are no external constraints beyond the current setup that would prevent implementing the solution (e.g., legacy infrastructure or complex interdependencies).
Step-by-Step Solution

Identify AZs and Deployment Stages:

Determine which environments need to be deployed in specific AZs.
Define deployment stages, where each stage is associated with a particular environment.

Infrastructure as Code (IaC):

Use IaC tools like Terraform or CloudFormation to define the infrastructure setup.
Create variables or parameters that can be dynamically set based on the current AZ and deployment phase.

CD System Integration:

Modify the CD system’s configuration to include logic that reads these variables or parameters.
Implement a step in the CD pipeline where it queries the current environment’s AZ information before initiating the deploy.

Stagger Deployment Logic:

Design the deployment script to pause and wait for confirmation from other zones before proceeding with the next phase of deployment.
Use conditional logic within the CD script to ensure that only one zone deploys at a time, or in sequence if needed.

Automated Testing and Rollback:

Integrate automated tests to verify each stage post-deployment.
Implement rollback mechanisms for any failed stages to ensure minimal impact on operations.

#### Closing Summary
and Common Mistakes

By automating the AZ-aware deployment process, we can significantly reduce the risk of a single bad rollout affecting all availability zones. This approach not only enhances reliability but also shifts responsibility from team discipline to automated processes, making the system more robust and maintainable over time.

#### Common mistakes
include underestimating the complexity of integrating IaC with existing CD systems or failing to properly test each stage before rolling out changes across multiple environments. It’s crucial to have a comprehensive testing strategy that accounts for both successful deployments and rollbacks in case something goes wrong.

**Comparison:**

```
AZ	Stage
us-west-1a	1
us-west-1b	2
us-west-1c	3
```

**Code:**

```python
# Example of a simple Python script to simulate AZ-aware deployment

import boto3
from time import sleep

def deploy_to_az(az):
    print(f"Deploying to AZ {az}")
    # Simulate actual deployment logic
    sleep(5)  # Simulating wait for deployment completion

def stagger_deployments():
    azs = ["us-west-1a", "us-west-1b", "us-west-1c"]

    for i, az in enumerate(azs):
        print(f"Deploying to {az} at stage {(i+1)} of 3")
        deploy_to_az(az)

stagger_deployments()
```

**Example input / output:**

```text
Input:
- Current environment variables: {"current_az_index": 0, "ami_id": "ami-0c55b159cbfafe1f0"}

Output:
Deploying to us-west-1a at stage 1 of 3
Deployment complete for us-west-1a

Deploying to us-west-1b at stage 2 of 3
Deployment complete for us-west-1b

Deploying to us-west-1c at stage 3 of 3
Deployment complete for us-west-1c
```

**Data:**

```yaml
# Example IaC configuration snippet using Terraform

resource "aws_instance" "example" {
  count = var.current_az_index + 1

  ami           = var.ami_id
  instance_type = "t2.micro"

  tags = {
    Name = "StaggeredDeploy-${var.current_environment}-${count.index}"
  }
}
```

**Flow:**

1. No
2. Success
3. Failure
4. Yes
5. Deployment Initiation
6. Is Current AZ Ready?
7. Wait for Next Stage
8. Deploy to Current AZ
9. Check Deployment Success
10. Proceed to Next Stage
11. Rollback Mechanism
12. Notify Team and Retry

