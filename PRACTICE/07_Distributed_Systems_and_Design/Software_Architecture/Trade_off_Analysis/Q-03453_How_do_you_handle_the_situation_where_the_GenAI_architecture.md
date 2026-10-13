---
type: "practice"
difficulty: "Deep"
---

# How do you handle the situation where the GenAI architecture presented in Phase 1 is the target-state diagram (a clean multi-agent design) while the reality is a pile of prompt-glue in a monolith? Do you evaluate the intended or the as-built system, and how does that shape your findings?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Regulatory Requirements in Utility Trees](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Regulatory_Requirements_in_Utility_Trees.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're facing a classic challenge in architectural trade-off analysis (ATAM): the gap between the ideal design presented during Phase 1 and the reality of an implementation. The key is to evaluate the as-built system while understanding its intended purpose, which involves a nuanced approach.

Firstly, let's clarify what ATAM entails. ATAM is a systematic method for analyzing architectural trade-offs by evaluating alternative designs. It’s crucial in ensuring that decisions made during the design phase align with operational realities and future needs. In this context, we need to assess whether the monolithic system with prompt-glue logic meets critical objectives such as scalability, maintainability, and performance.

#### Triaging the Question

The question is probing our ability to:

Evaluate the Current State: Understand that while Phase 1 provided a clean multi-agent design, reality often involves compromises.
Intended vs. As-Built Systems: Determine which system we are analyzing—whether it’s the intended design or the actual implementation.
Identify Trade-offs and Risks: Recognize the implications of these discrepancies on long-term maintainability and scalability.

#### First-Principles Solution

Step 1: Assessing the Intended vs. As-Built Systems

The first step is to clearly define what "intended" means in this context. The intended system, as outlined in Phase 1, likely included clear architectural principles such as modularity, separation of concerns, and a clean multi-agent design. In contrast, the monolithic implementation with prompt-glue logic might be more pragmatic for quick development but lacks many of these key attributes.

Step 2: Evaluating the Current System

We must evaluate the current system to identify its strengths and weaknesses:

Strengths: Identify any aspects where the monolith is working well. For example, if it has achieved rapid deployment or handles certain use cases effectively.
Weaknesses: Highlight issues such as difficulty in scaling, integration challenges, or maintenance overhead.

Step 3: Conducting Trade-off Analysis

Using ATAM, we need to map out various trade-offs:

Scalability vs. Development Speed: The monolith might offer quick development but could struggle with scalability.
Maintainability vs. Customizability: A multi-agent design may be more maintainable and customizable in the long run compared to a complex monolithic codebase.

Step 4: Recommending Adjustments

Based on our analysis, we can recommend adjustments that balance these trade-offs:

Hybrid Approach: Introduce microservices where feasible, while maintaining key services as monoliths.
Layered Architecture: Implement layers such as domain-driven design or clean architecture to improve maintainability without fully abandoning the current system.

#### Closing Summary
and Common Mistakes

In this analysis, we must balance pragmatism with long-term goals. Evaluating only the intended system would miss critical insights into why the monolith was built in the first place. Conversely, focusing solely on the as-built system without considering its limitations could lead to unnecessary rework or suboptimal solutions.

#### Common mistakes
include:

Over-emphasizing one aspect (e.g., performance over maintainability) at the expense of others.
Ignoring the context and constraints under which the current design was implemented.
Failing to recognize that both intended and as-built systems offer valuable insights for informed decision-making.

**Comparison:**

| Criteria        | Current State   | Target State    |
|-----------------|----------------|----------------|
| Performance     | 95             | N/A            |
| Scalability     | No             | Yes            |
| Maintainability | Low            | High           |
| Customizability | Limited        | High           |

**Code:**

```python
# Example pseudo-code for evaluating trade-offs in a monolithic system
def evaluate_system(current_state, target_state):
    strengths = []
    weaknesses = []

    # Assess current state's performance metrics
    if current_state.performance_metrics > 90:
        strengths.append("Good Performance")

    # Identify scalability issues
    if not is_scalable(current_state.codebase):
        weaknesses.append("Lack of Scalability")

    # Suggest adjustments based on trade-offs
    if weaknesses and not strengths:
        recommend_adjustments()
```

**Example input / output:**

```text
Input:
- Current State: Monolithic system with prompt-glue logic.
- Target State: Clean multi-agent design.

Output:
- Strengths of current state: Quick deployment, handles certain use cases effectively.
- Weaknesses of current state: Difficult scalability, integration challenges, high maintenance overhead.
- Recommended adjustments: Implement a hybrid approach, introduce microservices where feasible, maintain key services as monoliths.
```

**Data:**

```json
{
  "current_state": {
    "performance_metrics": 95,
    "is_scalable": false,
    "maintainability_score": 60
  },
  "target_state": {
    "modularity": true,
    "scalability": true,
    "customizability": true
  }
}
```

**Flow:**

1. No
2. Current State
3. Is Current State Satisfactory?
4. Identify Strengths andWeaknesses
5. Evaluate Trade-offs
6. Recommend Adjustments
7. Implement Hybrid Approach ifNecessary

