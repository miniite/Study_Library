---
type: "practice"
difficulty: "Deep"
---

# What's your governance framework for tracking which AIOps decisions were made under a specific model version and policy version, so a regulator can be shown the exact governing state at decision time?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Auditability and Compliance Evidence](<../../../../11_AI_and_ML/Responsible_AI/AI_Governance/Auditability_and_Compliance_Evidence.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively track and govern AI decisions in an AIOps environment, ensuring transparency and accountability to regulators, we need a comprehensive governance framework that meticulously records and correlates model versions, policy configurations, and the actual decisions made at runtime. This involves creating a detailed audit trail that captures not only the metadata of models and policies but also their interactions during decision-making processes.

The key idea is to establish a robust tracking mechanism that logs every relevant aspect of the AI system's operation, including version numbers, applied policies, input data, intermediate computations, and final output decisions. This log should be structured in a way that allows for easy retrieval and verification by regulators. Below, I'll outline this framework step-by-step from first principles.

**Code:**

```python
# Example code to illustrate the concept of logging decision metadata

class DecisionLog:
    def __init__(self, model_version, policy_version):
        self.model_version = model_version
        self.policy_version = policy_version
        self.decisions = []

    def log_decision(self, input_data, output_decision, timestamp):
        self.decisions.append({
            'timestamp': timestamp,
            'input_data': input_data,
            'output_decision': output_decision
        })

# Example usage
log = DecisionLog(model_version="v1.0", policy_version="p2")
log.log_decision(input_data={"feature1": 1.0, "feature2": 2.0},
                 output_decision=True,
                 timestamp="2023-10-05T14:30:00Z")

print(log.decisions)
```

**Example input / output:**

```text
# Example input and expected output

log = DecisionLog(model_version="v1.0", policy_version="p2")
log.log_decision({"feature1": 1.0, "feature2": 2.0}, True, "2023-10-05T14:30:00Z")

expected_output = [
    {
        'timestamp': "2023-10-05T14:30:00Z",
        'input_data': {"feature1": 1.0, "feature2": 2.0},
        'output_decision': True
    }
]
assert log.decisions == expected_output, "The decision log does not match the expected output."
```

**Data:**

```yaml
# Example data schema for decision logs
decision_log:
  model_version: "v1.0"
  policy_version: "p2"
  decisions:
    - timestamp: "2023-10-05T14:30:00Z"
      input_data: {feature1: 1.0, feature2: 2.0}
      output_decision: true
```

