---
type: "practice"
difficulty: "Deep"
---

# How do you handle a stakeholder who games the scenario-prioritization vote to protect their subsystem from scrutiny? What facilitation or structural countermeasure detects and neutralizes strategic voting in an ATAM?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Facilitating an ATAM](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Facilitating_an_ATAM.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Handling strategic voting in Scenario-Prioritization Votes during Architectural Trade-off Analysis (ATAM) is crucial to ensure that all subsystems are fairly evaluated. A stakeholder might strategically vote their scenario as a top priority to protect it from scrutiny, which can skew the analysis and lead to suboptimal architectural decisions. To address this, we need a robust facilitation technique or structural countermeasure that detects and neutralizes strategic voting.

Firstly, let's understand what strategic voting means in this context: A stakeholder may artificially inflate their scenario’s importance by gaming the voting process to ensure it gets prioritized, even if it isn’t truly critical. This can be done through multiple votes from stakeholders with vested interests or by repeatedly highlighting minor benefits while downplaying significant issues.

To tackle this problem, we will employ a combination of facilitation techniques and structural countermeasures. Here’s a step-by-step approach:

Facilitation Techniques
Transparent Voting Process: Ensure that the voting process is transparent and auditable. Use digital tools to record each vote with timestamps.
Conflict of Interest Disclosure: Require all stakeholders to declare any potential conflicts of interest before the voting starts.
Collaborative Environment: Foster a collaborative environment where stakeholders feel comfortable sharing their genuine concerns without fear of retribution.
Structural Countermeasures
Independent Scoring and Ranking: Implement an independent scoring mechanism that is not influenced by stakeholder opinions. This could involve using expert panels or automated tools.
Risk-Based Prioritization: Use risk-based criteria to prioritize scenarios independently of the voting results. High-risk areas should be given priority regardless of votes.
Scenario Validation: Validate each scenario based on objective metrics such as impact, complexity, and alignment with business goals.
Detailed Solution

Define Clear Criteria:

Establish clear, objective criteria for evaluating scenarios. For example, consider factors like technical debt, security risks, regulatory compliance, performance impacts, and user experience.

Independent Scoring Tool: Develop a scoring tool that can be used by an independent panel to rate each scenario based on predefined criteria. This tool should be accessible only to the appointed panel members to prevent tampering.

Risk Matrix:

Create a risk matrix where each scenario is plotted based on its identified risks and impacts. This provides a visual representation that helps in prioritizing scenarios irrespective of votes.

Scenario Review Sessions: Conduct review sessions with cross-functional teams, including developers, security experts, product managers, and stakeholders. These sessions should focus on discussing the merits and drawbacks of each scenario without reference to the voting results.

Post-Vote Analysis:

Perform a post-vote analysis where independent metrics are compared against stakeholder votes. This helps in identifying discrepancies that might indicate strategic voting.

#### Example
Workflow
Initial Voting: All stakeholders submit their top 3 scenarios based on subjective priorities.
Independent Scoring: The scoring tool rates each scenario based on predefined criteria.
Risk Matrix Plotting: Each scenario is plotted in a risk matrix for cross-functional review.
Review Sessions: Conducted with independent experts to validate the scoring and discuss any potential strategic voting.
Final Prioritization: A consensus prioritization list is created, taking into account both votes and independent scores.

#### Closing Summary

By employing these facilitation techniques and structural countermeasures, we can mitigate the risk of strategic voting in ATAM scenarios. The key is to maintain transparency, independence, and a focus on objective criteria. This ensures that all subsystems are fairly evaluated, leading to more informed architectural decisions.

#### Common Mistakes/Follow-ups

Ignoring Objective Criteria: Failing to define clear, objective metrics can lead to biased prioritizations.
Lack of Transparency: Without transparent processes, stakeholders might not trust the outcomes.
Over-reliance on Votes: Solely relying on stakeholder votes without independent validation can result in suboptimal decisions.

**Comparison:**

```
2023-10-01
2023-10-02
2023-10-03
2023-10-04
2023-10-05
2023-10-06
2023-10-07
2023-10-08
2023-10-09
2023-10-10
2023-10-11
T1
T2
T3
Technical Debt
Risk Matrix Plotting for ATAM Scenarios
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Initial Voting] --> B[Independent Scoring];
    B --> C[Risk Matrix Plotting];
    C --> D[Review Sessions];
    D --> E[Final Prioritization];
    A --> F[Stakeholder Conflicts of Interest Disclosure];
```

**Example input / output:**

```text
| Scenario | Technical Debt (30%) | Security Risks (30%) | Regulatory Compliance (20%) | Performance Impact (15%) | User Experience (15%) |
|----------|----------------------|----------------------|----------------------------|-------------------------|----------------------|
| S1       | 8                    | 7                    | 6                          | 5                       | 4                    |
| S2       | 9                    | 8                    | 7                          | 6                       | 5                    |
| S3       | 7                    | 6                    | 8                          | 4                       | 6                    |

Independent Scoring:
- S1: 7.5
- S2: 8.0
- S3: 7.0

Risk Matrix Plots (Simplified):
- High-Risk Areas: S1 and S2
- Medium-Risk Area: S3
```

**Data:**

```json
{
  "votingCriteria": [
    { "id": 1, "name": "Technical Debt", "weight": 0.3 },
    { "id": 2, "name": "Security Risks", "weight": 0.3 },
    { "id": 3, "name": "Regulatory Compliance", "weight": 0.2 },
    { "id": 4, "name": "Performance Impact", "weight": 0.15 },
    { "id": 5, "name": "User Experience", "weight": 0.15 }
  ]
}
```

**Flow:**

1. Initial Voting
2. Independent Scoring Tool
3. Risk Matrix Plotting
4. Review Sessions
5. Final Prioritization
6. Stakeholder Conflicts ofInterest Disclosure

