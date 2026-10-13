---
type: "practice"
difficulty: "Deep"
---

# How do you weight importance versus architectural-difficulty when prioritizing platform scenarios? Give an example where a low-importance but high-difficulty scenario (say cross-cloud disaster failover) was still worth analyzing because it exposed a foundational fleet risk.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Scenario Prioritization](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Scenario_Prioritization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When prioritizing platform scenarios, DevOps Architects and Principals often face complex trade-offs between importance and architectural difficulty. The goal is to ensure that high-impact issues are addressed in a manageable way, while also identifying critical areas that might not be immediately apparent but could have far-reaching consequences.

TL;DR: In evaluating the priority of platform scenarios, it's crucial to consider both the impact (importance) and the technical complexity. A scenario like cross-cloud disaster failover, despite being low on importance due to its less frequent occurrence, is still worth analyzing because it can reveal foundational risks that could affect multiple aspects of the system.

#### Triage
Understanding the Question

The question asks us to balance two key factors:

Importance: The significance and impact of a given platform scenario.
Architectural Difficulty: The complexity involved in addressing or implementing a solution for the scenario.

We need to assess scenarios based on these factors, even if one factor seems more favorable than the other. For instance, cross-cloud disaster failover might not seem immediately critical due to its infrequent occurrence but could expose deeper architectural issues that are crucial for overall system resilience and reliability.

#### First-Principles Solution

To prioritize effectively, let's break down the scenario of a low-importance but high-difficulty cross-cloud disaster failover:

Identify Importance:

Determine how critical this scenario is to business operations.
Assess the probability and potential impact if such an event were to occur.

Assess Architectural Difficulty:

Evaluate the complexity of implementing a robust cross-cloud disaster recovery solution.

#### Cons
ider the technical challenges, including integration with different cloud providers' APIs, security requirements, network latency, etc.

Risk Assessment:

Conduct a risk assessment to determine if this scenario could expose broader architectural weaknesses.
For example, a failing infrastructure component in one cloud might cascade and affect other services across multiple clouds.

Impact on Foundational Risks:

Identify whether addressing this low-importance but high-difficulty scenario could prevent more critical failures elsewhere.

#### Cons
ider the ripple effects of such a solution on overall system stability and scalability.

#### Example
Cross-Cloud Disaster Failover

Let's walk through an example to illustrate:

Scenario: A cross-cloud disaster failover mechanism is required for mission-critical applications hosted in two different cloud providers.

Importance:

This scenario might be considered low importance because it occurs infrequently.
However, its impact can be significant if the application needs to maintain continuous availability across both clouds.

Architectural Difficulty:

Implementing a cross-cloud disaster recovery solution requires complex orchestration and coordination between different cloud providers' APIs and services.
Ensuring data consistency and secure failover procedures add to the technical complexity.

Risk Assessment and Foundational Risks:

If this scenario is not addressed, it could lead to data inconsistencies or security vulnerabilities that affect other parts of the system.
For example, a failure in one cloud provider might not be immediately detected if the failover mechanism isn't well-defined, leading to cascading failures.

Priority Decision:

Given the high difficulty and potential foundational risks, this scenario is worth analyzing and prioritizing.
Although it may seem low importance at first glance, addressing it could prevent broader architectural issues that would be much harder to resolve later on.

#### Closing Summary
+ Common Mistakes

In conclusion, when prioritizing platform scenarios, we must consider both importance and architectural difficulty. A seemingly low-importance scenario like cross-cloud disaster failover can still be critical if it exposes foundational risks that could impact the system's overall resilience and stability.

#### Common Mistakes
to Avoid:

Overlooking high-difficulty scenarios because they seem less immediately important.
Failing to conduct a comprehensive risk assessment before making prioritization decisions.
Ignoring potential long-term impacts on system reliability due to initial low-importance considerations.

By thoroughly analyzing both factors, we can make informed and strategic decisions that ensure the platform remains robust and resilient against various failure scenarios.

**Comparison:**

| Scenario      | Importance (I) | Difficulty (D) | Priority Score |
|---------------|----------------|----------------|---------------|
| Cross-Cloud   | 3              | 8              | 12            |
| Disaster      |                | Failover       |               |
| Local         | 9              | 4              | 5             |
| Network       | Optimization   |               |               |

**Code:**

```python
# Example pseudo-code for assessing importance vs. difficulty

def prioritize_scenarios(scenario_list):
    prioritized_scenarios = []

    for scenario in scenario_list:
        importance = evaluate_importance(scenario)
        difficulty = evaluate_difficulty(scenario)

        if (importance * 0.75) + (difficulty * 1.25) > threshold:  # Adjust weights as needed
            prioritized_scenarios.append((scenario, importance, difficulty))

    return sorted(prioritized_scenarios, key=lambda x: x[1] * 0.75 + x[2] * 1.25)
```

**Example input / output:**

```text
| Scenario            | Importance (1-10) | Difficulty (1-10) | Impact Risk Assessment |
|---------------------|------------------|------------------|-----------------------|
| Cross-Cloud Disaster Failover | 3                | 8                | High                  |
| Local Network Optimization    | 9                | 4                | Low                   |

Prioritized Scenarios:
1. **Cross-Cloud Disaster Failover**
2. **Local Network Optimization**

Note: The prioritization is based on a weighted sum of importance and difficulty.
```

**Data:**

```json
{
  "scenarios": [
    {
      "name": "Cross-Cloud Disaster Failover",
      "importance": 3,
      "difficulty": 8,
      "impact_risk_assessment": "High"
    },
    {
      "name": "Local Network Optimization",
      "importance": 9,
      "difficulty": 4,
      "impact_risk_assessment": "Low"
    }
  ]
}
```

**Flow:**

1. High
2. Low
3. High
4. Low
5. Start
6. Evaluate Importance
7. Low Importance Scenario
8. Evaluate Difficulty
9. Prioritize High-DifficultyScenario
10. Ignore Low-Importance,Low-Difficulty Scenario
11. Analyze Foundational Risks
12. Prioritize if Foundational RiskExposure is Significant
13. Implement and Test Solution
14. End

