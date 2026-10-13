---
type: "practice"
difficulty: "Expert"
---

# A scenario reveals that scaling to 100x current load would require re-sharding a stateful service with no online resharding path today. How do you record this — risk, tradeoff, or sensitivity — and how do you frame the remediation roadmap?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Sensitivity Points](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Sensitivity_Points.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

In this scenario, we need to assess and document the impact of re-sharding a stateful service without an online resharding path as part of our architectural trade-off analysis (ATAM). This involves recording the risks involved and framing a remediation roadmap that prioritizes both technical feasibility and business impact.

#### Triaging the Question

The question is essentially asking us to:

Document the risk: Identify and articulate the potential issues and implications associated with re-sharding.
Frame the tradeoff: Understand the cost-benefit analysis of current vs. future scaling capabilities without an online resharding path.
Plan remediation: Develop a roadmap for addressing these issues, considering both short-term and long-term solutions.

#### Assumptions
The stateful service is mission-critical with high availability requirements.
There are existing data dependencies or transactional integrity concerns that complicate the sharding process.
Online resharding is not feasible due to technical constraints or business limitations.

#### First Principles Solution
1. Risk Assessment
Operational Risk: Potential service downtime during offline re-sharding, which could impact customer experience and business operations.
Data Integrity Risk: Ensuring data consistency across the new sharding strategy without disrupting ongoing transactions.
Recovery Risk: The complexity of rolling back changes if something goes wrong during the re-sharding process.
2. Tradeoff Analysis
Current State vs Future Scalability:
Current State: The service is currently stateful, tightly coupled with its database schema, and operates under high load.
Future Scalability: Re-sharding could significantly improve scalability but requires downtime and additional complexity in data management.
3. Remediation Roadmap

Short-Term Remediation:

Implement caching strategies to alleviate some of the current load and prevent immediate crashes due to high demand.
Perform offline re-sharding during maintenance windows, carefully planning to minimize downtime and ensure data integrity.
Develop a detailed rollback plan in case any issues arise during the re-sharding process.

Long-Term Remediation:

Invest in modernizing the service architecture by decoupling state from core logic and enabling online resharding through incremental migrations.
Explore alternative database technologies that support sharding natively or offer more flexible schema management capabilities.

#### Closing Summary

In summary, we need to document these risks and tradeoffs clearly and prioritize them based on their impact. The roadmap should focus on mitigating the immediate risks while laying out a strategy for long-term scalability improvements.

#### Common Mistakes/Follow-Ups
Overlooking Data Consistency: Ensuring that data integrity checks are robust during re-sharding is crucial.
Neglecting Business Impact: Failing to consider how these changes might affect business operations and customer satisfaction.
Ignoring Technical Feasibility: Overestimating the complexity of online resharding or underestimating the effort required for offline sharding.

#### Triaging the Question

The question is essentially asking us to:

Document the risk: Identify and articulate the potential issues and implications associated with re-sharding.
Frame the tradeoff: Understand the cost-benefit analysis of current vs. future scaling capabilities without an online resharding path.
Plan remediation: Develop a roadmap for addressing these issues, considering both short-term and long-term solutions.

#### Assumptions
The stateful service is mission-critical with high availability requirements.
There are existing data dependencies or transactional integrity concerns that complicate the sharding process.
Online resharding is not feasible due to technical constraints or business limitations.

#### First Principles Solution
1. Risk Assessment
Operational Risk: Potential service downtime during offline re-sharding, which could impact customer experience and business operations.
Data Integrity Risk: Ensuring data consistency across the new sharding strategy without disrupting ongoing transactions.
Recovery Risk: The complexity of rolling back changes if something goes wrong during the re-sharding process.
2. Tradeoff Analysis
Current State vs Future Scalability:
Current State: The service is currently stateful, tightly coupled with its database schema, and operates under high load.
Future Scalability: Re-sharding could significantly improve scalability but requires downtime and additional complexity in data management.
3. Remediation Roadmap

Short-Term Remediation:

Implement caching strategies to alleviate some of the current load and prevent immediate crashes due to high demand.
Perform offline re-sharding during maintenance windows, carefully planning to minimize downtime and ensure data integrity.
Develop a detailed rollback plan in case any issues arise during the re-sharding process.

Long-Term Remediation:

Invest in modernizing the service architecture by decoupling state from core logic and enabling online resharding through incremental migrations.
Explore alternative database technologies that support sharding natively or offer more flexible schema management capabilities.

#### Closing Summary

In summary, we need to document these risks and tradeoffs clearly and prioritize them based on their impact. The roadmap should focus on mitigating the immediate risks while laying out a strategy for long-term scalability improvements.

#### Common Mistakes/Follow-Ups
Overlooking Data Consistency: Ensuring that data integrity checks are robust during re-sharding is crucial.
Neglecting Business Impact: Failing to consider how these changes might affect business operations and customer satisfaction.
Ignoring Technical Feasibility: Overestimating the complexity of online resharding or underestimating the effort required for offline sharding.

**Comparison:**

| Risk Type        | Description                       |
|------------------|-----------------------------------|
| **Data Integrity** | Ensuring data consistency during sharding. |
| **Operational**  | Service downtime during maintenance windows. |
| **Recovery**     | Complexity of rollback process. |
| Tradeoff Type    | Current State                         | Future Scalability                      |
|------------------|---------------------------------------|-----------------------------------------|
| **Current State** | Tightly coupled, stateful service.       | Improved scalability but requires downtime and complexity. |

**Code:**

```
MERMAID
Copy
graph LR;
    A[Current State] --> B{Offline Resharding};
    B --> C[Data Integrity Risk];
    B --> D[Operational Risk];
    B --> E[Rollback Plan];
    F[Long-Term Remediation] --> G[Modernized Architecture];
    F --> H[Alternative DB Technologies];
```

**Example input / output:**

| Input           | Output                    |
|-----------------|---------------------------|
| Current State   | High Load, Tightly Coupled |
| Offline Resharding | Data Integrity Risk, Operational Risk, Rollback Plan |
| Long-Term Remediation | Decoupling state, Alternative DB Tech |

**Data:**

```json
{
  "risks": [
    {
      "name": "Data Integrity",
      "description": "Ensuring data consistency across sharding boundaries."
    },
    {
      "name": "Operational",
      "description": "Service downtime during maintenance windows."
    },
    {
      "name": "Recovery",
      "description": "Complexity of rollback process."
    }
  ],
  "tradeoffs": [
    {
      "currentState": "Tightly coupled, stateful service.",
      "futureScalability": "Improved scalability but requires downtime and complexity."
    }
  ],
  "remediationPlan": {
    "shortTerm": [
      "Caching",
      "Offline re-sharding during maintenance windows"
    ],
    "longTerm": [
      "Decoupling state from core logic",
      "Exploring alternative database technologies"
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Is offline re-sharding feasible?
7. Caching strategy
8. Offline re-sharding duringmaintenance windows?
9. Risk assessment and rollbackplan development
10. Invest in modernizedarchitecture, explorealternative DB technologies
11. Documentation of risks andtradeoffs
12. Documentation of long-termremediation roadmap
13. End

