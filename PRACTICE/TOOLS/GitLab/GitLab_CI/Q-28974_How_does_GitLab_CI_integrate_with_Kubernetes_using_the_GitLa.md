---
type: "practice"
difficulty: "Deep"
---

# How does GitLab CI integrate with Kubernetes using the GitLab agent or Auto DevOps?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ GitLab CI Fundamentals](<../../../../TOOLS/GitLab/GitLab_CI/GitLab_CI_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary: In this answer, we will discuss how GitLab CI integrates with Kubernetes through two primary methods: the GitLab Agent and Auto DevOps. We'll explore the key components, configuration steps, and practical use cases to ensure a deep understanding.

GitLab CI is a robust Continuous Integration/Continuous Deployment (CI/CD) tool that seamlessly integrates with Kubernetes for automated deployment pipelines. This integration leverages either the GitLab Agent or Auto DevOps to manage the lifecycle of applications deployed on Kubernetes clusters. The focus will be on explaining how these tools work under the hood, their setup processes, and best practices.

#### Triage
The question asks about integrating GitLab CI with Kubernetes using two methods: the GitLab Agent and Auto DevOps. The core idea is to understand the underlying mechanisms that facilitate this integration, including configuration and practical applications. We will assume a basic understanding of both GitLab CI and Kubernetes.

GitLab Agent: This agent runs on the nodes where your application pods are deployed. It communicates with the GitLab server to fetch CI/CD jobs and execute them within the cluster.
Auto DevOps: A predefined set of GitLab CI pipelines that automate the entire software development lifecycle, including build, test, and deployment.

Step-by-Step Solution:

GitLab Agent Integration

Prerequisites: Ensure you have a Kubernetes cluster and are familiar with basic Kubernetes concepts.
Setup:
Install the GitLab Agent on your Kubernetes nodes using Helm or by manually deploying the agent in a pod.
Configure the agent to communicate with the GitLab server via an API token.
Define CI/CD jobs that can run directly within the cluster, leveraging Kubernetes resources and capabilities.
BASH
# Example of installing GitLab Agent using Helm
helm repo add gitlab https://charts.gitlab.io
helm install my-agent gitlab/gitlab-agent --set agent.serviceToken=my-token

Auto DevOps Integration

Prerequisites: Ensure your project is hosted on GitLab and that Auto DevOps is enabled.
Configuration:
Enable Auto DevOps for your GitLab project by setting the CI.AUTO variable to true.
Define a Kubernetes cluster resource in GitLab CI/CD settings, providing details like API URL, token, and namespace.
YAML
# Example of enabling Auto DevOps in GitLab settings
variables:
  CI_AUTO: "true"
  KUBERNETES_API_URL: "<KUBE_APISERVER_URL>"
  KUBERNETES_TOKEN: "<API_TOKEN>"

Practical Use Cases:

Build and Test: Auto DevOps automatically triggers builds, tests, and lints your codebase.
Deployments: Deploy the application to a Kubernetes cluster using Helm charts or custom scripts.
Monitoring and Logging: Integrate with monitoring tools like Prometheus and Grafana for real-time insights.

#### Common Mistakes

Failing to configure API tokens securely.
Not properly setting up network policies in Kubernetes, leading to connectivity issues between the GitLab agent and cluster.
Ignoring security best practices during deployment automation.

#### Closing Summary
GitLab CI integrates with Kubernetes through the GitLab Agent or Auto DevOps. The agent approach requires manual setup but offers flexibility, while Auto DevOps provides a fully automated pipeline for continuous delivery. Both methods leverage Kubernetes resources to ensure efficient and secure deployments.

**Code:**

```
Example of installing GitLab Agent using Helm

helm repo add gitlab https://charts.gitlab.io helm install my-agent gitlab/gitlab-agent --set agent.serviceToken=my-token

CODE
Copy

```yaml
# Example of enabling Auto DevOps in GitLab settings
variables:
  CI_AUTO: "true"
  KUBERNETES_API_URL: "<KUBE_APISERVER_URL>"
  KUBERNETES_TOKEN: "<API_TOKEN>"
```

**Flow:**

1. Yes
2. No
3. GitLab Project
4. Enable Auto DevOps?
5. Trigger CI/CD
6. Build and Test
7. Deploy to Kubernetes
8. GITLAB AGENT Setup
9. Kubernetes Node Install Agent
10. Configure API Token
11. Docker Build & Push
12. Kubernetes Deployment

