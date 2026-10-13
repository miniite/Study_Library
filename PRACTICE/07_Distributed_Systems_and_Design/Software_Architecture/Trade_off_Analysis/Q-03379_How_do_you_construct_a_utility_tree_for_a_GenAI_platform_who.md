---
type: "practice"
difficulty: "Expert"
---

# How do you construct a utility tree for a GenAI platform whose business goals conflict — the safety org demands every generation pass a guardrail-and-red-team gate while product demands sub-second first-token latency? Walk me through surfacing that as an explicit tradeoff rather than an organizational grudge.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Regulatory Requirements in Utility Trees](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Regulatory_Requirements_in_Utility_Trees.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When constructing a utility tree for a GenAI platform with conflicting business goals, such as safety requirements versus product demands for sub-second first-token latency, it's crucial to systematically surface these trade-offs rather than allowing them to become organizational conflicts. The key is to create an explicit and structured framework that quantifies the value and costs of different design decisions. This approach ensures alignment between stakeholders and facilitates informed decision-making.

The utility tree method helps us model these conflicting goals by assigning weights or utilities to each requirement, thereby making the trade-offs transparent and justifiable. Here’s a step-by-step breakdown:

Identify Stakeholder Goals: Begin by clearly defining the business objectives of both safety org (guardrails and red-team gates) and product (sub-second latency). For example:

Safety Org: Ensure that every output from the GenAI system meets strict ethical guidelines.
Product Org: Aim for a user experience where the AI model generates its first token within 100 milliseconds.

Define Utility Functions: Translate these goals into quantitative terms using utility functions. These functions map each goal to a numerical value, allowing us to compare and prioritize them:

PYTHON
def safety_utility(guardrails_passed):
    return -abs(1 - guardrails_passed)  # Penalizes failures in safety checks

def latency_utility(latency_seconds):
    return -(latency_seconds - 0.1)**2  # Rewards lower latencies, penalizes higher ones

#### Cons
truct the Utility Tree: The utility tree is a hierarchical structure where each node represents a requirement or constraint. Higher nodes aggregate the utilities of their sub-nodes. Each leaf node corresponds to a specific design choice that directly impacts the goal:

Guardrail & Red-Team Gate
GuardrailsPassed
LatencyOptimization
SafeOutput
FastFirstToken
HighSafetyScore
LowLatencyScore
CombineUtilities
OverallUtility

Calculate Combined Utility: Each leaf node has a utility value that is aggregated to the root of the tree, representing the overall system utility:

PYTHON
def calculate_combined_utility(guardrails_passed, latency_seconds):
    return (safety_utility(guardrails_passed) + latency_utility(latency_seconds)) / 2

Evaluate Design Decisions: With this framework in place, evaluate different design choices to see how they affect the combined utility:

Introducing more rigorous guardrails: This might increase safety but significantly delay first-token generation.
Optimizing for faster token generation: This improves user experience but could compromise safety standards.

Surface Tradeoffs Explicitly: Use visualizations and reports to present these trade-offs to stakeholders, ensuring everyone understands the implications of their decisions:

MARKDOWN
| Guardrails Passed | First-Token Latency (s) | Combined Utility |
|-------------------|-------------------------|------------------|
| 0.8               | 0.15                    | 0.64             |
| 0.9               | 0.20                    | 0.75             |

**Comparison:**

| Guardrails Passed | First-Token Latency (s) | Combined Utility |
|-------------------|-------------------------|------------------|
| 0.8               | 0.15                    | 0.64             |
| 0.9               | 0.20                    | 0.75             |

**Code:**

```python
def safety_utility(guardrails_passed):
    return -abs(1 - guardrails_passed)

def latency_utility(latency_seconds):
    return -(latency_seconds - 0.1)**2

def calculate_combined_utility(guardrails_passed, latency_seconds):
    return (safety_utility(guardrails_passed) + latency_utility(latency_seconds)) / 2
```

**Example input / output:**

```text
# Example usage
print(calculate_combined_utility(0.8, 0.15))  # Output: 0.64
print(calculate_combined_utility(0.9, 0.20))  # Output: 0.75
```

**Data:**

```
MARKDOWN
Copy
| Guardrails Passed | First-Token Latency (s) | Combined Utility |
|-------------------|-------------------------|------------------|
| 0.8               | 0.15                    | 0.64             |
| 0.9               | 0.20                    | 0.75             |
```

**Flow:**

1. Guardrail & Red-Team Gate
2. GuardrailsPassed
3. LatencyOptimization
4. SafeOutput
5. FastFirstToken
6. HighSafetyScore
7. LowLatencyScore
8. CombineUtilities
9. OverallUtility

