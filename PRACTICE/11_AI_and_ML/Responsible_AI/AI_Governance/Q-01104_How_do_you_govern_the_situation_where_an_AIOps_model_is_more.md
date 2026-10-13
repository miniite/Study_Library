---
type: "practice"
difficulty: "Expert"
---

# How do you govern the situation where an AIOps model is more accurate than your best human on-call, and the org is tempted to remove human oversight entirely — where do you hold the line and why?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Human in the Loop Governance for AIOps](<../../../../11_AI_and_ML/Responsible_AI/AI_Governance/Human_in_the_Loop_Governance_for_AIOps.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

As an AIOps Architect/Principal, governing the balance between machine intelligence and human oversight in critical operations like on-call support requires a nuanced approach. The key is to ensure that the deployment of advanced AI models does not compromise safety, compliance, or the overall quality of care provided by the organization. Here’s how we can govern this situation:

First, it's crucial to understand the context: the model might be more accurate, but its reliability, explainability, and robustness need thorough evaluation. The organization must ensure that any decision to fully automate on-call support is well-informed.

#### Triaging the Question

The core of the question revolves around responsible AI governance in operations-critical domains. Specifically, we're probing how you would balance technological advancements with human oversight to maintain a safe and effective system. Key assumptions include:

The model's accuracy is verified through rigorous testing.
There are clear guidelines for when to intervene manually.
Human expertise can still add value beyond just the on-call role.

#### First-Principles Solution
Establish Clear Governance Frameworks: Develop policies that outline how AI models will be used, monitored, and updated. This includes setting thresholds for when human intervention is required, such as model confidence levels or specific failure cases.
Implement Explainability and Transparency: Ensure the AI model's decision-making process can be audited. This helps in understanding why certain decisions are made and facilitates quicker identification of issues.
Human-in-the-Loop Mechanisms: Design systems where human oversight is not completely removed but integrated at key points, such as during critical operations or when anomalies occur.
Continuous Monitoring and Feedback Loops: Set up mechanisms to continuously monitor the model's performance and gather feedback from both technical and operational teams. This helps in refining the AI’s algorithms and improving its accuracy over time.
Compliance and Ethical Considerations: Ensure that all actions comply with relevant regulations, ethical standards, and organizational values.

#### Closing Summary

By implementing a robust governance framework that balances technological advancements with human oversight, we can ensure safe and effective operations. The key is to maintain flexibility and adaptability in our approach, recognizing the ongoing need for human expertise even as AI capabilities evolve.

#### Common Mistakes/Follow-Ups

Over-reliance on automated systems without adequate safeguards.
Ignoring ethical and compliance considerations.
Failing to establish clear guidelines and roles for human intervention.
Lack of continuous monitoring and feedback mechanisms.

**Comparison:**

```
Thresholds for Human Intervention	Confidence Level	Action
Low	< 0.85	Raise alert
Medium	< 0.92	Manual review required
High	>= 0.97	Proceed with automation
```

**Code:**

```python
# Example code snippet for a simple threshold-based decision mechanism
def should_human_intervene(model_confidence):
    if model_confidence < 0.95:
        return True
    else:
        return False

# Sample data
model_output = {"confidence": 0.93, "prediction": "High"}
if should_human_intervene(model_output["confidence"]):
    print("Human intervention required.")
else:
    print("Proceed with the automated decision.")
```

**Example input / output:**

```text
Input:
- Model Confidence Level = 0.84

Output:
Human intervention required.

Explanation:
The model's confidence level is below the predefined threshold, triggering a manual review.
```

**Data:**

```json
{
  "thresholds": {
    "low_confidence": 0.85,
    "medium_confidence": 0.92,
    "high_confidence": 0.97
  },
  "human_intervention_points": [
    {"condition": "confidence < low_confidence", "action": "raise alert"},
    {"condition": "confidence < medium_confidence", "action": "manual review required"}
  ]
}
```

**Flow:**

1. < 0.85
2. >= 0.85 and < 0.92
3. >= 0.97
4. Model Output
5. Confidence Level
6. Raise Alert
7. Manual Review Required
8. Proceed with Automation

