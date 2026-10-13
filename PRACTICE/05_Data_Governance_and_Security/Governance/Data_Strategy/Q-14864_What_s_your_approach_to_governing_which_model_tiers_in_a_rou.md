---
type: "practice"
difficulty: "Expert"
---

# What's your approach to governing which model tiers in a routing fabric are approved for which data-sensitivity classes, and how do you enforce it at request time?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Governing Incident Data Reuse](<../../../../05_Data_Governance_and_Security/Governance/Data_Strategy/Governing_Incident_Data_Reuse.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To govern which model tiers in a routing fabric are approved for which data-sensitivity classes, I would approach this by defining clear policies and implementing robust enforcement mechanisms. The key is to ensure that sensitive data flows only through appropriately secured and vetted models, thereby maintaining compliance and security.

First, the question probes how we can map data sensitivity levels to model tiers effectively. This requires a deep understanding of both the data sensitivity classification framework (e.g., low, medium, high) and the characteristics of different model tiers (e.g., accuracy, training frequency, security measures). We also need to consider the routing fabric's architecture and the potential impact on performance.

#### Triaging the Question
What is being probed: The interviewer wants to understand how you would design a system that enforces data governance based on sensitive data classifications.

#### Assumptions
Assume we have a multi-tiered model structure with varying levels of security and accuracy. We need to map these tiers to different data sensitivity levels.
Key Concepts: Data classification, model tiering, routing policies, enforcement mechanisms.

#### First-Principles Solution

Define Data Sensitivity Classes:

Low: Public or semi-sensitive data.
Medium: Sensitive but not critical (e.g., personal identification information).
High: Highly sensitive and critical (e.g., financial transactions).

Model Tiering:

Tier 1: Basic, publicly accessible models.
Tier 2: Intermediate models with enhanced security measures.
Tier 3: Advanced, highly secure models.

Policy Mapping:

Assign each data sensitivity class to the appropriate model tier based on risk and compliance requirements.
For example, high-sensitivity data should be routed only through Tier 3 models.

Routing Fabric Design:

Implement a hierarchical routing system that dynamically routes requests based on their data sensitivity level.
Use a combination of metadata tagging and policy enforcement to ensure correct routing.

Enforcement Mechanisms:

Utilize API gateways or intermediary services to intercept requests before they reach the final model tier.
Apply access controls, encryption, and logging mechanisms at each stage to enforce security policies.

Continuous Monitoring and Auditing:

Monitor the routing fabric for compliance violations and implement automated alerts.
Regularly audit the system to ensure that data is not bypassing appropriate security measures.

#### Closing Summary

The approach involves defining clear mappings between data sensitivity classes and model tiers, designing a robust routing fabric that enforces these policies, and implementing continuous monitoring and auditing mechanisms. This ensures that sensitive data always flows through the most secure pathways, maintaining compliance and mitigating risks.

#### Common Mistakes/Follow-Ups
Mistake 1: Failing to define clear mappings between sensitivity classes and model tiers. Ensure a systematic approach is taken.

#### Follow-Up
Discuss how you would handle edge cases where data might need different levels of security for various operations.
Mistake 2: Overlooking the performance implications of enforcing strict routing policies. Consider trade-offs and optimizations.

#### Follow-Up
Talk about potential optimizations such as caching, load balancing, or using more efficient encryption methods.

**Comparison:**

```
Data Sensitivity	Model Tier	Characteristics
Low	1	Basic, publicly accessible models
Medium	2	Intermediate models with enhanced security measures
High	3	Advanced, highly secure models
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[User Request] --> B{Is Data Sensitive?};
    B -- No --> C[Basic Model Tier 1];
    B -- Yes --> D{Data Sensitivity Level};
    D -- Low --> E[Moderate Security Model Tier 2];
    D -- High --> F[Highly Secure Model Tier 3];
```

**Example input / output:**

```text
Request:
- Data Sensitivity Level: High

Response:
- Route Request to Highly Secure Model Tier 3
- Apply Advanced Encryption and Access Controls
- Log Detailed Audit Trail for Compliance Verification
```

**Data:**

```json
{
  "dataSensitivity": {
    "low": "Public or semi-sensitive data",
    "medium": "Sensitive but not critical (e.g., personal identification information)",
    "high": "Highly sensitive and critical (e.g., financial transactions)"
  },
  "modelTiers": {
    "1": "Basic, publicly accessible models",
    "2": "Intermediate models with enhanced security measures",
    "3": "Advanced, highly secure models"
  }
}
```

**Flow:**

1. No
2. Yes
3. Low
4. High
5. User Request
6. Is Data Sensitive?
7. Basic Model Tier 1
8. Data Sensitivity Level
9. Moderate Security Model Tier2
10. Highly Secure Model Tier 3

