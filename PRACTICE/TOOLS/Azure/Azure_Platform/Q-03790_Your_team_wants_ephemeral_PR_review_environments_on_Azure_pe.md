---
type: "practice"
difficulty: "Deep"
---

# Your team wants ephemeral PR review environments on Azure per pull request; how would you build and tear them down cost-effectively?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Azure Pipelines and Release Approvals](<../../../../TOOLS/Azure/Azure_Platform/Azure_DevOps_and_Artifact_Governance/Azure_Pipelines_and_Release_Approvals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address your team's requirement for ephemeral PR review environments on Azure, we need to ensure that these environments are created cost-effectively and can be torn down automatically when no longer needed. The key challenge here is balancing the need for agility in development with minimizing cloud costs. We'll explore a solution using Azure DevOps services, such as Pipelines and AKS (Azure Kubernetes Service), along with cost management tools like Azure Cost Management and automated lifecycle management through Azure Resource Manager templates.

First, let's break down what the question is asking:

Provisioning: Create environments that can be spun up quickly for PR reviews.
Cost Control: Minimize costs by ensuring these environments are torn down when not in use.
Automation: Automate the lifecycle management to reduce manual overhead and ensure consistency.

#### Triaging the Question

The question is probing on how we can efficiently manage temporary resources while adhering to cost constraints. We need to consider:

Resource Lifecycle Management: Ensuring that resources are created only when needed and destroyed automatically after a period.
Cost Optimization: Using tools like Azure Cost Management for budget alerts and optimization suggestions.
Automation: Implementing CI/CD practices to automate the creation and destruction of environments.

#### Solution
from First Principles

CI/CD Pipeline Setup:

We can use Azure DevOps Pipelines to trigger the provisioning of Kubernetes clusters on AKS whenever a PR is opened.
The pipeline will use a YAML template that defines the necessary resources (e.g., AKS cluster, storage accounts) and configures them with specific environment settings.

Environment Customization:

Each environment can be customized to match the requirements of the PR being reviewed. This includes setting up databases, deploying code, and configuring networking.
We use a templating mechanism (e.g., Helm charts or Terraform) to ensure consistency across environments while allowing for configuration overrides.

Resource Lifecycle Management:

Implement a lifecycle management strategy using Azure Resource Manager templates that define the resources and their cleanup policies.
Use Azure's az CLI commands within the pipeline to create, update, and delete these resources as needed.

Cost Control Measures:

Set up budget alerts in Azure Cost Management to monitor spending on PR environments.
Implement a cost optimization strategy that automatically scales down or terminates unused AKS clusters based on usage patterns.
Use reserved instances for critical services to save costs where possible.

Security and Compliance:

Ensure that the environments adhere to security best practices, such as using managed identities, securing network access, and applying role-based access control (RBAC).
Regularly audit and update the environment configurations to ensure compliance with internal policies.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, we can achieve cost-effective ephemeral PR review environments by leveraging Azure DevOps Pipelines, AKS, and resource lifecycle management tools. The key is to automate the creation and destruction of resources while ensuring they are configured correctly for each PR. Common mistakes include:

Failing to properly manage costs through budget alerts and optimization strategies.
Not using automation extensively, leading to increased manual overhead.
Ignoring security best practices, which can lead to compliance issues.

For follow-ups, we could discuss specific tools like Azure DevOps Pipelines vs. GitLab CI/CD, or explore different Kubernetes cluster management options such as AKS managed nodes versus self-managed clusters for greater control and customization.

**Comparison:**

```
Resource	Management Strategy
AKS Cluster	Auto-scaled and terminated based on usage
Storage Account	Cleaned up after PR is closed
```

**Code:**

```yaml
# Example YAML pipeline template
stages:
  - stage: SetupEnvironment
    jobs:
      - job: CreateAKSEnvironment
        steps:
          - task: AzureCLI@2
            inputs:
              azureSubscription: '<subscription>'
              scriptType: 'bash'
              scriptLocation: 'inlineScript'
              inlineScript: |
                az group create --name myResourceGroup --location eastus
                az aks create ...
```

**Example input / output:**

```

Input:

YAML
Copy
# PR environment setup request
name: myFeatureBranchPR
branch: feature/new-feature
environmentSettings: {}

Output:

JSON
Copy
{
  "aksClusterId": "/subscriptions/<subscription-id>/resourceGroups/myResourceGroup/providers/Microsoft.ContainerService/managedClusters/myAKSCluster",
  "podLabels": {
    "env": "pr-1234"
  },
  "storageAccountName": "<unique-storage-account-name>"
}
```

**Data:**

```json
{
  "resourceGroupName": "myResourceGroup",
  "aksClusterName": "myAKSCluster",
  "podLabels": {
    "env": "pr-1234"
  }
}
```

**Flow:**

1. Yes
2. No
3. PR Opened
4. Is Environment Needed?
5. Pipeline Triggers Setup
6. Azure CLI Commands CreateResources
7. Cleanup After PR Closed
8. Monitor Cost and Scale Down
9. Environment Ready/CleanupComplete

