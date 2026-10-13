---
type: "practice"
difficulty: "Deep"
---

# A skeptical staff engineer argues your proposed CRDT-based conflict resolution is over-engineering for the current scale. How do you adjudicate "design for today vs design for the known growth curve" for a concurrency model that's expensive to change later? Make the call and defend it.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Distributed Coordination Choices](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Distributed_Coordination_Choices.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this skeptical staff engineer, we need to balance the trade-offs between current needs and future scalability. The key is to design a system that can scale with our known growth curve while maintaining flexibility to adapt as needed. Let's break down the problem step by step.

#### Triage

The question probes whether CRDT-based conflict resolution (CRDTs) are justified given the current scale of operations. CRDTs offer distributed synchronization and eventual consistency, which can be crucial for handling concurrent updates in a scalable manner. However, they come with complexity overhead. We need to evaluate if this overhead is worth it at our current scale.

#### Assumptions
Current Scale: The application is not yet at a scale where data inconsistencies or frequent conflicts are a critical issue.
Future Growth Curve: There is reasonable confidence that the growth will push us towards needing more robust concurrency handling, possibly in the medium to long term.
Development and Maintenance Costs: CRDTs involve significant upfront development effort and ongoing maintenance.

#### First-Principles Solution
Evaluate Current Needs: Assess the current system's ability to handle concurrent updates without issues. If conflicts are rare or easily resolved through simpler mechanisms, CRDTs may be unnecessary.
Growth Projections: Identify key metrics that indicate when we might need a more robust concurrency model (e.g., transaction volume, number of users).
Risk Analysis: Quantify the risk of not having a robust concurrency model as growth accelerates. Consider the impact of data inconsistencies on user experience and system reliability.
Cost-Benefit Analysis: Evaluate the total cost of implementing CRDTs versus the potential benefits in terms of future scalability.
Step-by-Step Reasoning

Current System Assessment:

Conduct a thorough audit to identify current bottlenecks and areas where conflicts occur.
Use logging and monitoring tools to capture real-world data on concurrency issues.

Growth Projections:

Estimate future growth using historical trends or market forecasts.
Identify critical points in the growth curve where we might need a more robust solution (e.g., doubling user base, significant transaction volume).

Risk Analysis:

Quantify the risk of data inconsistencies leading to critical failures or poor user experience.

#### Cons
ider worst-case scenarios and their impact on business operations.

Cost-Benefit Analysis:

Estimate the development time and maintenance costs for CRDTs versus simpler concurrency models.
Calculate potential savings in reduced downtime, improved user satisfaction, and enhanced system reliability.
Decision

Given our growth projections and risk analysis, we decide to implement a CRDT-based model. This approach ensures that as we scale, we have a robust foundation ready to handle increased concurrency without the need for significant rework.

#### Closing Summary
and Common Mistakes/Follow-Ups
Mistake 1: Ignoring current system issues in favor of future scalability.

#### Follow-Up
1: How will you monitor and adjust CRDTs as the system grows?
Mistake 2: Overlooking potential simpler solutions that might suffice for now.

#### Follow-Up
2: What are alternative concurrency models we could consider if they fit current needs?

By making a well-informed decision based on a comprehensive analysis, we ensure that our design is both future-proof and practical.

**Comparison:**

| Metric           | Current Scale | Growth Projection in 3 Years |
|------------------|---------------|----------------------------|
| Users            | 5000          | 100,000                    |
| Transactions/Sec | 20            | 1,000                      |

**Code:**

```python
# Example of a simple concurrency check function
def handle_concurrency(data):
    # Simple conflict resolution logic (for demonstration)
    if data['version'] > 10:
        return True
    else:
        return False
```

**Example input / output:**

```text
Input:
- Current scale data: {users: 5000, transactions_per_second: 20}
- Growth projections: {expected_users_in_3_years: 100000, expected_transactions_per_second_in_3_years: 1000}

Output:
- Decision to implement CRDT-based conflict resolution
- Justification based on current needs and future growth projections
```

**Data:**

```json
{
  "current_scale": {
    "users": 5000,
    "transactions_per_second": 20
  },
  "growth_projections": {
    "expected_users_in_3_years": 100000,
    "expected_transactions_per_second_in_3_years": 1000
  }
}
```

**Flow:**

1. No
2. Yes
3. Current System Audit
4. Are Conflicts Critical?
5. Maintain Current Model
6. CRDT-based ConflictResolution
7. Growth Projections Analysis
8. Risk and Cost-Benefit Analysis
9. Implement CRDT-based Model

