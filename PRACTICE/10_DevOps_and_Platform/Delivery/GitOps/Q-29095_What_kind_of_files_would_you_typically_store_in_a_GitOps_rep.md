---
type: "practice"
difficulty: "Basic"
---

# What kind of files would you typically store in a GitOps repository?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Repo and Environment Layout](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Repo_and_Environment_Layout/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary: In a GitOps setup, you'll find configuration and infrastructure as code (IaC) files stored in your GitOps repository. These files ensure that your production environment matches the desired state defined by your DevOps team.

#### Triage
The question is asking about the types of files commonly found in a GitOps repository. The interviewer wants to see if you understand what GitOps entails and how it maintains consistency between development, staging, and production environments.

Step-by-Step Solution:

Configuration Files: These are YAML or JSON files that define various aspects like deployment configurations, service settings, and monitoring rules.
Infrastructure as Code (IaC) Files: Tools like Terraform, Ansible, or Kubernetes manifests are used to manage infrastructure. These files automate the provisioning of resources in a consistent manner.
Kubernetes YAML/JSON Files: If you're working with containerized applications, these manifest files define pods, services, and deployments that get deployed to your clusters.
Policies and Compliance Files: You might include files that enforce security policies or compliance requirements.
Charts for Helm Repositories: For Kubernetes users, Helm charts are used to manage complex application deployments.

#### Closing Summary
In essence, a GitOps repository contains the necessary scripts, configurations, and IaC files required to maintain a consistent environment across different stages of deployment. This setup ensures that changes are version-controlled and reproducible, aligning with best DevOps practices.

#### Common Mistakes/Follow-Ups

Mistake: Ignoring non-code related configuration files.

#### Follow-Up
The interviewer might ask about specific tools used in GitOps or how to ensure the security of your repository.

**Comparison:**

```
Type	Example File
Config	example-configmap.yaml
IaC	main.tf
Helm Chart	chart.yaml
Policy	security-policy.json
```

**Code:**

```yaml
# Example Kubernetes Deployment YAML file
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-deployment
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: example-app
    spec:
      containers:
      - name: example-container
        image: example-image:latest
```

**Example input / output:**

```text
Input:
- Deploy a new version of the application using Kubernetes YAML files.
Output:
- Updated deployment in the cluster with zero downtime, as changes are applied through automated rollouts managed by GitOps tools.
```

**Data:**

```json
{
  "configMap": {
    "name": "example-config",
    "data": {
      "db_connection_string": "mysql://user:pass@localhost/db"
    }
  },
  "secret": {
    "name": "example-secret",
    "data": {
      "api_key": "base64encodedapikey"
    }
  }
}
```

**Flow:**

1. Repository
2. Configuration Files
3. IaC Files (Terraform/Ansible)
4. Kubernetes YAML/JSON
5. Policies & Compliance

