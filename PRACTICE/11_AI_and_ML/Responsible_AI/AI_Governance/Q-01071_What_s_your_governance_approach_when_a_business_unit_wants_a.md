---
type: "practice"
difficulty: "Deep"
---

# What's your governance approach when a business unit wants an AIOps automation to auto-scale down a shared service to cut its cost, but that action externalizes risk onto other tenants of the service?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Governance Dashboards and Visibility](<../../../../11_AI_and_ML/Responsible_AI/AI_Governance/Governance_Dashboards_and_Visibility.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

When faced with the scenario where a business unit wants an AIOps automation to auto-scale down a shared service to reduce costs, but this action externalizes risk onto other tenants, we need to implement a comprehensive governance approach that balances cost optimization and risk management. This includes defining clear policies, conducting thorough impact assessments, implementing robust monitoring and logging, and establishing strong communication channels.

#### Triage

The core of the question is about how to handle a situation where an automated scaling action (in this case, reducing resource allocation) could negatively affect other tenants using the shared service, thereby externalizing risk. The interviewer is probing your understanding of responsible AI governance principles, specifically around fairness, transparency, and accountability.

#### Assumptions
Shared Service: A multi-tenant environment where multiple business units share a common service.
Automated Scaling: An AIOps tool that automatically adjusts resource allocation based on predefined criteria.
Cost Optimization vs. Risk Management: Balancing the need to reduce costs with minimizing potential negative impacts.

#### First Principles Solution

Policy Definition:

Establish clear policies defining when and how scaling actions can be taken. For example, ensure that such actions do not occur during critical service windows or peak usage times.
Define thresholds and rules for when automatic scaling is triggered to prevent over-scaling or under-scaling.

Impact Assessment:

Conduct a thorough risk assessment before implementing any automated scaling policies. This includes understanding the potential impact on other tenants' services, performance, and availability.
Use historical data to predict how different scaling actions might affect system behavior.

Monitoring and Logging:

Implement robust monitoring systems that track resource usage patterns, service health metrics, and tenant-specific performance indicators.
Ensure logging mechanisms are in place to capture any changes or anomalies resulting from the automated scaling actions.

Communication Channels:

Establish clear communication channels between the AIOps team, the business unit requesting the action, and other tenants who might be affected.
Regularly update stakeholders on the progress of these scaling policies and their impacts.

Feedback Loop:

Create a feedback mechanism where users can report issues or concerns about the automated scaling actions. Use this data to refine and improve the policies over time.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, implementing an effective governance approach requires a holistic view that encompasses policy definition, risk assessment, monitoring, communication, and continuous improvement. Failing to address these aspects can lead to unintended consequences such as degraded service quality for other tenants or operational disruptions.

Some common mistakes include:

Overlooking the potential impact on non-requesting tenants.
Insufficient policy enforcement leading to inconsistent application of scaling rules.
Lack of transparency in decision-making processes, which reduces trust among stakeholders.

#### Follow-up
questions might explore specific tools and techniques used for risk assessment, detailed steps for establishing monitoring systems, or examples of feedback mechanisms in use.

**Comparison:**

```
Metric	Threshold Value	Action
CPU Usage	>75%	Scale Down
Memory Usage	>60%	Scale Down
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Policy Definition] --> B(Risk Assessment);
    B --> C(Monitoring & Logging);
    C --> D(Communication Channels);
    D --> E[Feedback Loop];
```

**Example input / output:**

```

Sample input for a risk assessment:

JSON
Copy
{
  "service": "SharedDatabase",
  "tenants": [
    { "name": "TenantA", "resourceUsage": 80, "criticalityLevel": "High" },
    { "name": "TenantB", "resourceUsage": 30, "criticalityLevel": "Low" }
  ],
  "scalingAction": {
    "type": "DownScale",
    "thresholds": [
      { "metric": "CPU Usage", "value": 75, "action": "Scale Down" },
      { "metric": "Memory Usage", "value": 60, "action": "Scale Down" }
    ]
  }
}

Sample output for risk assessment:

JSON
Copy
{
  "service": "SharedDatabase",
  "riskAssessment": {
    "impactOnTenants": [
      { "tenantName": "TenantA", "riskLevel": "Medium", "reason": "High criticality, near threshold" },
      { "tenantName": "TenantB", "riskLevel": "Low", "reason": "Below threshold" }
    ],
    "recommendations": [
      "Consider manual review before applying scaling action.",
      "Adjust thresholds to minimize risk impact."
    ]
  }
}
```

**Data:**

```yaml
policies:
  - name: Critical Service Windows
    description: "Avoid scaling actions during peak usage times."
  - name: Resource Utilization Thresholds
    description: "Trigger automatic scaling when resource utilization exceeds certain thresholds."
```

**Flow:**

1. Start
2. Policy Definition
3. Risk Assessment
4. Monitoring & Logging
5. Communication Channels
6. Feedback Loop
7. End

