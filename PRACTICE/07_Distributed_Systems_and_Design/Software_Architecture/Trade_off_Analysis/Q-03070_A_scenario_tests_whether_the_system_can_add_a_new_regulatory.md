---
type: "practice"
difficulty: "Deep"
---

# A scenario tests whether the system can add a new regulatory market (say EU data residency) without re-architecting. Walk through analyzing the multi-tenancy and data-partitioning sensitivity points this modifiability scenario exposes.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Quality Attribute Trade-off Scenarios](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Quality_Attribute_Trade-off_Scenarios.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When adding a new regulatory requirement such as EU data residency to an existing system, one of the key challenges is ensuring that the architecture can accommodate these new constraints without necessitating a complete re-architecture. In this scenario, we need to analyze the sensitivity points related to multi-tenancy and data partitioning. Let's break down the analysis step-by-step.

Firstly, multi-tenancy refers to the capability of a single system or application to serve multiple clients while maintaining data isolation. The question is whether our current architecture supports this without requiring significant changes for EU-specific requirements. On the other hand, data partitioning involves dividing the data into smaller chunks based on specific criteria (e.g., geographic region) and storing them in different databases or servers.

#### Triage
What Is Being Probed

The interviewer is probing our understanding of how to manage regulatory compliance without disrupting existing systems. Specifically, they are interested in:

The current architecture's modifiability.
How multi-tenancy impacts the system's ability to handle new regulations.
Strategies for data partitioning that can help maintain compliance.

#### Assumptions
and Key Concepts
We assume a microservices-based architecture where services can be independently scaled and modified.
The system currently supports multiple tenants with some form of tenant isolation, but we need to check if this setup can handle the EU-specific requirements.
Data is partitioned by region or other criteria, which will impact how data residency regulations are handled.

#### First Principles Solution

Current Multi-Tenancy Analysis:

Tenant Isolation: Ensure that each tenant's data is isolated from others to prevent unauthorized access. This can be achieved through database schemas, namespaces, and security policies.
Service Independence: Check if the existing services are designed to operate independently of one another. If a service handles cross-tenant interactions, it may need reworking.
Configuration Management: Use environment-specific configurations (e.g., .env files) to manage different sets of rules or data access permissions for EU tenants.

Data Partitioning Sensitivity Points:

Geographic Data Storage: If the system already stores data geographically, consider if this can be extended to meet EU regulations. For instance, ensuring that all EU-related data is stored within EU data centers.
Replication and Synchronization: Evaluate how data is replicated and synchronized across regions. Ensure there are no cross-border data flows for sensitive data.
Access Control: Implement access controls at the database level to enforce regional compliance rules. This might involve additional layers of authentication or authorization.

Modifiability Strategy:

Incremental Rollouts: Plan an incremental rollout strategy where new services or modified services are introduced gradually, ensuring they meet regulatory requirements.
Service Refactoring: If necessary, refactor existing microservices to better support multi-tenancy and data partitioning for EU compliance. This might involve splitting large monolithic services into smaller, more focused ones.

#### Closing Summary

In summary, adding a new regulatory requirement like EU data residency requires a careful analysis of the current architecture's multi-tenancy and data-partitioning capabilities. By ensuring proper tenant isolation, leveraging environment-specific configurations, implementing robust access controls, and planning for incremental rollouts, we can modify the system without significant rearchitecting.

#### Common Mistakes
and Follow-Ups

#### Common mistakes
include:

Ignoring existing multi-tenant designs that might already support some form of geographic data partitioning.
Overlooking the need to update service dependencies that interact across tenants.
Failing to plan for incremental rollouts, leading to potential downtime or performance issues.

#### Follow-up
questions might involve detailed discussions on specific technologies (e.g., Kubernetes for multi-tenancy), data encryption methods, and compliance frameworks like GDPR.

**Comparison:**

```
Tenant ID	Name	Data Storage Region
1	Tenant A	us-central1
2	EU Tenant	europe-west1
Service ID	Service Name	Tenant ID(s)
1	Service X	1, 2
3	New EU-Specific Ser	2
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Current Multi-Tenancy Analysis] --> B1[Check Tenant Isolation];
    A --> B2[Check Service Independence];
    A --> B3[Environment-Specific Configurations];
    C[Data Partitioning Sensitivity Points] --> D1[Geographic Data Storage];
    C --> D2[Replication and Synchronization];
    C --> D3[Access Control];
    E[Modifiability Strategy] --> F1[Incremental Rollouts];
    E --> F2[Service Refactoring];
```

**Example input / output:**

| Input Action | Output Effect |
|--------------|---------------|
| Add New Service (Service3) for EU Tenants | Modify Service Definitions and Configuration Files to Include New EU-Specific Rules. Incremental Rollouts Ensure Smooth Transition Without Downtime. |

**Data:**

```yaml
# Sample Data Schema for Multi-Tenancy
tenant:
  id: integer
  name: string

service:
  id: integer
  tenant_id: integer
  name: string
  description: string

# Example Configuration File (for EU Tenant)
EU_ENV_FILE:
  services:
    - service1
    - service2
  data_storage_regions:
    - europe-west1
    - europe-central1
```

**Flow:**

1. (Start)
2. (Check Multi-TenancyCapabilities)
3. (Ensure Tenant Data Isolation)
4. (Refactor Services if Necessary)
5. (Plan for Incremental Rollouts)
6. (Check Data PartitioningNeeds)
7. (Update Replication andSynchronization Rules)
8. (Implement Access Controls atDatabase Level)
9. (Use Environment-SpecificConfigurations for EU Tenants)

