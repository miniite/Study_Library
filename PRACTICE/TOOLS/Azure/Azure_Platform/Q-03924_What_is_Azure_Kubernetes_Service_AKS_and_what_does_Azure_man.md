---
type: "practice"
difficulty: "Basic"
---

# What is Azure Kubernetes Service (AKS) and what does Azure manage for you versus what you manage?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ AKS and Container Platform Standards](<../../../../TOOLS/Azure/Azure_Platform/Azure_Functions_and_Cosmos_DB_Tuning/AKS_and_Container_Platform_Standards.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Azure Kubernetes Service (AKS) is a managed Kubernetes service by Microsoft that simplifies the deployment, management, and scaling of containerized applications on a Kubernetes cluster. Essentially, AKS allows you to run Kubernetes without having to worry about the underlying infrastructure setup and maintenance.

#### Triaging the Question

The question asks us to differentiate between what Azure manages for you in AKS and what we, as users, manage. This is important because understanding these roles helps in making informed decisions when deploying applications or managing clusters on AKS.

#### First Principles Solution

Azure's Management:

Networking and Security: Azure handles the network security of the Kubernetes cluster, including firewalls and ingress controllers.
Identity and Access Control: Azure manages the identity and access control for both the Kubernetes cluster and the underlying infrastructure.
Storage and Backup: It also takes care of storage solutions like Azure Blob Storage or Azure Files, and backup solutions to ensure data resilience.

User's Management:

Application Deployment and Configuration: Users are responsible for deploying applications, configuring their containers, and managing application state.
Scaling and Load Balancing: Although AKS can automatically scale based on demand, users need to configure this manually or use built-in tools like HPA (Horizontal Pod Autoscaler).
Monitoring and Logging: Users must implement monitoring and logging solutions to keep track of their applications' health and performance. Azure offers services like Application Insights for these purposes.
Security Policies and Compliance: Users are still responsible for securing the application code, secrets management, and ensuring compliance with organizational policies.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, while Azure manages much of the infrastructure setup and security, it's the user’s responsibility to manage application deployment, scaling, monitoring, and security at a more granular level. A common mistake is assuming that all management needs are handled by AKS without configuring necessary services.

#### Follow-ups
could include questions about specific scenarios like disaster recovery planning or integrating with other Azure services.

**Code:**

```bash
# Example of deploying a simple application on AKS using kubectl

kubectl apply -f https://raw.githubusercontent.com/Azure/kubernetes-app/main/deployment.yaml
```

