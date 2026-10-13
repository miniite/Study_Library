---
type: "practice"
difficulty: "Deep"
---

# An AKS upgrade to a new Kubernetes version broke an admission webhook; how do you plan and stage cluster upgrades to catch API deprecations safely?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ AKS and Container Platform Standards](<../../../../TOOLS/Azure/Azure_Platform/Azure_Functions_and_Cosmos_DB_Tuning/AKS_and_Container_Platform_Standards.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When planning and staging cluster upgrades in Azure Kubernetes Service (AKS), especially with admission webhooks, it’s crucial to take a strategic approach that ensures smooth transitions while minimizing risks. The key steps involve understanding the upgrade process, validating APIs, and creating comprehensive testing environments.

Firstly, understand that an AKS cluster is managed by Microsoft, but upgrading involves changes in the underlying Kubernetes versions, which might impact custom admission webhooks. This means we need to be vigilant about API deprecations and ensure our webhook logic is robust enough to handle any breaking changes.

To plan and stage these upgrades effectively:

Identify Critical Components: Determine which parts of your application rely on specific APIs or webhooks.
API Validation: Use tools like the Kubernetes API documentation and the kubectl api-versions command to understand what APIs are supported in different versions.
Setup Staging Environments: Create a staging AKS cluster that mirrors production as closely as possible, including all custom webhooks and configurations.
Continuous Integration/Deployment (CI/CD): Integrate these environments into your CI/CD pipeline so you can automatically test changes before applying them to production.

Now let's break it down step by step:

Step-by-Step Solution
Identify Critical Components: Start by listing all the custom resources and webhooks that are critical for your application’s functionality.
API Validation: Review the Kubernetes API version matrix in the official documentation to see what APIs will be available post-upgrade.
Setup Staging Environment:
Create a new AKS cluster with the target Kubernetes version.
Deploy all necessary custom resources and webhooks into this environment.
Continuous Integration/Deployment (CI/CD): Configure your CI/CD pipeline to run tests against the staging cluster before applying changes to production.
Manual Testing: Perform manual testing on both the staging and production clusters to ensure that the upgraded webhook works as expected without breaking anything.

#### Closing Summary

By taking a staged approach, validating APIs, setting up comprehensive testing environments, and integrating these into your CI/CD pipeline, you can effectively manage AKS upgrades while ensuring minimal disruptions. This strategic planning helps catch API deprecations early, reducing the risk of downtime or functionality issues in production.

#### Common Mistakes/Follow-Ups

Forgetting to update webhook logic when APIs change.
Not having a staging environment that fully mirrors production.
Overlooking manual testing phases which can reveal edge cases not caught by automated tests.

**Comparison:**

```
Stage	Action	Notes
Pre-Upgrade	Review API versions & custom resources.	Ensure compatibility with target version.
Staging Cluster Setup	Create new AKS cluster, deploy resources & webhooks.	Mirrors production environment closely.
CI/CD Pipeline Integration	Run automated tests in staging.	Detect issues before applying to production.
Manual Testing	Perform manual testing on both clusters.	Catch edge cases not covered by automation.
```

**Code:**

```shell
# Example command to check available Kubernetes versions in AKS
az aks get-versions --location <your_location> | jq '.orchestrators'
```

**Example input / output:**

```text
# Example input->output for staging environment setup
Input: Create a new AKS cluster with Kubernetes version 1.25.x.
Output: New AKS cluster is created and configured with all necessary custom resources and webhooks deployed.

Input: Run tests against the staging cluster.
Output: Tests pass, confirming that the upgraded webhook works as expected without breaking anything.
```

**Data:**

```yaml
# Sample YAML for deploying custom webhooks in a staging cluster
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: example-webhook-configuration
webhooks:
- name: example.webhook.example.com
  rules:
  - operations: [ "CREATE", "UPDATE" ]
    apiGroups: [ "" ] # all API groups are allowed
    apiVersions: [ "*" ]
    resources: [ "<your_custom_resource>" ]
  clientConfig:
    service:
      namespace: <namespace>
      name: example-webhook-service
      path: /webhook
```

**Flow:**

1. Identify Critical Components
2. Review API Versions
3. Setup Staging Cluster
4. Integrate CI/CD Pipeline
5. Manual Testing on Staging andProd Clusters
6. Apply Changes to Production

