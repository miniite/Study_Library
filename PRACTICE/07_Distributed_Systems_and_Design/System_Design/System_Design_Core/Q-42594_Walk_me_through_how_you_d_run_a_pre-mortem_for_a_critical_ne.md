---
type: "practice"
difficulty: "Deep"
---

# Walk me through how you'd run a pre-mortem for a critical new system before launch. What failure modes do principals surface that pass every code review, and how do you turn the pre-mortem into architectural changes?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Frontend Graceful Degradation](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/Frontend_Graceful_Degradation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

Running a pre-mortem involves imagining a post-launch disaster to identify potential risks early on. As a Backend Architect, I would lead a structured pre-mortem session where we simulate system failures and brainstorm ways they could happen. This process surfaces architectural flaws that might pass code reviews but are critical for long-term stability. We then use this insights to make necessary adjustments before going live.

#### Triaging the Question

The question is asking how to conduct a pre-mortem, which involves preparing for potential failures in a system even before it launches. The interviewer wants to understand my approach and the failure modes I consider important enough to address beyond just code reviews.

#### Assumptions
and Approach:
We are dealing with a critical new system that needs robustness and reliability.
The pre-mortem should be structured, involving key stakeholders from development, operations, security, and business teams.
Failure modes can range from technical issues (like outages due to resource constraints) to non-technical risks (e.g., legal or regulatory compliance).
Step-by-Step Solution:

Define Objectives:

Identify potential failure points in the system architecture.
Ensure all key stakeholders are aware of the pre-mortem process and its importance.

Identify Key Stakeholders:

Development team to ensure technical issues are considered.
Operations team for understanding operational risks.
Security team for compliance and data integrity concerns.
Business stakeholders to align with business goals and customer expectations.

Structure the Pre-Mortem Session:

Introduction: Explain the purpose of a pre-mortem, emphasizing that it’s not about blaming but identifying risks early.
Brainstorm Potential Failures: As a group, imagine what could go wrong after launch. For example:
System crashes due to insufficient capacity during peak loads.
Data breaches compromising customer information.
Regulatory compliance failures leading to legal penalties.

Analyzing Failure Modes:

Technical Failures: Consider scalability issues, resource constraints, and potential bugs that could lead to downtime or performance degradation.
Operational Failures: Think about human error in deployment processes, configuration management, or monitoring systems.
Security Failures: Evaluate how vulnerabilities might be exploited post-launch. For instance, lack of proper encryption, misconfigured access controls, or weak authentication mechanisms.

Risk Mitigation Strategies:

Develop mitigation plans for each identified failure mode. For example:
Implement load balancing and auto-scaling to handle peak loads.
Conduct regular security audits and penetration testing.
Establish robust logging and monitoring systems to detect anomalies early.

Adjust Architecture Based on Insights:

Use the findings from the pre-mortem to refine the system architecture. For instance, add more redundancy in critical components or improve backup mechanisms.
Ensure that all architectural changes align with long-term goals and maintain high availability.

Document and Communicate Findings:

Create a detailed report summarizing key failure modes, mitigation strategies, and proposed architectural changes.
Share this document with the entire team to ensure everyone is on the same page about potential risks and how they will be addressed.

#### Closing Summary

The pre-mortem process is crucial for identifying and mitigating potential failures early in the system lifecycle. By involving key stakeholders, we can surface technical and non-technical risks that might otherwise pass unnoticed during code reviews. Using this information to make informed architectural changes ensures that our system is robust and resilient once it goes live.

#### Common Mistakes
and Follow-Ups:

#### Common mistakes
include underestimating the complexity of failure modes or failing to involve all necessary stakeholders in the pre-mortem process. Follow-ups might explore specific examples from past projects where a similar approach was taken, discussing both successes and lessons learned.

**Comparison:**

| Failure Mode   | Mitigation Strategy                    |
|---------------|----------------------------------------|
| Insufficient Capacity  | Auto-scaling and Load Balancing             |
| Human Error During Deployment | Automated Deployment Tools       |
| Data Breaches      | Strong Encryption, Regular Audits          |

**Code:**

```
MERMAID
Copy
graph LR;
    A[Define Objectives] --> B[Identify Key Stakeholders];
    B --> C[Introduce Pre-Mortem Session];
    C --> D[Brainstorm Potential Failures];
    D --> E[Analyze Failure Modes];
    E --> F[Develop Mitigation Strategies];
    F --> G[Adjust Architecture Based on Insights];
    G --> H[Document and Communicate Findings];
```

**Example input / output:**

| Input (Pre-Mortem Session) | Output (Mitigation Strategies) |
|----------------------------|------------------------------|
| Identify Failure Modes      | Implement Load Balancing       |
|                             | Conduct Regular Audits        |
|                             | Improve Monitoring Systems    |

**Data:**

```json
{
  "failureModes": [
    {
      "mode": "Technical",
      "scenarios": ["Insufficient Capacity", "Bugs in Critical Code"]
    },
    {
      "mode": "Operational",
      "scenarios": ["Human Error During Deployment", "Misconfigured Monitoring Systems"]
    },
    {
      "mode": "Security",
      "scenarios": ["Data Breaches", "Regulatory Non-Compliance"]
    }
  ]
}
```

**Flow:**

1. Start
2. Introduce Objectives
3. Identify Key Stakeholders
4. Introduce Pre-Mortem Session
5. Brainstorm Potential Failures
6. Analyze Failure Modes
7. Develop Mitigation Strategies
8. Adjust Architecture Based onInsights
9. Document and CommunicateFindings
10. End

