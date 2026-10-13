---
type: "practice"
difficulty: "Expert"
---

# You must pick a datastore standard for the whole org: one blessed database vs teams choosing per-workload. Argue both sides at the platform level (paved road vs autonomy), and describe the governance model you'd actually ship so you get 80% of the benefit without becoming a bottleneck.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Federated vs Centralized Data Ownership](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Federated_vs_Centralized_Data_Ownership.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding on a datastore standard for an entire organization, we face two primary choices: using one blessed database across all teams or allowing each team to choose their own databases per workload. The decision hinges on balancing platform-level convenience and autonomy with the efficiency of governance and the overall adoption of best practices.

TL;DR

We'll argue both sides of the argument for a single standard versus per-workload databases, then propose a hybrid model that leverages centralized governance while still allowing teams some flexibility. This approach aims to achieve 80% of the benefits of a unified database without becoming a bottleneck or overly restrictive.

#### Triage
The Core Question and What's Being Probed

The question revolves around balancing platform-level convenience with team autonomy in terms of choosing datastores. It probes our understanding of governance models, trade-offs between centralized control and decentralized flexibility, and how to implement effective management strategies at scale.

#### First-Principles Solution
One Blessed Database (Centralized Governance)

#### Pros

Simplified Management: Easier for IT operations to manage a single database system.
Standardization: Ensures all teams adhere to the same best practices and standards, reducing integration issues.
Scalability: Centralized systems can scale more efficiently due to better resource utilization.

#### Cons

Resistance to Change: Teams may resist using a new or unfamiliar database system, leading to slow adoption.
Limited Autonomy: Lack of flexibility for teams with specific needs that aren't met by the chosen database.
Bottleneck Risk: If not managed properly, centralized governance can become a bottleneck.
Per-Workload Databases (Decentralized Governance)

#### Pros

Team Autonomy: Teams can choose solutions best suited to their workload requirements.
Innovation: Faster innovation cycles as teams are free to experiment with different technologies.
Better Fit for Workloads: Tailored databases might better fit specific workloads, improving performance and efficiency.

#### Cons

Complexity in Management: More challenging to manage multiple database systems.
Standardization Issues: Lack of uniform standards can lead to integration challenges and increased maintenance costs.
Scalability Issues: Each team managing their own databases may result in fragmented resource allocation and inefficiencies.
Proposed Hybrid Governance Model

To achieve a balanced solution, we propose a hybrid model that leverages centralized governance while allowing teams some flexibility:

Centralized Standards and Best Practices:

Define and enforce a set of standards for security, performance, and compatibility.
Provide guidelines on when to use specific database technologies based on workload requirements.

Decentralized Decision-Making:

Allow teams to choose their databases within the bounds of predefined standards.
Implement an approval process where new database choices must be vetted by a central governance team before deployment.

Governance Tools and Processes:

Use automation tools for database management and monitoring to reduce overhead.
Regularly review and update best practices to ensure they remain relevant and effective.

Centralized Support and Training:

Offer centralized support and training resources for all teams, ensuring everyone has access to the same knowledge base.
Implement a help desk system where teams can request assistance with database issues.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By adopting this hybrid model, we aim to achieve 80% of the benefits of centralized governance without becoming overly restrictive. However, it's crucial to continuously monitor and adapt the governance model based on feedback from teams and changing organizational needs.

#### Common Mistakes

Overzealous Centralization: Failing to give teams enough autonomy can stifle innovation.
Lack of Clear Standards: Without clear guidelines, teams may make suboptimal choices that create integration issues.
Inadequate Monitoring: Failing to implement proper monitoring and support mechanisms can lead to scalability issues.

#### Follow-Ups

How would you handle a situation where a team wants to use a new database technology not currently in the approved list?
What role do you see DevOps teams playing in this governance model?

**Comparison:**

```
Governance Model	Centralized Standards	Decentralized Decision-Making
Pros	Simplified management, standardization, scalability	Team autonomy, innovation, better fit for workloads
Cons	Resistance to change, limited autonomy, bottleneck risk	Complexity in management, standardization issues, scalability issues
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Define Standards] --> B(Implement Approval Process);
    C[Decentralized Decision-Making] --> D[Use Automation Tools];
    E[Centralized Support and Training] --> F(Monitor and Adapt Governance Model);
```

**Example input / output:**

```text
Input:
- Team requests to use Apache Cassandra for their new big data project.
- Existing approved list includes MySQL, PostgreSQL, and MongoDB.

Output:
- Team's request is reviewed by the governance team.
- If deemed fit, the governance team approves the usage of Apache Cassandra for this specific workload.
- Monitoring tools are updated to include Apache Cassandra metrics.
```

**Data:**

```json
{
  "standards": {
    "securityLevel": "high",
    "performanceRequirements": ["responseTime", "throughput"]
  },
  "approvalProcess": [
    { "team": "Sales", "database": "MySQL" },
    { "team": "Marketing", "database": "PostgreSQL" }
  ],
  "monitoringTools": ["Prometheus", "Grafana"],
  "trainingResources": {
    "DBMS": ["Oracle", "MongoDB", "Cassandra"],
    "SecurityBestPractices": true
  },
  "governanceModel": "hybrid"
}
```

**Flow:**

1. Approval
2. Denial
3. New Database Request
4. Review by Governance Team
5. Approval or Denial
6. Update Monitoring Tools
7. Revert to Original Choice

