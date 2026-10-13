---
type: "practice"
difficulty: "Deep"
---

# What's your strategy for governing derived-signal ownership disputes, where team A emits raw metrics that team B transforms into an SLI both claim to own?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Governing Platform Telemetry](<../../../../05_Data_Governance_and_Security/Governance/Data_Strategy/Governing_Platform_Telemetry.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with derived-signal ownership disputes in a complex AIOps environment, our strategy involves a combination of clear communication, well-defined policies, and robust technical enforcement mechanisms. The core idea is to establish a shared understanding of ownership and responsibilities across teams, while ensuring that the data remains transparent and traceable.

The question probes into how we handle conflicts over intellectual property (IP) in data transformation processes. We need to define what "ownership" means in this context—whether it’s about who created the SLI or who is responsible for its maintenance and interpretation. Additionally, the interviewer may be interested in understanding our approach to traceability, accountability, and the technical infrastructure that supports these policies.

#### Triage

Ownership disputes often arise when multiple teams claim responsibility for a derived signal, such as an SLI (Service Level Indicator). To resolve this, we need clear guidelines on who is responsible for what at every stage of data processing. This involves defining roles, responsibilities, and communication channels to ensure that everyone understands their part in the process.

#### Assumptions
Team A emits raw metrics.
Team B transforms these raw metrics into an SLI.
Both teams claim ownership over the derived signal.
The organization has a Data Governance Framework (DGF) in place, which outlines policies and procedures for data management.

#### First-Principles Solution

Define Clear Ownership: Establish clear rules on who owns the raw metrics, the transformed SLI, and the overall transformation process. This might involve defining different levels of ownership such as "creator," "transformer," and "consumer."

Documentation and Traceability: Implement a system where every step in the data pipeline is documented and traceable. This can be achieved using metadata tags or a centralized logging system that captures who created, transformed, and consumed the data.

Communication Channels: Set up regular communication channels (e.g., meetings, Slack channels) to ensure all teams are aware of any changes or disputes regarding ownership. This transparency helps in resolving conflicts early on.

Technical Enforcement Mechanisms: Use technical tools like event logging, automated checks, and audit trails to enforce the defined policies. For example, if a team A metric is used to create an SLI by Team B, this relationship should be documented and enforced through a monitoring system.

Policies and Procedures: Develop comprehensive data governance policies that outline the responsibilities of each team at every stage of the process. These policies should cover not only ownership but also data quality, security, and compliance.

#### Closing Summary

In summary, our strategy for governing derived-signal ownership disputes involves a combination of clear documentation, strong communication channels, technical enforcement mechanisms, and well-defined policies. By establishing these elements, we can ensure that conflicts are minimized and resolved efficiently.

#### Common Mistakes/Follow-Ups
Lack of Clear Definitions: Teams might misunderstand what "ownership" means in different contexts.
Insufficient Documentation: Without proper documentation, disputes can escalate without clear resolution paths.
Failed Communication: Poor communication channels can lead to misunderstandings and delays in conflict resolution.

In follow-up questions, the interviewer may delve deeper into specific technical solutions or explore scenarios where these strategies fail. Be prepared to discuss trade-offs between ease of implementation and enforcement strength.

**Code:**

```
MERMAID
Copy
graph TD;
    A[Team A Emits Raw Metrics] --> B[Raw Data];
    B --> C[Team B Transforms SLI];
    C --> D[SLI];
    E[Policies & Procedures] --> F[Ownership Clarification];
    F --> G[Communication Channels];
    G --> H[Tech Enforcement Mechanisms];
```

**Data:**

```json
{
  "ownership_policy": {
    "raw_metrics_owner": "Team A",
    "derived_signal_transformer": "Team B",
    "derived_signal_consumer": ["Team C", "Team D"]
  },
  "communication_channels": [
    {"type": "Slack Channel", "name": "#data-governance"},
    {"type": "Meeting", "frequency": "Weekly"}
  ],
  "technical_enforcement": {
    "logging_system": "EventLogger",
    "audit_trail": true
  }
}
```

