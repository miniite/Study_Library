---
type: "practice"
difficulty: "Basic"
---

# What is an Azure subscription and how does it relate to resource groups and management groups?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Subscriptions and Resource Hierarchy](<../../../../TOOLS/Azure/Azure_Platform/Subscriptions_and_Resource_Hierarchy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

An Azure subscription is a billing entity that enables you to manage your Azure resources, with each subscription being separate in terms of billing, policy enforcement, and management. Resource groups are collections of Azure resources grouped together to simplify deployment, management, and cost control. Management groups help in organizing multiple subscriptions into a hierarchy for centralized governance.

#### Triaging the Question

The question is asking you to explain three key concepts in Azure: what an Azure subscription is, how it relates to resource groups, and the role of management groups. The interviewer is probing your understanding of the hierarchical structure of Azure resources and how these structures help in managing them effectively.

#### First-Principles Solution
1. Azure Subscription:

An Azure subscription is a billing entity that allows you to create, manage, and monitor Azure resources within an organization. Each subscription is distinct from others, meaning they have separate budgets, policies, and access controls. Typically, a user or administrator can have multiple subscriptions to manage different projects or teams.

2. Resource Groups:

A resource group in Azure is a container that holds related resources for an application or project. It allows you to organize your resources logically, making it easier to deploy, manage, and delete them as a unit. Resource groups are typically used to isolate resources by environment (e.g., development, staging, production) or by business units.

3. Management Groups:

Management groups provide a way to organize multiple subscriptions into hierarchies for centralized governance. They help in managing policies, access control, and compliance across different subscriptions without the need to create separate Azure AD tenants. Management groups can be nested within each other, allowing you to create a hierarchical structure that reflects your organizational needs.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

In summary, an Azure subscription is a billing entity, resource groups are containers for organizing resources, and management groups help in centralizing governance across multiple subscriptions. A common mistake beginners make is not understanding the separation between these entities; it’s essential to ensure that each layer (subscription, resource group, and management group) serves its purpose correctly.

Common follow-ups could include:

How do you decide which subscription to use for a specific project?
Can you move resources across different subscriptions within the same Azure account?
What are some best practices when using management groups?
Triaging Further:
Subscriptions: Often, projects or teams might require separate billing, so each would have its own subscription.
Resource Groups: Typically used for organizing related services and making deployments more efficient.
Management Groups: Helpful for large organizations that need centralized governance without the complexity of multiple Azure AD tenants.

#### Example Scenario

Imagine a software development company with three teams: Frontend, Backend, and DevOps. Each team works on separate projects but needs access to similar resources like storage accounts or virtual machines. You might create one subscription per team and use resource groups within each subscription for specific project components (e.g., frontend-resources, backend-resources).

#### Common Mistakes
Overcomplicating the structure by creating too many subscriptions and management groups.
Not leveraging resource groups effectively to manage related resources.

**Comparison:**

| Subscription | Resource Groups |
|--------------|----------------|
| DevOps       | frontend-resources, backend-resources |
| Backend      | backend-resources          |
| Frontend     | frontend-resources        |

**Code:**

```
MERMAID
Copy
graph TD;
    Subscription1((Subscription 1)) --> ResourceGroup1(DevOps Resources);
    Subscription2((Subscription 2)) --> ResourceGroup2(Frontend Resources);
    Subscription3((Subscription 3)) --> ResourceGroup3(Backend Resources);

    ManagementGroupA((Management Group A)) --> [Hierarchy] --> Subscription1;
    ManagementGroupB((Management Group B)) --> [Hierarchy] --> Subscription2;
    ManagementGroupC((Management Group C)) --> [Hierarchy] --> Subscription3;
```

**Example input / output:**

```

Example Input (CLI):

BASH
Copy
az group create --name "DevOpsResources" --location eastus
az group create --name "FrontendResources" --location westcentralus
az group create --name "BackendResources" --location centralus

Example Output:

CODE
Copy
{
  "frontend-resources": {
    "location": "westcentralus",
    "provisioned": true,
    "resources": [
      "StorageAccount"
    ]
  },
  "backend-resources": {
    "location": "centralus",
    "provisioned": true,
    "resources": [
      "VirtualMachine",
      "LoadBalancer"
    ]
  }
}
```

**Data:**

```yaml
subscriptions:
  - name: DevOps
    resourceGroups:
      - frontend-resources
      - backend-resources

managementGroups:
  - name: MainCorp
    subscriptions:
      - subscription1
      - subscription2
      - subscription3
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph LR;
3. A[Subscription] --> B(Resource Group);
4. C[Management Group] --> D(Subscription);
5. subgraph AzureHierarchy
6. Subscription1((DevOps))
7. Subscription2((Backend))
8. Subscription3((Frontend))
9. ManagementGroupA((MainCorp)) --> [Hierarchy] --> Subscription1;
10. ManagementGroupB((SubcorpA)) --> [Hierarchy] --> Subscription2;
11. ManagementGroupC((SubcorpB)) --> [Hierarchy] --> Subscription3;
12. end

