---
type: "practice"
difficulty: "Deep"
---

# You must present a concurrency-architecture decision (strong vs eventual consistency for a core domain) to an architecture review board with dissenting principals. How do you structure the trade-off analysis, quantify the business impact of each anomaly, and drive to a decision without authority over the dissenters?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Lock-free Read Paths](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Lock-free_Read_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When presenting a concurrency-architecture decision on strong vs eventual consistency in a core domain, it's crucial to approach this from multiple angles to ensure a thorough understanding of both trade-offs and implications. The goal is to structure the discussion around clear, measurable criteria that resonate with the business stakeholders while addressing the concerns of dissenting principals.

#### Triaging the Question

The question revolves around choosing between strong consistency (ensuring all read operations see the most recent write operation) and eventual consistency (eventually converging on a single state but allowing temporary divergences). The core challenge is to present a compelling argument that balances these choices, quantify their business impact, and facilitate a consensus-driven decision.

#### Assumptions
We are dealing with a critical system where data integrity is paramount.
There may be trade-offs between performance, availability, and data accuracy.
The architecture review board has diverse perspectives on what consistency means for the specific use case.
Dissenting principals have valid concerns that need to be heard and addressed.

#### First Principles Solution

Understand the Core Requirements: Identify the key business processes affected by strong vs eventual consistency. For example, in a banking application, transactions must be strongly consistent to avoid double-spending. In a social media platform, it might be acceptable for posts to eventually converge but not for transactions.

Model the Anomalies: Define the types of anomalies that could occur under each consistency model:

Strong Consistency: Data is guaranteed to be in a single, uniform state at any point in time.
Eventual Consistency: Data can temporarily diverge and may require retries or additional operations to converge.

Quantify Business Impact: Assess the impact of these anomalies on key business metrics such as:

Customer Trust: The perception that transactions are secure and reliable.
Regulatory Compliance: Adherence to industry regulations requiring strong consistency.
User Experience: How temporary divergences affect user satisfaction.
Operational Costs: The cost implications of implementing robust solutions for eventual consistency.

Risk Assessment: Evaluate the risks associated with each choice, considering both short-term and long-term impacts:

Short-Term: Immediate availability vs. potential downtime during data convergence.
Long-Term: Scalability challenges, operational complexity, and maintenance costs.

Stakeholder Alignment: Engage stakeholders to understand their pain points and priorities. Use a structured approach like the MoSCoW method (Must Have, Should Have, Could Have, Won't Have) to prioritize features based on business value.

Decision Criteria: Develop a set of decision criteria that align with the organization's strategic goals:

Business Impact: Which consistency model supports the core business processes best?
Technical Feasibility: Can both models be implemented effectively within existing infrastructure and team expertise?
Cost-Benefit Analysis: What are the long-term costs and benefits of each approach?

#### Example
Quantification

Let’s consider an example where a finance application requires strong consistency for transaction processing but can tolerate eventual consistency in reporting:

Strong Consistency: High initial cost due to distributed transactions, potential downtime during data synchronization. Business impact is high on customer trust and regulatory compliance.
Eventual Consistency: Lower initial cost with asynchronous updates, less complex infrastructure. Risk of temporary divergences affecting user experience but can be mitigated with retry mechanisms.
Structuring the Presentation

Introduction:

Introduce the core problem: ensuring data integrity in a high-stress environment.
Highlight why this decision is critical for the business.

Anomaly Analysis:

Present specific scenarios where anomalies could occur.
Use hypothetical examples like failed transactions or delayed reports to illustrate potential issues.

#### Trade-Offs

Discuss the trade-offs between strong and eventual consistency in terms of cost, performance, and risk.
Provide a comparative analysis using metrics such as MTTR (Mean Time To Repair), SLAs (Service Level Agreements), and TCO (Total Cost of Ownership).

Decision Criteria:

Summarize the decision criteria based on stakeholder input.
Propose a balanced solution that addresses key requirements while minimizing risks.

#### Conclusion

Conclude by reiterating the importance of the decision and its impact on business goals.
Encourage an iterative approach to refine the chosen model over time as more data becomes available.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Emphasize the need for a collaborative approach, acknowledging that no single solution fits all scenarios.
Highlight common mistakes such as underestimating the complexity of eventual consistency or ignoring short-term costs.
Prepare for follow-up questions on specific implementation details and potential fallback strategies.

**Comparison:**

```
Metric	Strong Consistency	Eventual Consistency
MTTR	120 seconds	30 seconds
SLA	99.5%	98.7%
TCO	$500k	$350k
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Define Anomalies] --> B[Quantify Business Impact];
    B --> C[Evaluate Risks];
    C --> D[Stakeholder Alignment];
    D --> E[Decision Criteria];
    E --> F[Present Trade-Offs];
    F --> G[Presentation Outline];
    G --> H[Collaborative Decision-Making];
```

**Example input / output:**

```text
Input: Metrics for Strong Consistency vs Eventual Consistency

Output:
- MTTR (Strong Consistency): 120 seconds vs (Eventual Consistency): 30 seconds
- SLA Compliance (Strong Consistency): 99.5% vs (Eventual Consistency): 98.7%
- TCO (Strong Consistency): $500k vs (Eventual Consistency): $350k

Decision: Based on these metrics, the organization decided to implement strong consistency for critical transactions and eventual consistency for non-critical reporting.
```

**Data:**

```json
{
  "Anomalies": {
    "StrongConsistency": ["Failed Transactions", "Regulatory Non-Compliance"],
    "EventualConsistency": ["Delayed Reports", "User Experience Downtime"]
  },
  "Metrics": {
    "MTTR": [120, 30],
    "SLA Compliance": [99.5, 98.7],
    "TCO": [500k, 350k]
  }
}
```

**Flow:**

1. Define Problem
2. Identify Anomalies
3. Quantify Impact
4. Evaluate Risks
5. Stakeholder Alignment
6. Decision Criteria
7. Propose Solution
8. Presentation Outline
9. Collaborative Decision-Making

