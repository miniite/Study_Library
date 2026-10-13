---
type: "practice"
difficulty: "Expert"
---

# How would you architect a compliance-as-code program on Azure so that PCI-DSS and SOC 2 controls are continuously enforced across every new subscription, and who owns exceptions?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Azure DevOps and Artifact Governance](<../../../../TOOLS/Azure/Azure_Platform/Azure_DevOps_and_Artifact_Governance/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect a compliance-as-code program in Azure to enforce PCI-DSS and SOC 2 controls continuously, we need to ensure that all new subscriptions adhere strictly to these standards without manual intervention. This architecture should be scalable, automated, and capable of managing exceptions transparently.

The core idea is to leverage Azure Security Center for policy enforcement, integrate with Azure DevOps for version-controlled policies, and use Azure Policy and Security Center rules to define and apply compliance controls. We also need a system in place to manage exceptions, ensuring that any deviation from the standards can be tracked and approved by authorized personnel.

First, we will outline the key components and then walk through how they work together.

**Comparison:**

```
Subscription	Policy Name	Status	Violations
myrg	PCI-DSS-Logging	Non-compliant	1
otherrg	SOC2-AuditLogs	Compliant	0
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Subscription Creation] --> B(Azure Policy)
    B --> C[Azure Security Center]
    C --> D[Azure DevOps Repositories]
    E[Exception Management] --> F[Approval Workflow]
```

**Example input / output:**

```

For a new subscription created, the following steps would be taken:

Subscription Creation: A new Azure subscription is created.
Azure Policy Application: Azure Policy enforces predefined policies for PCI-DSS and SOC 2 compliance.
Continuous Monitoring: Azure Security Center continuously monitors the environment against these policies.
Policy Violations: Any violations trigger alerts, which are then processed by the exception management system.

Example:

JSON
Copy
{
  "request": {
    "resourceId": "/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/myRG/providers/Microsoft.Web/sites/mysite",
    "policyDefinitionName": "PCI-DSS-Logging"
  },
  "response": {
    "status": "violation",
    "message": "The logging configuration for this web site does not meet PCI-DSS requirements."
  }
}
```

**Data:**

```yaml
# Azure Policy Definitions for PCI-DSS and SOC 2 Compliance
- policyDefinitionName: "PCI-DSS-Logging"
  policyRule:
    if:
      allOf:
        - field: type
          equals: Microsoft.Web/sites
    then:
      effect: deny

- policyDefinitionName: "SOC2-AuditLogs"
  policyRule:
    if:
      allOf:
        - field: type
          equals: Microsoft.Network/virtualNetworks
    then:
      effect: audit
```

**Flow:**

1. Subscription Creation
2. Azure Policy Enforcement
3. Azure Security CenterMonitoring
4. Policy Violation Alert
5. Exception ManagementSystem
6. Approval Workflow forExceptions

