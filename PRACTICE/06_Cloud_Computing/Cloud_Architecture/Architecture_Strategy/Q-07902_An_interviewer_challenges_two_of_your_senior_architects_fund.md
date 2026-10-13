---
type: "practice"
difficulty: "Deep"
---

# An interviewer challenges: two of your senior architects fundamentally disagree on the target architecture. As principal, how do you resolve it and keep both engaged?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Balancing Architecture Pillars](<../../../../06_Cloud_Computing/Cloud_Architecture/Architecture_Strategy/Balancing_Architecture_Pillars.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When faced with a disagreement between senior architects over cloud architecture strategy, my approach involves a structured yet collaborative process to ensure both perspectives are heard, evaluated, and reconciled in a way that aligns with our overall business objectives. Here’s how I would handle it step-by-step:

Initial Assessment: Understand the core of the disagreement by actively listening to both architects. This involves clarifying their specific concerns, goals, and proposed architectures.
Objective Alignment: Ensure that both architects understand the broader organizational context, including key business goals, technical constraints, compliance requirements, and cost considerations.
Common Ground Identification: Identify shared values and objectives to establish a common starting point for discussion.
Structured Analysis: Evaluate each proposal against predefined criteria such as scalability, security, availability, performance, and cost-effectiveness.
Risk Assessment: Conduct a thorough risk analysis on both options to determine which approach poses fewer risks or can be mitigated more effectively.
Stakeholder Involvement: Engage key stakeholders for input, ensuring that the solution aligns with organizational priorities.
Decision-Making Process: Use collective wisdom from cross-functional teams and consider external consultants if necessary.
Resolution Strategy: Develop a resolution strategy that incorporates valid points from both proposals to create an integrated architecture.

By following these steps, we can ensure that both architects remain engaged and committed to the final decision, fostering a culture of collaboration and mutual respect within the team.

**Comparison:**

```
Criteria	Weight	Proposal 1 Score	Proposal 2 Score
Scalability	0.3	8	7
Security	0.25	7	8
Availability	0.15	6	5
Performance	0.1	9	8
Cost-effectiveness	0.2	5	6
```

**Code:**

```text
# Example Code for Risk Assessment
def assess_risk(proposal):
    # Define criteria weights
    weight_scalability = 0.3
    weight_security = 0.25
    weight_availability = 0.15
    weight_performance = 0.1
    weight_cost_effectiveness = 0.2

    # Calculate risk score
    risk_score = (proposal['scalability'] * weight_scalability +
                  proposal['security'] * weight_security +
                  proposal['availability'] * weight_availability +
                  proposal['performance'] * weight_performance +
                  proposal['cost-effectiveness'] * weight_cost_effectiveness)

    return risk_score
```

**Example input / output:**

```text
Input:
Assess risk of proposal1 and proposal2.

Output for Proposal 1:
Risk Score: 7.0

Output for Proposal 2:
Risk Score: 6.5
```

**Data:**

```json
{
  "proposal1": {
    "scalability": 8,
    "security": 7,
    "availability": 6,
    "performance": 9,
    "cost-effectiveness": 5
  },
  "proposal2": {
    "scalability": 7,
    "security": 8,
    "availability": 5,
    "performance": 8,
    "cost-effectiveness": 6
  }
}
```

**Flow:**

1. Identify Core Disagreement
2. Understand Both Perspectives
3. Objective Alignment
4. Common GroundIdentification
5. Structured Analysis
6. Risk Assessment
7. Stakeholder Involvement
8. Decision-Making Process
9. Resolution StrategyDevelopment
10. Engage Both Architects
11. Consolidate and ImplementArchitecture

